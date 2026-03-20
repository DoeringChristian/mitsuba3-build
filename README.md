# Mitsuba 3 Build Environment

A comprehensive build, test, and profiling environment for Mitsuba 3 and DrJit, managed with Pixi and direnv.

## Overview

This repository provides a streamlined development environment for working with Mitsuba 3 and its dependencies. It includes:

- Automated dependency management via [Pixi](https://pixi.sh/)
- Environment setup with [direnv](https://direnv.net/)
- Build scripts for Mitsuba 3 and DrJit
- Testing utilities for both Python and C++ codebases
- Profiling tools for CPU and GPU performance analysis
- Debugging helpers with GDB integration

## Prerequisites

### Required
- **direnv**: For automatic environment activation
- **pixi**: For dependency management (conda-based)
- **bear**: For generating compile commands (system requirement)

### System Requirements
- **Platform**: Linux (x86_64)
- **CUDA**: Version 13

## Getting Started

### 1. Clone and Setup

```bash
# Clone the repository with submodules
git clone --recursive <repository-url>
cd mitsuba3-build

# Allow direnv to load the environment
direnv allow
```

The `.envrc` file will automatically:
- Initialize the Pixi environment
- Add `bin/` scripts to your PATH
- Set `MITSUBA_ROOT` to the mitsuba3 submodule
- Disable auto-formatting with `DISABLE_AUTOFORMAT=true`

### 2. Install Dependencies

Pixi will automatically install all dependencies when you enter the directory. Dependencies include:

**Build Tools:**
- CMake 3.28, Ninja, GCC 13.2, Clang 18
- Linkers: mold, lld
- Build acceleration: ccache

**Libraries:**
- CUDA toolkit with Nsight Compute profiler
- Embree 4.3+ (ray tracing kernels)
- zlib 1.3.1

**Python Environment:**
- Python 3.12
- PyTorch 2.5+ with torchvision and torchaudio
- Scientific computing: numpy, scipy, matplotlib
- Development: pytest, notebook, jupyter
- Documentation: Sphinx with Furo theme

**Optional Dependencies:**
- tiny-cuda-nn (installed via `pixi run tcnn-install`)

## Build Scripts

All build scripts are located in the `bin/` directory and are automatically added to your PATH.

### Building DrJit

```bash
# Build DrJit (main library)
build-drjit

# Build DrJit core components
build-drjit-core

# Build DrJit documentation
build-drjit-docs
```

**Build Configuration (DrJit):**
- Build type: Debug
- Generator: Ninja
- Tests: Enabled
- Location: `$MITSUBA_ROOT/build-drjit`

### Building Mitsuba

```bash
# Build Mitsuba 3
build-mitsuba
```

**Build Configuration (Mitsuba):**
- Build type: Debug
- Generator: Ninja
- Compiler: GCC 13.2
- Linker: mold (for fast linking)
- Compile commands: Exported (for IDE integration)
- Location: `$MITSUBA_ROOT/build-mitsuba`

## Testing

### Python Tests

```bash
# Test DrJit (Python bindings)
test-drjit [pytest-args]

# Test Mitsuba (Python interface)
test-mitsuba [pytest-args]

# Test Mitsuba with AddressSanitizer
test-mitsuba-asan [pytest-args]
```

Examples:
```bash
# Run specific test file
test-mitsuba src/render/tests/test_scene.py

# Run with verbose output
test-drjit -v

# Run specific test by name
test-mitsuba -k test_sensor
```

### C++ Tests

```bash
# Test DrJit core (C++ tests)
test-drjit-core <test-name> [args]
```

Example:
```bash
test-drjit-core array
```

## Debugging

Both DrJit and Mitsuba tests can be run under GDB for debugging:

```bash
# Debug DrJit tests
debug-drjit [pytest-args]

# Debug Mitsuba tests
debug-mitsuba [pytest-args]
```

Example debugging session:
```bash
debug-mitsuba -k test_integrator
# In GDB:
(gdb) break mi::Integrator::render
(gdb) run
```

## Profiling

### CPU Profiling

```bash
# Profile with Linux perf
profile <command>
```

The script will:
- Set `kernel.perf_event_paranoid=1` for user-space profiling
- Record call graphs with DWARF format
- Use 8MB buffer with AIO and compression
- Output to `perf.data`

Example:
```bash
profile python -m pytest test_performance.py
perf report -i perf.data
```

### GPU Profiling

#### NVIDIA Nsight Systems

```bash
# Profile and automatically open UI
nsys-prof <nsys-args> <command>
```

Example:
```bash
nsys-prof -o my_profile python render_scene.py
# Automatically opens the most recent .nsys-rep file in nsys-ui
```

#### NVIDIA Nsight Compute

```bash
# Profile CUDA kernels
ncu-prof <command>
```

The script runs with:
- Full metric collection (`--set full`)
- UI opening enabled (`--open-in-ui`)
- No config file constraints
- Force overwrite of existing reports

Example:
```bash
ncu-prof python cuda_benchmark.py
```

### Benchmarking

For stable GPU benchmarking results, lock GPU clocks:

```bash
# Lock GPU and memory clocks, run command, then reset
lock-gpu <command>
```

This script:
- Enables persistent mode
- Locks GPU clocks to 1215 MHz
- Locks memory clocks to 810 MHz
- Runs your command
- Automatically resets clocks afterward

Example:
```bash
lock-gpu python benchmark.py
```

## Utility Scripts

### Logging

Capture command output to a file:

```bash
log <output-file> <command>
```

Example:
```bash
log build.log build-mitsuba
```

## Project Structure

```
mitsuba3-build/
├── .envrc                    # direnv configuration
├── pixi.toml                 # Pixi project configuration
├── pixi.lock                 # Locked dependency versions
├── bin/                      # Build, test, and profiling scripts
│   ├── build-drjit          # Build DrJit
│   ├── build-drjit-core     # Build DrJit core
│   ├── build-drjit-docs     # Build DrJit documentation
│   ├── build-mitsuba        # Build Mitsuba
│   ├── test-drjit           # Test DrJit (pytest)
│   ├── test-drjit-core      # Test DrJit core (C++)
│   ├── test-mitsuba         # Test Mitsuba (pytest)
│   ├── test-mitsuba-asan    # Test with AddressSanitizer
│   ├── debug-drjit          # Debug DrJit with GDB
│   ├── debug-mitsuba        # Debug Mitsuba with GDB
│   ├── profile              # CPU profiling with perf
│   ├── nsys-prof            # GPU profiling with Nsight Systems
│   ├── ncu-prof             # CUDA kernel profiling
│   ├── lock-gpu             # Lock GPU clocks for benchmarking
│   └── log                  # Capture command output
└── mitsuba3/                 # Mitsuba 3 source (submodule)
    ├── ext/drjit/           # DrJit source
    ├── build-drjit/         # DrJit build directory
    ├── build-drjit-core/    # DrJit core build directory
    └── build-mitsuba/       # Mitsuba build directory
```

## Environment Variables

The following environment variables are automatically set by `.envrc`:

- **`PATH`**: Includes `$(pwd)/bin` for easy access to scripts
- **`MITSUBA_ROOT`**: Points to the mitsuba3 submodule
- **`DISABLE_AUTOFORMAT`**: Set to `true`
- **`CMAKE_CXX_COMPILER_LAUNCHER`**: Set to `ccache` for faster rebuilds
- **`CMAKE_PREFIX_PATH`**: Points to Conda prefix
- **`LD_LIBRARY_PATH`**: Includes Conda libraries
- **`CUDA_TOOLKIT_ROOT_DIR`**: Points to CUDA installation

## Tips and Best Practices

### Fast Iteration

1. **Use ccache**: Already configured to speed up recompilation
2. **Use mold**: Fast linker already configured for Mitsuba builds
3. **Incremental builds**: Scripts preserve build directories for fast rebuilds

### Memory Usage

If builds fail due to memory constraints:
- Limit parallel jobs: `ninja -j4` (edit build scripts to add `-j` flag)
- Use swap space for large link operations

### GPU Profiling Tips

- Use `lock-gpu` for benchmarking to get consistent results
- For iterative profiling, use `nsys-prof` to automatically view results
- Use `ncu-prof` for detailed kernel analysis, but note it's much slower

### Debugging Tips

- Build type is Debug by default for symbol information
- Use `debug-*` scripts to run tests under GDB
- Compile commands are exported for IDE integration (`.json` in build dirs)

## Troubleshooting

### "direnv: error .envrc is blocked"
Run `direnv allow` to authorize the `.envrc` file.

### "command not found" errors
Ensure direnv is properly installed and the environment is loaded. Exit and re-enter the directory, or run `direnv reload`.

### CUDA/GPU errors
Verify CUDA 13 is available:
```bash
nvidia-smi
nvcc --version
```

### Build failures
Clean and rebuild:
```bash
rm -rf mitsuba3/build-*
build-mitsuba  # or build-drjit
```

## License

This build environment configuration is provided as-is. Refer to the Mitsuba 3 and DrJit repositories for their respective licenses.
