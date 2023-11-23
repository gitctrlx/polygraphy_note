# Comparing Across Runs

## Prerequisites
要了解如何使用 `polygraphy run` 来比较不同框架输出的概述，请参阅[比较框架](../../../cli/run/01_comparing_frameworks)的示例。

## Introduction

在某些情况下，您可能需要比较不同调用 `polygraphy run` 命令的结果。这包括以下情况：

- 在不同平台上比较结果
- 在不同版本的 TensorRT 之间比较结果
- 比较具有兼容输入/输出的不同模型类型

在这个示例中，我们将演示如何使用 Polygraphy 来实现这一点。

## Running The Example

### Comparing Across Runs

1. 保存第一次运行的输入和输出值：

    ```bash
    polygraphy run identity.onnx --onnxrt \
        --save-inputs inputs.json --save-outputs run_0_outputs.json
    ```

2. 再次运行模型，这次从第一次运行中加载保存的输入和输出。保存的输入将用作当前运行的输入，保存的输出将用来与第一次运行进行比较。
    
    ```bash
    polygraphy run identity.onnx --onnxrt \
        --load-inputs inputs.json --load-outputs run_0_outputs.json
    ```
    
    `--atol/--rtol/--check-error-stat` 选项的使用与[比较框架](../../../cli/run/01_comparing_frameworks)示例中的使用相同：

    ```bash
    polygraphy run identity.onnx --onnxrt \
        --load-inputs inputs.json --load-outputs run_0_outputs.json \
        --atol 0.001 --rtol 0.001 --check-error-stat median
    ```

### Comparing Different Models

我们也可以使用这种技术来比较不同的模型，比如 TensorRT 引擎和 ONNX 模型（如果它们的输出匹配的话）。

1. 将 ONNX 模型转换为 TensorRT 引擎并将其保存到磁盘：

    ```bash
    polygraphy convert identity.onnx -o identity.engine
    ```

2. 在 Polygraphy 中运行保存的引擎，使用 ONNX-Runtime 运行的保存输入作为引擎的输入，并将引擎的输出与保存的 ONNX-Runtime 输出进行比较：
    
    ```bash
    polygraphy run --trt identity.engine --model-type=engine \
        --load-inputs inputs.json --load-outputs run_0_outputs.json
    ```


## Further Reading

有关如何使用 Python API 访问和处理保存的输出的详细信息，请参阅 [API 示例 08](../../../api/08_working_with_run_results_and_saved_inputs_manually/)。

有关与自定义输出进行比较的信息，请参阅 [`run` 示例 06](../06_comparing_with_custom_output_data/)。
