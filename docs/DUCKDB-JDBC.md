# Worked example: mixed Java + C/C++ with the DuckDB JDBC driver

This walks through a small **mixed-language** project that exercises the parts of
this Apache-licensed CND build that matter most:

- a **C/C++ dynamic-library** make project (built and `gdb`-debugged with your
  `gcc`/`clang` toolchain), and
- a **Java** application that queries an embedded [DuckDB](https://duckdb.org)
  database through the **DuckDB JDBC driver**.

The Java side asks DuckDB for a result, then hands that result to a C++ function
over **JNI** for native processing — both sides built, run and debugged inside
one Apache NetBeans IDE.

> **Note.** Automatic Java ↔ C/C++ *navigation* (jump from a Java `native`
> method to its C++ implementation) was provided by `cnd.mixeddev`, which is
> **not** part of this Apache build — it depends on the native code model that
> was not donated to Apache. The example below works fully without it; you
> simply switch between the Java and C++ editors yourself.

```
┌────────────┐   JDBC      ┌──────────┐
│  Java App  │ ──────────▶ │  DuckDB  │  (embedded, in-process)
│            │ ◀────────── │  (JNI)   │
│  native    │             └──────────┘
│  describe()│   JNI
│      │     │ ──────────▶ libducknative.(dylib|so|dll)   ← your C/C++ project
└────────────┘
```

---

## 0. Prerequisites

- This plugin set installed into Apache NetBeans (see [`../releases/README.md`](../releases/README.md)).
- A C/C++ toolchain on `PATH`: `gcc`/`g++` or `clang`/`clang++`, plus `make`.
  - macOS: `xcode-select --install` (and optionally `brew install llvm` for clangd).
  - Debian/Ubuntu: `sudo apt install build-essential clang clangd`.
- A JDK (17 or 21). Note `JAVA_HOME` — you need its `include/` headers for JNI.
- The DuckDB JDBC driver `org.duckdb:duckdb_jdbc` (it bundles the native DuckDB
  library for your OS/arch, so there is nothing else to install).
- *(Optional)* `clangd` if you want C/C++ completion & diagnostics — set it under
  **Tools → Options → C/C++ → clangd** (or the LSP options panel).

---

## 1. The C/C++ side — a JNI dynamic library

In NetBeans: **File → New Project → C/C++ → C/C++ Dynamic Library**. Name it
`ducknative`.

The JNI entry point must match the Java class and method you'll declare in
step 2 (`com.example.App.describe`). Add a source file `ducknative.cpp`:

```cpp
#include <jni.h>
#include <string>

// Must match: package com.example; class App; native method describe(long)
extern "C"
JNIEXPORT jstring JNICALL
Java_com_example_App_describe(JNIEnv *env, jobject /*self*/, jlong rowCount) {
    std::string msg =
        "DuckDB returned " + std::to_string(rowCount) + " rows — formatted in C++";
    return env->NewStringUTF(msg.c_str());
}
```

Point the compiler at the JNI headers. Right-click the project →
**Properties → C++ Compiler → Include Directories**, and add:

- `${JAVA_HOME}/include`
- `${JAVA_HOME}/include/darwin`  (macOS) · `.../include/linux` (Linux) · `.../include/win32` (Windows)

Build it (**F11**). You get `dist/.../libducknative.dylib` (macOS),
`libducknative.so` (Linux) or `ducknative.dll` (Windows).

> **Tip — generate the signature instead of hand-writing it.** After step 2,
> run `javac -h . src/com/example/App.java`; the generated header shows the exact
> `Java_com_example_App_describe` prototype.

---

## 2. The Java side — DuckDB over JDBC + the native call

Create a Java project (Ant, Maven or Gradle). If Maven, add the driver:

```xml
<dependency>
    <groupId>org.duckdb</groupId>
    <artifactId>duckdb_jdbc</artifactId>
    <version>1.1.3</version>
</dependency>
```

(For an Ant/plain project, just put `duckdb_jdbc-1.1.3.jar` on the classpath.)

`src/com/example/App.java`:

```java
package com.example;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class App {

    // Loads libducknative.(dylib|so) / ducknative.dll from java.library.path
    static { System.loadLibrary("ducknative"); }

    // Implemented in C++ in ducknative.cpp (see step 1).
    private native String describe(long rowCount);

    public static void main(String[] args) throws Exception {
        // "jdbc:duckdb:" = in-memory; use "jdbc:duckdb:/path/to/file.db" to persist.
        try (Connection conn = DriverManager.getConnection("jdbc:duckdb:");
             Statement st = conn.createStatement()) {

            st.execute("CREATE TABLE t AS SELECT * FROM range(1000) AS r(id)");

            try (ResultSet rs = st.executeQuery("SELECT count(*) FROM t")) {
                rs.next();
                long rows = rs.getLong(1);

                // Hand the DuckDB result to C++ over JNI:
                System.out.println(new App().describe(rows));
            }
        }
    }
}
```

Run with the native library's folder on `java.library.path`, e.g.:

```bash
java -Djava.library.path=/path/to/ducknative/dist/Debug/<toolchain> \
     -cp build/classes:duckdb_jdbc-1.1.3.jar \
     com.example.App
# → DuckDB returned 1000 rows — formatted in C++
```

(The DuckDB driver auto-registers via the JDBC SPI; no `Class.forName` needed.
Its native library is unpacked from the jar automatically.)

---

## 3. Working across the two languages

With both projects open in the same IDE you build, run and debug the Java app and
the C/C++ library side by side: set a breakpoint in `ducknative.cpp`, attach
`gdb`, and step into the native `describe` call. If you install **`clangd`**
(see the note below) you also get completion and diagnostics on the C++ side.

> **No automatic cross-language jump.** The `native` declaration in `App.java`
> and the `Java_com_example_App_describe` function in `ducknative.cpp` are linked
> only by the JNI naming convention — this Apache build has no `cnd.mixeddev`, so
> there is no Ctrl/Cmd-click that hops between them. Keep the naming in sync by
> generating the C header with `javac -h` (see the tip in step 1).

---

## 4. Troubleshooting

| Symptom | Fix |
|---|---|
| `UnsatisfiedLinkError: no ducknative in java.library.path` | Point `-Djava.library.path` at the folder holding the built `lib...`/`.dll`. |
| `UnsatisfiedLinkError: ...describe` (library loads, method missing) | The C++ symbol name must match `Java_<pkg>_<Class>_<method>` exactly — regenerate with `javac -h`. |
| `jni.h: No such file or directory` | Add `${JAVA_HOME}/include` and the OS subdir to the project's include paths (step 1). |
| No C/C++ completion/diagnostics | Install `clangd` and set its path in the C/C++ LSP options — intelligence is clangd-based in this build. |
| `No suitable driver found for jdbc:duckdb:` | The `duckdb_jdbc` jar isn't on the runtime classpath. |

---

## 5. Why DuckDB here

DuckDB's JDBC driver is a single self-contained jar with an **embedded native
engine** — no server to run — so it's an ideal companion for demonstrating a
mixed Java/C++ workflow: fast analytical SQL from Java, with your own native
C/C++ code alongside it, all built, run and navigated inside one Apache NetBeans
IDE.
