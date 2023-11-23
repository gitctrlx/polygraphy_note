# Comparing With Custom Output Data

## Introduction

在某些情况下，与在 Polygraphy 外部生成的输出值进行比较可能是有用的。最简单的方法是创建一个 `RunResults` 对象并将其保存到文件。

这个示例说明了如何在 Polygraphy 外部生成自定义输入和输出数据，并无缝地加载到 Polygraphy 中进行比较。

## Running The Example

1. 生成输入和输出数据：

    ```bash
    python3 generate_data.py
    ```

2. **[可选]** 检查数据。 

    对于输入：

    ```bash
    polygraphy inspect data custom_inputs.json
    ```

    对于输出：

    ```bash
    polygraphy inspect data custom_outputs.json
    ```

3. 使用生成的输入数据运行推理，然后将输出与自定义输出进行比较：

    ```bash
    polygraphy run identity.onnx --trt \
        --load-inputs custom_inputs.json \
        --load-outputs custom_outputs.json
    ```

## Further Reading

有关如何使用 Python API 访问和处理存储在 `RunResults` 对象中的输出的详细信息，请参阅 [API 示例 08](../../../api/08_working_with_run_results_and_saved_inputs_manually/)。
