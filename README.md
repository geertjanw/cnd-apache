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

## 5. Building

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

## 6. Installing the plugins

See [`releases/README.md`](releases/README.md). In short: **Tools → Plugins →
Downloaded → Add Plugins…**, select the `.nbm` files, Install, restart.

---

## 7. Using it with the DuckDB Java JDBC driver

A full worked example — a mixed **Java + C/C++** project where the Java side
talks to an embedded [DuckDB](https://duckdb.org) database through its JDBC
driver and the C/C++ side is a JNI native library built and debugged in the same
IDE — is in [`docs/DUCKDB-JDBC.md`](docs/DUCKDB-JDBC.md).

---

## 8. Upstream

- Official sources: <https://github.com/apache/netbeans/tree/cnd>
- Donation background: <https://cwiki.apache.org/confluence/display/NETBEANS/4th+and+5th+donations>
- Discussion on reviving CND: <https://github.com/apache/netbeans/discussions/5882>
