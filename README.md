# usdtweak-macos

Reproducible build of [usdtweak](https://github.com/cpichard/usdtweak) with
[Blender Cycles](https://www.cycles-renderer.org/) as a Hydra render delegate
on macOS Apple Silicon.

## What you get

- **usdtweak.app** — USD scene editor with viewport
- **Storm** — OpenGL/Metal rasterizer (default, from conda-forge OpenUSD)
- **Cycles** — GPU path tracer via Metal (Blender's hdCycles Hydra delegate)

## Prerequisites

- macOS on Apple Silicon (M1/M2/M3/M4)
- Xcode Command Line Tools (`xcode-select --install`)
- ~30 GB free disk space
- Internet connection (clones repos, downloads conda packages)

## Build

```
make build
```

This will:
1. Install [pixi](https://pixi.sh) (conda package manager) if needed
2. Clone & build usdtweak from the `develop` branch
3. Clone Blender + precompiled ARM64 libs
4. Create a USD compatibility wrapper (conda-forge USD ↔ Blender's `libusd_ms`)
5. Patch & build hdCycles against conda-forge USD 26.03
6. Assemble the final app bundle with both Storm and Cycles

## Run

```
make run
```

## Patches

`patches/hdcycles-usd26.patch` adapts Blender's hdCycles to USD 26.03:

- **mesh.cpp** — `ComputeTriangulatedFaceVaryingPrimvar` returns `HdMeshComputationResult` enum instead of `bool`
- **plugin.h/cpp** — add `IsSupported(HdRendererCreateArgs)` override (new pure virtual in USD 26.03)
- **material.cpp** — initialize shader with empty graph to prevent null dereference in Cycles internals
- **render_delegate.cpp** — include universal render context so UsdPreviewSurface materials work

## Configuration

| Variable | Default | Description |
|---|---|---|
| `USDTWEAK_BRANCH` | `develop` | usdtweak git branch |
| `BLENDER_BRANCH` | `main` | Blender git branch |
