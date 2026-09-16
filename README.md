# SpineRuntime

SpineRuntime is SpacemiT's Tile-SPMD execution runtime for launching parallel
kernels on SpacemiT RISC-V AI compute cores.

It is distributed as a prebuilt SDK. This repository provides public
documentation, integration examples, release information, and issue tracking.

[Releases](https://github.com/spacemit-com/spine-runtime/releases) |
[Documentation](docs/) |
[Issues](https://github.com/spacemit-com/spine-runtime/issues) |
[License](LICENSE)

## Prerequisites

### Supported Runtime Targets

| Target | Package | Status |
|---|---|---|
| RISC-V 64-bit Linux | `spine-runtime.riscv64.0.6.2.tar.gz` | Supported |
| RISC-V 64-bit OpenHarmony | `spine-runtime.riscv64.ohos.0.6.2.tar.gz` | Supported |
| GitHub `Source code` archives | Not an SDK | Documentation only |

The GitHub-generated `Source code` archives do not contain `libspert`, SDK
headers, CMake package files, or `pkg-config` metadata. Use the platform SDK
archive from the Releases page.

### Supported Backends

| Backend | Target | Default topology | Shared buffer | Status |
|---|---|---:|---:|---|
| `spacemit-k1` | SpacemiT K1 AI compute cores | 4 cores | 128 KiB/core | Supported |
| `spacemit-k3` | SpacemiT K3 A100 compute cores | 8 cores | 384 KiB/core | Verified on K3 |
| `spacemit-k3-x100` | K3 X100-compatible execution mode | 4 cores | 384 KiB equivalent | Platform-specific |
| `generic/qemu` | Linux, qemu-user, or systems without hardware | Simulation target | Simulation-dependent | Functional validation |

### Build Requirements

For native RISC-V Linux development, install a C++17 compiler, CMake, and
`pkg-config`.

For cross-compilation, use a RISC-V 64-bit toolchain and a matching sysroot.

## Quickstart

The quickest path is to download the SDK, build the repository quickstart
example, and run it on a compatible target.

### 1. Get the Repository

```console
git clone https://github.com/spacemit-com/spine-runtime.git
cd spine-runtime
```

### 2. Download the SDK

```console
curl -LO https://github.com/spacemit-com/spine-runtime/releases/download/0.6.2/spine-runtime.riscv64.0.6.2.tar.gz
```

### 3. Verify the SDK

```console
sha256sum spine-runtime.riscv64.0.6.2.tar.gz
```

Expected SHA-256:

```text
fc062cc83a49c98011976fa9620739841c74a619379785172d0f8959df65be47
```

### 4. Extract the SDK

```console
tar -xf spine-runtime.riscv64.0.6.2.tar.gz
export SPINE_RUNTIME_SDK="$PWD/spine-runtime.riscv64.0.6.2"
```

### 5. Build the Quickstart Example

```console
cmake -S examples/quickstart \
  -B build/quickstart \
  -DCMAKE_PREFIX_PATH="$SPINE_RUNTIME_SDK"

cmake --build build/quickstart --parallel
```

### 6. Run

```console
export LD_LIBRARY_PATH="$SPINE_RUNTIME_SDK/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
./build/quickstart/spine_quickstart
```

Expected output:

```text
SpineRuntime quickstart passed on N core(s)
```

`N` is the number of compute cores granted by the selected Backend.

## Quickstart on Bianbu K3

Install SpineRuntime and build tools on a K3 board:

```console
$ sudo apt update
$ sudo apt install -y git spacemit-runtime g++ cmake pkg-config
```

Clone this repository:

```console
$ git clone https://github.com/spacemit-com/spine-runtime.git
$ cd spine-runtime
```

Build and run the quickstart example:

```console
$ cmake -S examples/quickstart -B build/quickstart
$ cmake --build build/quickstart --parallel
$ ./build/quickstart/spine_quickstart
```

Expected output:

```text
SpineRuntime quickstart passed on N core(s)
```

`N` is the number of compute cores granted to the Stream. It can vary depending
on Backend selection and current compute-core availability.

For cross-compilation and SDK-based builds, see
[`examples/quickstart`](examples/quickstart).

## SDK Contents

The Linux SDK package contains public headers, the prebuilt runtime library,
CMake package files, `pkg-config` metadata, version metadata, and license
information.

| Path | Purpose |
|---|---|
| `include/spert.hpp` | Public C++17 programming API |
| `include/spert_engine.hpp` | Runtime engine declarations distributed with the SDK |
| `include/spert_abi.h` | Compiler-integration C ABI |
| `lib/libspert.so` | Prebuilt SpineRuntime shared library |
| `lib/cmake/spine-runtime/` | CMake package files |
| `lib/pkgconfig/spine-runtime.pc` | `pkg-config` metadata |
| `manifest.json` | Package metadata |
| `VERSION_NUMBER` | SDK version |
| `LICENSE` | SDK license information |

The SDK does not contain the Runtime implementation source code, runtime test
code, or example applications. Examples are maintained in this repository.

Applications should use headers and `libspert` from the same SDK release.

## CMake Integration

Use the extracted SDK root as `CMAKE_PREFIX_PATH`.

```console
cmake -S /path/to/your/application \
  -B /path/to/your/application/build \
  -DCMAKE_PREFIX_PATH="$SPINE_RUNTIME_SDK"
```

The SDK CMake package provides the `spine-runtime::spert` target.

## pkg-config Integration

Add the SDK metadata directory to `PKG_CONFIG_PATH`.

```console
export PKG_CONFIG_PATH="$SPINE_RUNTIME_SDK/lib/pkgconfig${PKG_CONFIG_PATH:+:$PKG_CONFIG_PATH}"
pkg-config --modversion spine-runtime
pkg-config --cflags --libs spine-runtime
```

## Backend Selection

By default, SpineRuntime selects an available Backend automatically. For
functional validation without physical SpacemiT AI compute cores, use
`generic/qemu`.

```console
export SPERT_BACKEND=generic/qemu
```

Physical-platform detection takes precedence over this environment variable.

## Documentation

| Document | Description |
|---|---|
| [Architecture](docs/architecture.md) | SPINE compiler and runtime stack |
| [Programming Model](docs/programming-model.md) | Tile-SPMD, `Grid`, `Context`, `Stream`, `Future`, synchronization, and shared buffers |
| [Scheduling](docs/scheduling.md) | Core allocation, tile scheduling, cooperation, and resource contention |
| [API Reference](docs/api-reference.md) | Public C++ API, status codes, and compiler integration ABI |
| [Compatibility](docs/compatibility.md) | SDK, ABI, platform, Backend, and release compatibility |

## What SpineRuntime Provides

- C++17 API with typed launch arguments, RAII handles, and `enum class Status`.
- 1D, 2D, and 3D Tile-SPMD execution through `Grid` and `Context`.
- Stream-level compute-core allocation through Backend-managed resource grants.
- Asynchronous execution through `Future`, timed waits, and combined waits.
- Cooperative synchronization through grid synchronization, `Barrier`, `Event`,
  `spawn`, `join`, and `yield`.
- Per-core shared buffers for compute-core scratchpad access.
- Multiple Backend modes for SpacemiT hardware and `generic/qemu`.
- Compiler integration ABI for generated dispatch code.

## Project Scope

This repository does not contain the SpineRuntime implementation source code.

It contains:

- public documentation;
- integration examples;
- release package information;
- issue and contribution entry points.

The Runtime implementation is distributed as prebuilt SDK packages through
GitHub Releases or platform package channels.

## SPINE Stack

SPINE is SpacemiT's AI compiler and runtime stack for RISC-V processors.

| Repository | Role |
|---|---|
| [spine-triton](https://github.com/spacemit-com/spine-triton) | Triton compiler and RISC-V CPU backend |
| [spine-mlir](https://github.com/spacemit-com/spine-mlir) | MLIR-based compiler infrastructure |
| [spine-runtime](https://github.com/spacemit-com/spine-runtime) | Runtime infrastructure for launching and executing kernels |

## Troubleshooting

If CMake cannot find SpineRuntime, check that `CMAKE_PREFIX_PATH` points to the
SDK root, not to `include/` or `lib/`.

If `pkg-config` cannot find SpineRuntime, check that `PKG_CONFIG_PATH` points
to the directory containing `spine-runtime.pc`.

If the executable cannot load `libspert.so`, configure the target system's
dynamic-library search path or install the runtime library through the platform
package manager.

See [Compatibility](docs/compatibility.md) for SDK and ABI requirements.

## Contributing

This repository accepts documentation fixes, integration examples, issue
reports, and release packaging feedback.

Runtime implementation source code is not maintained in this public repository.

## License

See [LICENSE](LICENSE).
