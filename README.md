# libsql — Windows ARM64

Produces a native **Windows ARM64** build of [`libsql`](https://github.com/tursodatabase/libsql-js)
(Turso's Neon-based Node bindings) as a `@libsql/win32-arm64-msvc` artifact — the one
platform package Turso does not publish, which blocks native ARM64 Electron/Node apps
that depend on libsql.

Consumed by [`superset-windows-arm64`](https://github.com/khairm/superset-windows-arm64),
which materializes this artifact instead of the missing upstream package.

## Status

🔬 **Spike phase.** `spike-build.yml` (manual `workflow_dispatch` only) proves the
`aarch64-pc-windows-msvc` compile on the native `windows-11-arm` runner before the full
nightly is wired up.

## Planned (after spike proves out)

Same shape as `superset-windows-arm64`: a nightly GitHub Action that **only builds when
the needed libsql version changes** (change-gated `check` job), uses Claude Code to apply
the Windows-ARM64 build patches from `PATCHES.md` (currently: add
`aarch64-pc-windows-msvc → @libsql/win32-arm64-msvc` to `package.json` `neon.targets`),
builds via Neon, and publishes a versioned `@libsql/win32-arm64-msvc` Release with a SHA256.

## Attribution

Upstream: [tursodatabase/libsql-js](https://github.com/tursodatabase/libsql-js) (MIT).
This repo only compiles the missing Windows ARM64 target; not affiliated with Turso.
