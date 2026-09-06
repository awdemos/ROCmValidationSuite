# ROCmValidationSuite

Agent context for the ROCm Validation Suite (RVS) repository.

## Project overview

- Languages: C/C++
- Build system: CMake ≥ 3.25
- Platform: Linux (Ubuntu, CentOS/RHEL, SUSE) with ROCm and AMD GPU(s)
- Layout: top-level CMake plus per-module `*.so/` directories; `rvs/` holds the CLI, `rvslib/` the shared runtime, and each `*.so/` directory is a loadable test module.
- Runtime: loadable modules are discovered by the CLI from a configured module directory.

## Prerequisites

Ubuntu dependencies:

```bash
sudo apt-get update && sudo apt-get install -y libpci3 libpci-dev doxygen unzip cmake git libyaml-cpp-dev libnuma-dev
```

CMake must be ≥ 3.25. Ubuntu 22.04 and earlier ship CMake 3.22; install a newer CMake from Kitware's APT repository if needed.

Install ROCm, rocBLAS, and the SMI library. For ROCm 6.4 and earlier use `rocm-smi-lib`; for ROCm 7.0 and later use `amd-smi-lib`.

## Setup commands

Clone with submodules (TransferBench is vendored under `external/TransferBench`):

```bash
git clone --recurse-submodules https://github.com/ROCm/ROCmValidationSuite.git
cd ROCmValidationSuite
```

If already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Build commands

Configure and build directly with CMake:

```bash
cmake -B ./build -DROCM_PATH=/opt/rocm -DCMAKE_INSTALL_PREFIX=/opt/rocm -DCPACK_PACKAGING_INSTALL_PREFIX=/opt/rocm
make -C ./build -j$(nproc)
```

Or use the local packaging helper:

```bash
./build_packages_local.sh
```

The bundled TransferBench CLI is off by default in the local packaging helper; enable it with `BUILD_TRANSFERBENCH_CLI=ON` if desired.

## Test / package commands

```bash
# Build Debian/RPM package
cd ./build && make package

# Install on Ubuntu
sudo dpkg -i rocm-validation-suite*.deb

# Install on RHEL/CentOS/SUSE
sudo rpm -i --replacefiles --nodeps rocm-validation-suite*.rpm

# Run the CLI after install
rvs -help

# Run a module config file
rvs -c conf/example.conf
```

## Key conventions

- CMake 3.25+ required.
- Each module is a shared object under a directory named `<module>.so/`.
- Module configuration files use YAML and are passed to `rvs -c <path>`.
- `external/TransferBench` is a git submodule.
- Match the RVS branch to the installed ROCm release for compatibility.

## Important gotchas

- The project requires a ROCm environment and AMD GPU hardware for meaningful module execution; some build steps may work without a GPU, but most runtime tests need one.
- If `/opt/rocm/lib/librocm_smi64.so` is missing after installing `rocm-smi-lib`, reinstall the package.
- `BUILD_TRANSFERBENCH_CLI` controls whether the vendored TransferBench CLI is built.
- Only one package type (DEB or RPM) will be produced per OS; ignore the unrelated packaging error.

## Useful shortcuts

```bash
# List available modules / options
rvs -help

# Run a smoke config after building
./build/bin/rvs -c ./conf/gpup_*.conf
```
