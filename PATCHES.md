# Windows ARM64 build notes — libsql

> **No upstream source patch is required.** Unlike the Superset port, building
> [`tursodatabase/libsql-js`](https://github.com/tursodatabase/libsql-js) for
> Windows ARM64 needs **zero modifications** to upstream source. The nightly
> compiles upstream unmodified and assembles the platform package
> deterministically. This file documents the recipe (the "patch" is purely a
> build/packaging procedure, applied deterministically by the workflow).

## The recipe (per `.github/workflows/nightly-build.yml`)

1. **Runner:** `windows-11-arm` (native ARM64; host triple
   `aarch64-pc-windows-msvc`). Has Rust-capable VS 2022 + libclang.
2. **Toolchain:** `rustup` honoring libsql-js's `rust-toolchain.toml` pin; add
   target `aarch64-pc-windows-msvc`. Set `LIBCLANG_PATH` (libSQL uses bindgen).
3. **Build:** `cargo build --release --target aarch64-pc-windows-msvc`.
   - **Do NOT use `neon dist` / `yarn build`.** neon-rs 0.0.165's `neon dist`
     rejects `--target` and breaks the Windows `cargo | neon` pipe
     (`os error 232`). It is unnecessary: for a Neon `cdylib` the produced
     `target/<triple>/release/libsql_js.dll` **is** the Node addon once renamed
     `index.node` (no symbol rewrite / trailer needed).
4. **Package:** copy `libsql_js.dll` → `index.node`; emit a minimal
   `package.json` `{ name: "@libsql/win32-arm64-msvc", version, os:["win32"],
   cpu:["arm64"], main:"index.node" }`. Prove it loads under ARM64 Node
   (`require(index.node)`), then tar + sha256 and publish a per-version Release
   (tag = the libsql version).

## Why this package is needed

Turso publishes `@libsql/{darwin,linux}-*` and `@libsql/win32-x64-msvc`, but
**no `@libsql/win32-arm64-msvc` at any version**. `libsql/index.js` does
`require("@libsql/" + target)` with no fallback, so native ARM64 Electron/Node
apps depending on `libsql` (e.g. via `@libsql/client` / Mastra) crash on
Windows ARM64. This repo fills exactly that gap.

## If upstream changes break the recipe

`tursodatabase/libsql-js` is compiled unmodified, so upstream churn rarely
matters. If a future libsql version fails to compile for
`aarch64-pc-windows-msvc`, add the minimal build fix here and document it in
this file (keeping the build deterministic).
