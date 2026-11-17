# llama.cpp builds

This repository contains a workflow to create builds of llama.cpp from the upstream source at https://github.com/ggml-org/llama.cpp

## Build

To trigger a new build, the maintainers should create a new [release](https://github.com/jpm-canonical/llama.cpp-builds/releases) with a tag matching exactly a tag from the llama.cpp [releases](https://github.com/ggml-org/llama.cpp/releases).
This triggers a Github Workflow which builds and adds the artifacts one-by-one to the corresponding release.

## Usage

1. Install the dependency:
   ```
   sudo apt install libgomp1
   ```
   libgomp is required for detecting and loading a backend.
   
3. Download an artifact from [releases](https://github.com/ggml-org/llama.cpp/releases)
4. Extract it
5. Run a binary, e.g. `llama-cli` as follows:
   ```shell
   LD_LIBRARY_PATH=$LD_LIBRARY_PATH:lib bin/llama-cli --help
   ```

## AMD64 architecture variants

Ubuntu supports AMD64 architecture variants as announced in [this](https://ubuntu.com/blog/optimising-ubuntu-performance-on-amd64-architecture) blog post and discussed in [this](https://discourse.ubuntu.com/t/introducing-architecture-variants-amd64v3-now-available-in-ubuntu-25-10/71312) forum post.

Definition of flags that are supported by each variant on the architecture is available [here](https://git.launchpad.net/ubuntu/+source/dpkg/tree/data/varianttable).

Available build flags for CPU builds of llama.cpp is defined [here](https://github.com/ggml-org/llama.cpp/blob/1f5accb8d0056e6099cd5b772b1cb787dd590a13/ggml/CMakeLists.txt#L157).

| Build Flag         | amd64v1 | amd64v2 | amd64v3 | amd64v4 |
|--------------------|---------|---------|---------|---------|
| GGML_SSE42         | OFF     | ON      | ON      | ON      |
| GGML_AVX           | OFF     | OFF     | ON      | ON      |
| GGML_AVX_VNNI      | OFF     | OFF     | OFF     | OFF     |
| GGML_AVX2          | OFF     | OFF     | ON      | ON      |
| GGML_BMI2          | OFF     | OFF     | ON      | ON      |
| GGML_AVX512        | OFF     | OFF     | OFF     | ON      |
| GGML_AVX512_VBMI   | OFF     | OFF     | OFF     | OFF     |
| GGML_AVX512_VNNI   | OFF     | OFF     | OFF     | OFF     |
| GGML_AVX512_BF16   | OFF     | OFF     | OFF     | OFF     |
| GGML_FMA           | OFF     | OFF     | ON      | ON      |
| GGML_F16C          | OFF     | OFF     | ON      | ON      |
| GGML_AMX_TILE      | OFF     | OFF     | OFF     | OFF     |
| GGML_AMX_INT8      | OFF     | OFF     | OFF     | OFF     |
| GGML_AMX_BF16      | OFF     | OFF     | OFF     | OFF     |

## RISC-V (riscv64+spacemit)

The `llamacpp-riscv64+spacemit.tar.gz` artifact is a **cross-build for RISC-V
SpacemiT SoCs** (Key Stone K1 / X60 and K3 / X100+A100), built with
`GGML_CPU_RISCV64_SPACEMIT=ON` following llama.cpp's
[docs/build-riscv64-spacemit.md](https://github.com/ggml-org/llama.cpp/blob/master/docs/build-riscv64-spacemit.md),
using the [SpacemiT cross toolchain](https://github.com/spacemit-com/toolchain)
(the stock Ubuntu riscv64 binutils cannot assemble the IME instructions).

It is a **single artifact with runtime hardware dispatch**:

- On SpacemiT boards running the vendor BSP kernel (Bianbu), the binaries detect
  the A100 "AI core" cluster, the IME `vmadot` matrix-extension instructions and
  TCM automatically, and engage them with no configuration (watch for
  `CPU_RISCV64_SPACEMIT:` lines in the log).
- On any other riscv64 system — including SpacemiT boards running mainline
  kernels — the same binary falls back to the portable RVV 1.0 kernels.

Runtime requirements: an **RVA23-class CPU** (the binary is built for
`rv64gcv_zfh_zvfh_zicbop_zihintpause_zba` plus the SpacemiT IME extension, which
is only executed on hardware that reports it) and `libgomp1` as for the other
artifacts.

Reference performance on a SpacemiT K3 Pico-ITX (16 GB, Bianbu 4.0.8), Qwen3.5-2B
Q4_1 (`llama-bench -t 8 -p 128 -n 64`, default flash attention `auto`): ~114 tok/s
prompt processing / ~13.5 tok/s generation on the A100 IME path, vs ~8 / ~5 tok/s
for a generic RVV build — and ~890 / ~130 tok/s on SmolLM2-135M. Flash attention
is stable on this path; with `-fa off` prompt processing drops to ~70 tok/s.
