# SpineRuntime

English | [简体中文](README_ZH.md)

**SpacemiT RISC-V AI Many-Core Execution Runtime**

SpineRuntime (library name: `spert`) targets SpacemiT RISC-V SoCs and coordinates parallel tasks between general-purpose host cores and AI compute cores. The SDK is distributed as the prebuilt `libspert` shared library, public C++ headers, and a compiler-integration ABI. Applications describe only the compute grid and tile kernel; the runtime handles Backend selection, compute-core resource allocation, task scheduling, synchronization, and resource reclamation.

## Introduction

SpineRuntime provides a Tile-SPMD (Single Program, Multiple Data) programming model: each `launch` instantiates the same kernel as a set of tiles, with every tile processing one coordinate in the grid. Applications do not need to manage AI compute-core threads directly. The same scheduling code runs across different SpacemiT platforms and generic/qemu environments through a unified interface.

Key features include:

- **Modern C++17 API**: Uses strongly typed parameters, RAII handles, and `enum class Status`, eliminating the need to manually wrap `void*` arguments.
- **1D/2D/3D Tile-SPMD**: Describes the parallel space with `Grid` and queries tile coordinates and grid dimensions through `Context`.
- **Stream-level multicore execution**: Each `Stream` owns a set of compute-core resources granted by the Backend. Tiles submitted to that Stream run only within the corresponding resource set.
- **Asynchronous tasks and combined waits**: `launch` returns a `Future`, supporting waits for a single task, timed waits, and combined waits for multiple tasks.
- **In-tile cooperation**: Provides whole-grid synchronization, Barriers, Events, child-task `spawn/join`, and cooperative `yield`.
- **Per-core shared buffers**: A kernel can access the current compute core's shared scratchpad or allocate and release tile-private slices on demand.
- **Multiple coexisting Backends**: Supports automatic selection of the default Backend and access to independent `BackendHandle` instances by name.
- **Stable integration boundary**: Template arguments are wrapped into types on the application side, while the prebuilt runtime receives tasks through non-template interfaces. Binary compatibility is bounded by matching SDK headers and the `libspert` SONAME major version.

The overall usage relationship is as follows:

```text
Host application / compiler-generated code
           │
           ├── C++ API: spert.hpp
           └── Integration ABI: spert_abi.h
                       │
                 BackendHandle
                       │
          ┌────────────┴────────────┐
        Stream A                  Stream B
          │                         │
     Grid → Tiles              Grid → Tiles
          │                         │
     Granted CC Cores          Granted CC Cores
```

## Quickstart

The quickstart demo is a standalone CMake project under
[`examples/quickstart`](examples/quickstart); its README covers native
builds on K3 and cross-compilation with a prebuilt SDK.

## Programming Model

This section moved to [docs/programming-model.md](docs/programming-model.md).

## Multicore Scheduling Model

This section moved to [docs/scheduling.md](docs/scheduling.md).

## API and Feature Overview

This section moved to [docs/api-reference.md](docs/api-reference.md).

## Supported Backend Modes

SpineRuntime supports both automatic and explicit selection:

```cpp
// Automatically detect real hardware; use generic/qemu if none is detected.
spert::BackendHandle automatic = spert::default_backend();

// Explicitly obtain a Backend by name.
spert::BackendHandle k1 = spert::backend("spacemit-k1");
spert::BackendHandle k3 = spert::backend("spacemit-k3");
spert::BackendHandle generic = spert::backend("generic/qemu");
```

| Backend Name | Target Platform / Purpose | Default Compute-Core Topology | Per-Core Shared Buffer | Selection Method |
|---|---|---|---:|---|
| `spacemit-k1` | SpacemiT K1 AI compute cores | CC Core `{0, 1, 2, 3}` | 128 KiB | Selected automatically on K1 hardware, or explicitly by name. |
| `spacemit-k3` | SpacemiT K3 A100 AI compute cores | CC Core `{8, 9, 10, 11, 12, 13, 14, 15}` | 384 KiB | Selected automatically on K3 hardware, or explicitly through `spacemit-k3` / `spacemit`. |
| `spacemit-k3-x100` | Compatible execution mode on the K3 X100 core set | Core `{4, 5, 6, 7}` | 384 KiB equivalent fallback | Selected explicitly by name, or used as a generic simulation target. |
| `generic/qemu` | Functional development and validation on standard Linux, qemu-user, or systems without hardware | Logical Core `0..N-1`, determined by the simulation target | Follows the simulation target | Selected automatically when no hardware is detected, or explicitly through `generic` / `generic/qemu`. |

When no physical SpacemiT platform is detected, generic/qemu simulates the core count and hardware information of `spacemit-k3` by default. Before starting the process, set `SPERT_BACKEND` to select `spacemit-k1`, `spacemit-k3`, or `spacemit-k3-x100` as the simulation target. Physical-platform detection takes precedence over this environment variable.

generic/qemu is intended for validating the programming model, scheduling flow, and API integration; it does not represent the performance, core-affinity behavior, or TCM behavior of physical AI compute cores. Applications can query the final visible core count, shared-buffer capacity, vector length, and architecture ID at runtime through `backend_info()` instead of hard-coding platform parameters in application logic.
