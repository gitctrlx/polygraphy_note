
# Generating A Script For Advanced Comparisons


## Introduction

对于更高级的需求，您可能想使用 API。您可以使用 `run` 的 `--gen-script` 选项来创建一个 Python 脚本，作为您的起点，而不是从头开始写脚本。


## Running The Example

1. 生成一个比较脚本：

    ```bash
    polygraphy run identity.onnx --trt --onnxrt \
        --gen-script=compare_trt_onnxrt.py
    ```

    生成的脚本将完全执行 `run` 命令原本会做的事情。

2. 运行比较脚本，在此之前可以选择修改它：

    ```bash
    python3 compare_trt_onnxrt.py
    ```
