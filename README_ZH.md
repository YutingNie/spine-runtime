# SpineRuntime

[English](README.md) | 简体中文

**SpacemiT RISC-V AI 众核执行运行时**

SpineRuntime（库名 `spert`）面向 SpacemiT RISC-V SoC，负责在宿主通用核与 AI 计算核之间组织并行任务。SDK 以预编译动态库 `libspert`、C++ 公共头文件和编译器集成 ABI 的形式交付；应用只需描述计算网格与 tile kernel，运行时负责 Backend 选择、计算核资源申请、任务调度、同步和资源回收。

## 简介

SpineRuntime 提供 Tile-SPMD（Single Program, Multiple Data）编程模型：一次 `launch` 将同一个 kernel 实例化为一组 tile，每个 tile 处理网格中的一个坐标。应用无需直接管理 AI 计算核线程，可通过统一接口在不同 SpacemiT 平台和 generic/qemu 环境中运行相同的调度代码。

核心特性包括：

- **现代 C++17 API**：使用强类型参数、RAII 句柄和 `enum class Status`，无需手工封装 `void*` 参数。
- **1D/2D/3D Tile-SPMD**：通过 `Grid` 描述并行空间，通过 `Context` 查询 tile 坐标和网格维度。
- **Stream 级多核执行**：每个 `Stream` 持有一组 Backend 授予的计算核资源，提交到该 Stream 的 tile 只在对应资源范围内执行。
- **异步任务与组合等待**：`launch` 返回 `Future`，支持单任务等待、超时等待和多个任务的合并等待。
- **tile 内协作**：提供全网格同步、Barrier、Event、子任务 `spawn/join` 和协作式 `yield`。
- **每核共享缓冲区**：kernel 可访问当前计算核的共享暂存区，也可按需申请和释放 tile 私有片段。
- **多 Backend 共存**：支持自动选择默认 Backend，也支持通过名称获取独立的 `BackendHandle`。
- **稳定集成边界**：模板参数在应用侧完成类型封装，预编译运行时通过非模板接口接收任务；二进制兼容范围以匹配的 SDK 头文件和 `libspert` SONAME 主版本为边界。

整体使用关系如下：

```text
宿主应用 / 编译器生成代码
           │
           ├── C++ API：spert.hpp
           └── 集成 ABI：spert_abi.h
                       │
                 BackendHandle
                       │
          ┌────────────┴────────────┐
        Stream A                  Stream B
          │                         │
     Grid → Tiles              Grid → Tiles
          │                         │
     获授予的 CC Core          获授予的 CC Core
```

## 快速开始

quickstart demo 是 [`examples/quickstart`](examples/quickstart) 下的独立 CMake 工程，其 README 介绍 K3 原生编译与使用预编译 SDK 交叉编译两种流程。

## 编程模型

本节内容已移至 [docs/programming-model_ZH.md](docs/programming-model_ZH.md)。

## 多核调度模型

本节内容已移至 [docs/scheduling_ZH.md](docs/scheduling_ZH.md)。

## API 与功能介绍

本节内容已移至 [docs/api-reference_ZH.md](docs/api-reference_ZH.md)。

## 支持的 Backend 模式

SpineRuntime 支持自动选择和显式选择两种方式：

```cpp
// 自动探测真实平台；未探测到实板时使用 generic/qemu。
spert::BackendHandle automatic = spert::default_backend();

// 按名称显式获取 Backend。
spert::BackendHandle k1 = spert::backend("spacemit-k1");
spert::BackendHandle k3 = spert::backend("spacemit-k3");
spert::BackendHandle generic = spert::backend("generic/qemu");
```

| Backend 名称 | 目标平台 / 用途 | 默认计算核拓扑 | 每核共享缓冲区 | 选择方式 |
|---|---|---|---:|---|
| `spacemit-k1` | SpacemiT K1 AI 计算核 | CC Core `{0, 1, 2, 3}` | 128 KiB | K1 实板自动选择，或按名称显式选择。 |
| `spacemit-k3` | SpacemiT K3 A100 AI 计算核 | CC Core `{8, 9, 10, 11, 12, 13, 14, 15}` | 384 KiB | K3 实板自动选择，或使用 `spacemit-k3` / `spacemit` 显式选择。 |
| `spacemit-k3-x100` | K3 X100 核集的兼容执行模式 | Core `{4, 5, 6, 7}` | 384 KiB 等价回退区 | 按名称显式选择，或作为 generic 模拟目标。 |
| `generic/qemu` | 普通 Linux、qemu-user 和无实板环境下的功能开发与验证 | 逻辑 Core `0..N-1`，由模拟目标决定 | 跟随模拟目标 | 未探测到实板时自动选择，或使用 `generic` / `generic/qemu` 显式选择。 |

在未探测到真实 SpacemiT 平台时，generic/qemu 默认模拟 `spacemit-k3` 的核数量与硬件信息。可在进程启动前通过 `SPERT_BACKEND` 选择 `spacemit-k1`、`spacemit-k3` 或 `spacemit-k3-x100` 作为模拟目标。真实平台探测结果优先于该环境变量。

generic/qemu 用于验证编程模型、调度流程和 API 集成，不代表真实 AI 计算核的性能、绑核效果或 TCM 行为。应用可通过 `backend_info()` 在运行时查询最终可见的核数量、共享缓冲区容量、向量长度和架构 ID，避免把平台参数写死在业务代码中。
