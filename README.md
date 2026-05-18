# libsql — Windows ARM64

Native **Windows ARM64** builds of [`tursodatabase/libsql-js`](https://github.com/tursodatabase/libsql-js)
published as the `@libsql/win32-arm64-msvc` platform package — the one
[Turso](https://github.com/tursodatabase) does not ship, which otherwise blocks
native ARM64 Electron/Node apps that depend on `libsql`.

Consumed by [`superset-windows-arm64`](https://github.com/khairm/superset-windows-arm64).

## How it works

1. A GitHub Actions workflow runs nightly (also `workflow_dispatch`).
2. **Change-gated:** it determines the target libsql version (latest on npm, or
   a `workflow_dispatch` input) and **only builds if that version is not
   already released here** — no rebuilds when nothing changed.
3. On a native `windows-11-arm` runner it compiles upstream `libsql-js`
   **unmodified** for `aarch64-pc-windows-msvc` (Rust + VS 2022), bypassing the
   neon-rs CLI (the built cdylib *is* the Node addon — see [`PATCHES.md`](PATCHES.md)).
4. It verifies the `index.node` is ARM64 and loads under ARM64 Node, then
   publishes a **per-version Release** (tag = the libsql version) with
   `libsql-win32-arm64-msvc.tar.gz` + a `.sha256`.

## Consuming it

Download the Release tagged with the libsql version you need:

```bash
gh release download <libsql-version> --repo khairm/libsql-windows-arm64 \
  -p 'libsql-win32-arm64-msvc.tar.gz' -p 'libsql-win32-arm64-msvc.tar.gz.sha256'
sha256sum -c libsql-win32-arm64-msvc.tar.gz.sha256
mkdir -p out && tar -xzf libsql-win32-arm64-msvc.tar.gz -C out   # -> out/index.node + out/package.json
```

Drop `out/` in as `node_modules/@libsql/win32-arm64-msvc` so `libsql`'s
`require("@libsql/win32-arm64-msvc")` resolves.

If a version you need has not been built yet, trigger it:

```bash
gh workflow run "Nightly libsql Windows ARM64 Build" \
  --repo khairm/libsql-windows-arm64 -f libsql_version=<version>
```

## Requirements / CI

Runs on the free `windows-11-arm` GitHub-hosted runner (public repos). No
secrets required — the build is deterministic (no AI patch step; upstream is
compiled unmodified).

## Attribution

Upstream: [tursodatabase/libsql-js](https://github.com/tursodatabase/libsql-js)
(MIT). This repo only compiles the missing Windows ARM64 target; not affiliated
with Turso.
