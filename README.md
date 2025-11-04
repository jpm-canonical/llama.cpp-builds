# llama.cpp builds

This repository contains a workflow to create builds of llama.cpp from the upstream source at https://github.com/ggml-org/llama.cpp


## AMD64 architecture variants

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
