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

This section moved to [docs/backends.md](docs/backends.md).
