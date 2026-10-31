# Apache NetBeans C/C++ (CND) — Apache-Licensed Build

This repository packages the **C/C++ development support ("CND")** of Apache
NetBeans — the modules that were donated by Oracle to the Apache Software
Foundation (the so‑called *"4th and 5th donations"*) and relicensed under the
**Apache License, Version 2.0**.

Every source file here carries the standard Apache 2.0 header and is taken
**unmodified** from the official [`apache/netbeans`](https://github.com/apache/netbeans)
repository's [`cnd` branch](https://github.com/apache/netbeans/tree/cnd). No
CDDL/GPL (Oracle/Sun) code is included.

> This is an independent build/packaging of official Apache NetBeans sources. It
> is **not** an official Apache NetBeans release and is **not** endorsed by the
> Apache Software Foundation. "Apache", "Apache NetBeans" and "NetBeans" are
> trademarks of the Apache Software Foundation.

---

## 1. Licensing — why this is Apache-licensed

The C/C++ support once shipped in NetBeans 8.2 under Sun/Oracle's
**CDDL‑1.0 OR GPL‑2.0‑only WITH Classpath‑exception‑2.0** dual license. That
8.2 code **cannot** simply be relabeled Apache.

However, Oracle **donated** the C/C++ support to the ASF and it went through
Apache's formal IP‑clearance process. The result is maintained on the
`apache/netbeans` **`cnd` branch**, where it has already been **relicensed to the
Apache License 2.0**. See
<https://cwiki.apache.org/confluence/display/NETBEANS/4th+and+5th+donations>.

This repository takes **only** that Apache‑2.0 code. Verification over `cnd/`,
`cpplite/` and `dlight/`:

| Check | Result |
|---|---|
| First‑party source files with the Apache 2.0 header | ~4,460 |
| First‑party source files with a CDDL/GPL license‑grant header | **0** |
| First‑party source files with an Oracle/Sun copyright line | **0** |

The only occurrences of the strings "CDDL" / "Sun Microsystems" anywhere in the
tree are **not** license grants, and all appear verbatim on the official branch:

- the build marker `# NOCDDL` in generated/sample Makefiles (it means *"do not
  stamp a CDDL header here"* — the files' real header is Apache);
- JavaHelp **DTD public identifiers** (`-//Sun Microsystems Inc.//DTD JavaHelp…`);
- the license **texts of bundled third‑party binaries** (`cpplite-mi`,
  `fs_server`) and the Apache‑RAT `licenseinfo.xml` entries that declare them.

The full license text is in [`LICENSE`](LICENSE); attribution is in
[`NOTICE`](NOTICE).

---

## 2. How this differs from `geertjanw/cnd`

[`geertjanw/cnd`](https://github.com/geertjanw/cnd) is a sibling project with a
**different goal and a different license**. The two are easy to confuse, so:

| | **this repo (`cnd-apache`)** | **`geertjanw/cnd`** |
|---|---|---|
| Source | Official `apache/netbeans@cnd` branch | NetBeans **8.2** sources (Sun/Oracle era) |
| License | **Apache License 2.0** | **CDDL‑1.0 OR GPL‑2.0‑only WITH Classpath‑exception‑2.0** |
| Copyright headers | ASF / Apache 2.0 | "Sun Microsystems" / "Oracle and/or its affiliates" (retained) |
| Plugins | 26 `cnd` + 4 `cpplite` + 12 `dlight` (**42 `.nbm`**) | ~55 `cnd` + 1 `cndext` + 5 `dlight` (61 `.nbm`) |
| C/C++ intelligence | **clangd via LSP** (install clangd separately) | **native** NetBeans code model (zero‑setup) |
| Refactoring, call graph, class view, DWARF discovery, CppUnit | ❌ not donated to Apache | ✅ included |
| Java ↔ C/C++ JNI navigation (`cnd.mixeddev`) | ❌ depends on the native code model, which was not donated | ✅ included |
| Can go toward Apache NetBeans upstream | **Yes** — already IP‑cleared & Apache‑licensed | **No** — would require ASF IP clearance first |
| Affiliation | unofficial build of official Apache sources | independent, unofficial 8.2 restoration |

**In one sentence:** `geertjanw/cnd` is the *fuller* C/C++ experience (the
complete NetBeans 8.2 feature set, including the native semantic engine and
refactoring) but it is **CDDL/GPL** code that was never relicensed and therefore
cannot be pushed to Apache NetBeans; **this repo** is the *leaner but genuinely
Apache‑2.0* C/C++ support — the subset Oracle actually donated and the ASF
relicensed — that delegates editing intelligence to `clangd`.

Pick `geertjanw/cnd` if you want maximum 8.2 feature parity for local use and are
fine with CDDL/GPL. Pick this repo if you need Apache‑2.0 licensing (e.g. to
redistribute, or to contribute toward Apache NetBeans).

The 31 NetBeans 8.2 modules that exist in `geertjanw/cnd` but are **absent** here
(because they were not part of the Apache donation) are the native code model
(`cnd.modelimpl`, `cnd.apt`, `cnd.antlr`, `libs.clank`, …), completion/highlight/
navigation, `cnd.refactoring`, the `cnd.repository`/indexing layer, project
`discovery`, `cnd.callgraph`, `cnd.classview`, and the CppUnit test integrations.

> **Note on `cnd.mixeddev`.** The official `cnd` branch carries the Java ↔ C/C++
> JNI-navigation module under `contrib/`, but it **depends on the native code
> model** (`cnd.api.model`, `cnd.modelutil`, `cnd.model.services`) that was *not*
> donated to Apache. It therefore cannot be built against the Apache `cnd`
> cluster and is **not** included here, in source or in the prebuilt plugins.

---

## 3. What's in here

Source is copied verbatim from `apache/netbeans@cnd`
(tip `f04505c1e7`, 2024‑01‑13):

### `cnd/` — C/C++ language & project support (26 modules)

Core model & utilities (`cnd`, `cnd.utils`, `cnd.utils.ui`, `cnd.ui`,
`cnd.source`, `cnd.lexer`, `cnd.editor`), **make projects**
(`cnd.makeproject`, `cnd.makeproject.ui`, `cnd.makeproject.source.bridge`),
**toolchains** (`cnd.toolchain`, `cnd.toolchain.ui`), **debugging**
(`cnd.debugger.common2`, `cnd.debugger.gdb2`, `cnd.dwarfdump`), **remote
development** (`cnd.remote`, `cnd.remote.ui`, `cnd.remote.projectui`,
`cnd.api.remote`, `cnd.api.remote.ui`, `cnd.api.project`), **clangd/LSP
intelligence** (`cnd.lsp`), **build systems** (`cnd.meson`), plus `cnd.kit`,
`cnd.script` and `jellytools.cnd`.

### `cpplite/` — lightweight clangd-based C/C++ editing (4 modules)

`cpplite.editor`, `cpplite.project`, `cpplite.debugger`, `cpplite.kit` — a
lighter LSP/clangd-driven C/C++ experience.

### `dlight/` — native execution & remote infrastructure (12 modules)

`dlight.remote*`, `dlight.libs.common`, `dlight.kit`, `dlight.sendto`, and the
remote VCS bridges (`git.remote`, `mercurial.remote`, `subversion.remote`,
`remotefs.versioning*`).

---

## 4. What this gives you (and what it does not)

C/C++ intelligence in this Apache build is delegated to **`clangd`** over LSP
(the `cnd.lsp` / `cpplite` modules) — it is **not** bundled. For code
completion, diagnostics and navigation you must install clangd (e.g.
`brew install llvm`, or your distro's `clang-tools`) and point the IDE at it.

| Capability | Status |
|---|---|
| C/C++ make projects (incl. Qt) — open / build / run | ✅ |
| Editor, syntax highlighting, toolchain management | ✅ |
| `gdb` debugging | ✅ |
| Remote development / `dlight` | ✅ |
| Completion / diagnostics / go‑to | ✅ **via clangd** (install separately) |
| Java ↔ C/C++ JNI navigation | ❌ (`cnd.mixeddev` depends on the native code model, not donated) |
| Native semantic refactoring, call graph, DWARF discovery, CppUnit | ❌ (these NB 8.2 modules were not part of the Apache donation) |

---

## 5. Tools & technologies for C/C++ in NetBeans

NetBeans CND is an **IDE front-end**, not a toolchain — it drives tools you
install on the host. Here is what you need, from "always" to "only for certain
samples".

### Base toolchain — needed for *any* C/C++ project

| Tool | What it does | macOS | Debian/Ubuntu | Windows |
|---|---|---|---|---|
| **C/C++ compiler** (GCC or Clang) | compiles your code | `xcode-select --install` (clang) | `sudo apt install build-essential` | MSYS2/MinGW‑w64 or Cygwin |
| **`make`** (GNU make) | NetBeans make projects build through it | comes with the CLT | `build-essential` | MSYS2/MinGW |
| **`gdb`** | the C/C++ debugger NetBeans integrates | `brew install gdb` (needs code‑signing) or use `lldb` | `sudo apt install gdb` | via MSYS2/MinGW |
| **`clangd`** | **completion, diagnostics & go‑to** — this build delegates C/C++ intelligence to clangd over LSP (it is *not* bundled) | `brew install llvm` | `sudo apt install clangd` | MSYS2 `mingw-w64-clang-tools-extra` |

After installing, point the IDE at the toolchain under **Tools → Options →
C/C++ → Build Tools**, and at clangd under the C/C++ **LSP / clangd** options.

### Java side (for the IDE itself and for mixed Java + C/C++)

- A **JDK (17 or 21)** runs Apache NetBeans and builds the Java half of a mixed
  project. JNI work also needs the JDK's `include/` headers (`${JAVA_HOME}/include`).

### Per-sample extras

The bundled samples (**File → New Project → C/C++ → Samples**) need these in
*addition* to the base toolchain:

| Sample | Extra tool / technology | Install |
|---|---|---|
| `helloqt` | **Qt** (qmake + moc; Qt core/gui/widgets) | `brew install qt` · `sudo apt install qtbase5-dev qttools5-dev-tools` — then set it under *Options → C/C++ → Qt* |
| `freeway` (GtkFreeway) | **GTK+ 2.0** and **`pkg-config`** (links via `` `pkg-config --libs gtk+-2.0` ``) | `brew install gtk+ pkg-config` · `sudo apt install libgtk2.0-dev pkg-config` |
| `lexyacc` | **flex** (`lex`) and **bison** (`yacc`) | `brew install flex bison` · `sudo apt install flex bison` |
| `hellocmake` | **CMake** | `brew install cmake` · `sudo apt install cmake` |
| `pi` | **POSIX threads** (pthreads) | part of the base toolchain — nothing extra |
| `mp` | **POSIX `fork()`** | part of the base toolchain — nothing extra |
| `hello`, `welcome`, `args`, `io`, `quote`, `fractal`, `helloapp` | — | base toolchain only |

### For the DuckDB walkthrough ([`docs/DUCKDB-JDBC.md`](docs/DUCKDB-JDBC.md))

- The **DuckDB JDBC driver** `org.duckdb:duckdb_jdbc` (a single jar that bundles
  the native engine — nothing else to install), plus the base toolchain for the
  JNI library and a JDK for the Java side.

---

## 6. Building

This is official Apache NetBeans source and builds with the standard Apache
NetBeans Ant build. Place these module trees in an `apache/netbeans` checkout
(or build the `cnd` branch directly) and run:

```bash
ant -Dcluster.config=cnd     build-nbms   # 26 cnd + 12 dlight .nbm plugins
ant -Dcluster.config=cpplite build-nbms   # 4 cpplite .nbm plugins
```

JDK 17 or 21 is required. The 42 prebuilt plugins are in
[`releases/`](releases/).

---

## 7. Installing the plugins

Two options:

- **Release zip** (easiest): download
  [`netbeans-cnd-apache-1.0-alpha.zip`](https://github.com/geertjanw/cnd-apache/releases/latest)
  from the [Releases](https://github.com/geertjanw/cnd-apache/releases) page — it
  bundles all 42 `.nbm`s, an autoupdate catalog and `INSTALL.txt`.
- **This repo**: the same `.nbm`s are in [`releases/`](releases/).

Either way: **Tools → Plugins → Downloaded → Add Plugins…**, select the `.nbm`
files, Install, accept the Apache 2.0 license, restart. See
[`releases/README.md`](releases/README.md) for details.

---

## 8. Usage

A full example — a mixed **Java + C/C++** project where the Java side
talks to an embedded [DuckDB](https://duckdb.org) database through its JDBC
driver and the C/C++ side is a JNI native library built and debugged in the same
IDE — is in [`docs/DUCKDB-JDBC.md`](docs/DUCKDB-JDBC.md).

---

## 9. Upstream

- Official sources: <https://github.com/apache/netbeans/tree/cnd>
- Donation background: <https://cwiki.apache.org/confluence/display/NETBEANS/4th+and+5th+donations>
- Discussion on reviving CND: <https://github.com/apache/netbeans/discussions/5882>
