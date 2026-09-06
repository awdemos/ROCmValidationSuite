# Agent Guidelines for ROCm Validation Suite

ROCm Validation Suite (RVS) is a system-level validation and diagnostics tool for AMD GPUs. It is a C++20 CMake project built against the ROCm stack, rocBLAS, and the SMI library.

## Project Layout

```
CMakeLists.txt              # top-level build
src/                        # shared utilities (rsmi, hsa, logger)
gm.so/ gpup.so/ gst.so/ ... # per-module shared libraries
rvs/conf/                   # example YAML configuration files for each module
bin/                        # helper scripts
external/TransferBench/     # bundled TransferBench sources
```

## Critical Rules

1. **Every module has a test target.** Add tests under `<module>/test/` and register them with CMake `enable_testing()`.
2. **YAML configs are load-tested.** Example `.conf` files under `rvs/conf/` are the canonical input format; any config parser change must keep them valid.
3. **GPU targets are configurable.** Use `-DGPU_TARGETS` or `GPU_FAMILY` to select architectures; the default is multi-arch.
4. **Packaging affects many files.** Avoid touching `DEBIAN/`, `build_packages_local.sh`, or `.github/workflows/*-tests.yml` unless the change is packaging-related.

## Build Commands

Prerequisites: CMake 3.5+, ROCm stack, rocBLAS, `rocm-smi-lib` (or `amd-smi-lib` on ROCm 7.0+), yaml-cpp, libnuma, libpci.

```bash
# Standard Release build
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release       -DGPU_FAMILY=gfx110X-all       -DBUILD_TRANSFERBENCH_CLI=OFF       ..
make -j$(nproc)
```

## Test Commands

```bash
cd build

# Run CTest suite
ctest --output-on-failure

# Run a specific RVS module with a config
./rvs -c ../rvs/conf/gst_single.conf -d 3
```

## Lint / Format

```bash
# Project uses no central formatter; keep C++ style consistent with surrounding files.
# CI enforces via review; run a local build with -DCMAKE_BUILD_TYPE=Debug before pushing.
```

## Local Package Build

```bash
# Mimics CI package generation locally
GPU_FAMILY=gfx110X-all ./build_packages_local.sh
```

## Gotchas

- `rocm-smi-lib` was renamed to `amd-smi-lib` in ROCm 7.0; the build switches on `FETCH_ROCMPATH_FROM_ROCMCORE`.
- In-source builds are blocked by CMake (`CMAKE_BINARY_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR`).
- The default `GPU_FAMILY` is `multiarch`; set it explicitly when building for a known target.
