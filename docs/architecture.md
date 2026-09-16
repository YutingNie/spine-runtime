# SPINE Architecture

SPINE is SpacemiT's AI compiler and runtime stack for RISC-V CPUs, 
covering the path from Triton kernel development to execution on SpacemiT hardware.

```text
                 AI Model / PyTorch
                         │
                         ▼
                  Triton Kernel
                         │
                         ▼
              ┌─────────────────────┐
              │    spine-triton     │
              │ Compiler + Backend  │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │     spine-mlir      │
              │ Compiler Infra      │
              └──────────┬──────────┘
                         │
                         ▼
                    LLVM / RISC-V
                         │
                         ▼
                 Compiled Kernel
                         │
                         ▼
              ┌─────────────────────┐
              │    spine-runtime   │
              │ Runtime + Execution │
              └──────────┬──────────┘
                         │
                         ▼
                  SpacemiT CPUs
                   K1 / K3 / ...
```

| Repository                                                       | Role                                                                |
| ---------------------------------------------------------------- | ------------------------------------------------------------------- |
| [`spine-triton`](https://github.com/spacemit-com/spine-triton)   | Triton compiler and RISC-V CPU backend for AI kernels               |
| [`spine-mlir`](https://github.com/spacemit-com/spine-mlir)       | MLIR-based compiler infrastructure for the SPINE stack              |
| [`spine-runtime`](https://github.com/spacemit-com/spine-runtime) | Runtime infrastructure for launching and executing compiled kernels |
