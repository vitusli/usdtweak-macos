# usdtweak-macos

Build [usdtweak](https://github.com/cpichard/usdtweak) with
[Cycles](https://www.cycles-renderer.org/) path tracing on macOS Apple Silicon (M-series).

This repo contains a self-contained build script that clones usdtweak and
Blender sources, patches Cycles' Hydra render delegate for OpenUSD 26.03,
and compiles everything into a single app bundle with both Storm (rasterizer)
and Cycles (Metal GPU path tracer) as viewport renderers.

## Build from source

```
git clone https://github.com/vitusli/usdtweak-macos.git
cd usdtweak-macos
make build   # ~45 min first time
make run
```

## Requirements

- macOS on Apple Silicon
- Xcode command-line tools (`xcode-select --install`)
- ~30 GB disk space
- Internet connection (clones repos, downloads conda packages)

The build script will install [pixi](https://pixi.sh) (conda package manager)
if not present.

## Make targets

| Target | Description |
|--------|-------------|
| `make build` | Clone, patch, and build everything |
| `make run` | Launch usdtweak with Cycles enabled |
| `make clean` | Remove source, build, and deps directories |

## What the patches fix

`patches/hdcycles-usd26.patch` adapts Blender's hdCycles to OpenUSD 26.03:

- **mesh.cpp** — `ComputeTriangulatedFaceVaryingPrimvar` returns
  `HdMeshComputationResult` enum instead of `bool` in USD 26.03
- **plugin.h/cpp** — add `IsSupported(HdRendererCreateArgs)` override,
  a new pure virtual in USD 26.03
- **material.cpp** — initialize shaders with an empty graph to prevent
  null dereference in Cycles internals when materials have no network
- **render_delegate.cpp** — include the universal render context so
  UsdPreviewSurface materials are picked up (not just `cycles:`-prefixed ones)

The build script also creates a `libusd_ms.dylib` compatibility wrapper that
re-exports all individual conda-forge USD libraries as the single monolithic
library Blender's build system expects.

## Known limitations

- First launch is slow (~60s) as Cycles compiles Metal shaders
- Only Metal GPU rendering (no CUDA/HIP/OneAPI on macOS)
- No OSL shading
- hdCycles.dylib has absolute rpaths to the build machine's dependency
  directories (not relocatable without `install_name_tool` fixups)

## Project structure

```
usdtweak-macos/
  build.sh                       # Main build script
  Makefile                       # Convenience targets
  patches/hdcycles-usd26.patch   # Cycles ↔ USD 26.03 compat patches
```

After building:

```
  source/          # usdtweak clone + pixi env
  deps/blender/    # Blender clone + precompiled ARM64 libs
  deps/conda_usd_compat/  # USD wrapper library
  build/usdtweak.app      # Final app bundle
```

## License

The build script and patches in this repository are provided under the
[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0),
matching the licenses of usdtweak, Cycles standalone, and OpenUSD.

Note: Pre-built binaries are not distributed because hdCycles links
statically against `bf_intern_guardedalloc` (GPL-2.0) from the Blender
source tree. Build from source to use.
