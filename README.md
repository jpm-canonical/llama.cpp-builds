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

## Snapdragon Hexagon artifact

The `llamacpp-arm64+hexagon.tar.gz` artifact is built for Linux on Snapdragon
using Qualcomm's `arm64-linux:v0.7` toolchain image. It contains the CPU-side
Hexagon backend and HTP skel libraries for the supported Hexagon versions.

On the target device, set the library paths before running llama.cpp:

```shell
export LD_LIBRARY_PATH="$PWD/lib"
export ADSP_LIBRARY_PATH="$PWD/lib"
export GGML_HEXAGON_DEVICES=HTP0
./bin/llama-server --device HTP0 --n-gpu-layers 99 --model model.gguf
```

The target must provide the Qualcomm FastRPC userspace driver and CDSP device
access. The artifact does not bundle proprietary host driver libraries.

The current Hexagon backend supports these model weight types for HTP matmul
offload: `Q4_0`, `Q4_1`, `Q8_0`, `IQ4_NL`, and `MXFP4`. K-quants such as
`Q4_K_M` are not HTP-compatible in this backend and may fall back to the CPU.
Model size and layer placement still need to be validated on the target
device.

Inference snaps should consume those platform dependencies through a trusted
runtime provider rather than reading host `/usr` paths directly. The
application-facing contract used by the Gemma HTP engine is:

- `inference-npu`: a gadget-provided `custom-device` slot for the FastRPC CDSP
  and DMA-heap nodes.
- `inference-npu-runtime-qcom-htp`: a versioned content provider exposing the
  matching `libcdsprpc.so` runtime and QCOM DSP configuration/files.

For a QCS8300-class target, the runtime provider must expose the equivalent of
`/usr/share/qcom/conf.d` and the matching CDSP shell files. The gadget remains
responsible for firmware, udev permissions, and the device slot. These logical
contracts are deliberately platform-neutral so other NPU implementations can
reuse the same application snap interface.

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
