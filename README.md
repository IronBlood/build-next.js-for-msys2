# CI workflow

Uses MSYS2 with rustup to build SWC, Turborepo, and Next.js on GNU Windows targets, relying on each project’s pinned refs and nightly toolchain settings they require.

## Environment
- Defaults: `NEXT_JS_REF=v16.0.3`, `SWC_REF=v1.11.24`, `TURBO_REF=v2.5.5`, `CARGO_BUILD_TARGET=x86_64-pc-windows-gnu`.
- MSYS2 toolchain setup via `msys2/setup-msys2`; corepack enabled for Node.

## Releases
Artifacts (the native Next.js binding plus its dependencies) are stored as GitHub Releases, tagged with the Next.js ref plus a build timestamp (`NEXT_JS_REF-YYYYMMDDHHMMSS`).

## How to use
**Just don't.**

This is experimental work to explore running Next.js on unsupported platforms. Artifacts build, but on MSYS2 (UCRT64) loading the native module still fails:

```
$ node -e "require('./next-swc.win32-x64-gnu.node')"
node:internal/modules/cjs/loader:1920
  return process.dlopen(module, path.toNamespacedPath(filename));
                 ^

Error: A dynamic link library (DLL) initialization routine failed.
\\?\C:\msys64\home\runner\next-swc.win32-x64-gnu.node
    at Module._extensions..node (node:internal/modules/cjs/loader:1920:18)
    at Module.load (node:internal/modules/cjs/loader:1480:32)
    at Module._load (node:internal/modules/cjs/loader:1299:12)
    at TracingChannel.traceSync (node:diagnostics_channel:322:14)
    at wrapModuleLoad (node:internal/modules/cjs/loader:244:24)
    at Module.require (node:internal/modules/cjs/loader:1503:12)
    at require (node:internal/modules/helpers:152:16)
    at [eval]:1:1
    at runScriptInThisContext (node:internal/vm:219:10)
    at node:internal/process/execution:451:12 {
  code: 'ERR_DLOPEN_FAILED'
}
```

See the workflow at [.github/workflows/ci.yml](.github/workflows/ci.yml). There are a few hacks:

1. Packages for next.js are not installed, instead only `@napi-rs/cli` and `turbo` are installed. Haven't figured out why some `napi_` related native binding is called during `pnpm install`, and it blocks furthur steps.
2. `turbo.exe` is replaced by the custom build.
