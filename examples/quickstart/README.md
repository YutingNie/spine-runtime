# SpineRuntime Quickstart

This example launches an eight-tile grid. Each tile writes the square of its
tile ID to an output buffer. The host waits for completion and verifies the
result.

The source and CMake project in this directory were moved from the main
SpineRuntime README so that the example can be built directly.

## Files

```text
examples/quickstart/
├── CMakeLists.txt
├── demo.cpp
└── README.md
```

## Option 1: Cross-compile with a Prebuilt SDK

Use this path on a development host that has a RISC-V cross toolchain.

Download and extract the RISC-V 64-bit Linux SDK from the
[SpineRuntime Releases](https://github.com/spacemit-com/spine-runtime/releases)
page.

For release `0.6.2`:

```console
$ curl -LO https://github.com/spacemit-com/spine-runtime/releases/download/0.6.2/spine-runtime.riscv64.0.6.2.tar.gz
$ echo "fc062cc83a49c98011976fa9620739841c74a619379785172d0f8959df65be47  spine-runtime.riscv64.0.6.2.tar.gz" | sha256sum --check
spine-runtime.riscv64.0.6.2.tar.gz: OK
$ tar -xf spine-runtime.riscv64.0.6.2.tar.gz
$ export SPINE_RUNTIME_SDK="$PWD/spine-runtime.riscv64.0.6.2"
```

From the repository root, point CMake at both the RISC-V toolchain file and the
extracted SDK:

```console
$ cmake -S examples/quickstart \
    -B build-k3/quickstart \
    -DCMAKE_TOOLCHAIN_FILE=/absolute/path/to/riscv64-toolchain.cmake \
    -DCMAKE_PREFIX_PATH="$SPINE_RUNTIME_SDK"
$ cmake --build build-k3/quickstart --parallel
```

The toolchain file and sysroot are toolchain-specific.

Before running the executable, the K3 target must provide an ABI-compatible
`libspert`. You can install the board package:

```console
$ export K3_HOST=root@k3-board-address
$ ssh "$K3_HOST" 'apt install -y spacemit-runtime'
$ scp build-k3/quickstart/spine_quickstart "$K3_HOST":/tmp/
$ ssh "$K3_HOST" /tmp/spine_quickstart
SpineRuntime quickstart passed on 8 core(s)
```

Confirm that the board package's `libspert` SONAME and API are compatible with
the SDK used for cross-compilation. If they are not compatible, deploy
`libspert` from the same SDK archive and configure the board's dynamic-library
search path.

This cross-compilation workflow is an integration template. It requires a
separately provided RISC-V toolchain and matching sysroot.

## Option 2: Build Natively on K3

On a K3 board running Bianbu, install SpineRuntime and the native build tools.

Run the package command as `root`, or prefix it with `sudo`. Refresh the APT
package index first if necessary.

```console
$ apt install -y spacemit-runtime g++ pkg-config
```

From the repository root, compile the example with `pkg-config`:

```console
$ g++ -std=c++17 examples/quickstart/demo.cpp \
    $(pkg-config --cflags --libs spine-runtime) \
    -pthread \
    -o spine_quickstart
$ ./spine_quickstart
SpineRuntime quickstart passed on 8 core(s)
```

This native workflow was verified on a Bianbu 4.0.2 RISC-V 64-bit K3 board
with `spacemit-runtime 0.6.0+1`.

The reported core count is the number actually granted to the Stream. It can
vary when other Streams or processes are using K3 compute cores.
