# Prebuilt plugins (`.nbm`)

These **42** `.nbm` files are the C/C++ (CND) plugins built from the Apache-2.0
source in this repository (`apache/netbeans@cnd`, tip `f04505c1e7`): **26** in
[`cnd/`](cnd/), **12** in [`dlight/`](dlight/) and **4** in
[`cpplite/`](cpplite/). They are licensed under the **Apache License,
Version 2.0** (see [`../LICENSE`](../LICENSE)).

## Requirements

- Apache NetBeans (built against the module versions on the `cnd` branch; a
  recent release is recommended).
- JDK 17 or 21.
- A C/C++ toolchain (`gcc`/`clang` + `make`) for building C/C++ projects.
- *(Optional)* `clangd` for code completion/diagnostics — this build delegates
  C/C++ intelligence to clangd over LSP.

## Install

1. In NetBeans: **Tools → Plugins**.
2. Open the **Downloaded** tab → **Add Plugins…**.
3. Select **all** the `.nbm` files in this folder → **Open**.
4. Click **Install**. The wizard resolves the correct install order.
5. Accept the **Apache License 2.0** → **Next** → **Finish**.
6. Restart the IDE when prompted.

## After restart

- New C/C++ projects: **File → New Project → C/C++**.
- Open existing make-based C/C++ projects.
- Debug with `gdb`; develop remotely via the `dlight` infrastructure.

See [`../docs/DUCKDB-JDBC.md`](../docs/DUCKDB-JDBC.md) for a worked mixed
Java + C/C++ example using the DuckDB JDBC driver.
