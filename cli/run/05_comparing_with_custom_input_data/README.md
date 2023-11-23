# Comparing With Custom Input Data

## Introduction

在某些情况下，我们可能想要使用自定义输入数据进行比较。Polygraphy 提供了多种方法来实现这一点，详细信息在[这里](../../../how-to/use_custom_input_data.md)。

在这个示例中，我们将演示两种不同的方法：

1. 使用数据加载器脚本，通过在 Python 脚本（`data_loader.py`）中定义 `load_data()` 函数。Polygraphy 将使用 `load_data()` 在运行时生成输入。
2. 使用包含预生成输入的 JSON 文件。为了方便，我们将使用上面的脚本（`data_loader.py`）将 `load_data()` 生成的输入保存到一个名为 `custom_inputs.json` 的文件中。

*提示：通常，当处理大量输入数据时，数据加载器脚本是更可取的，因为它避免了写入磁盘的需要。另一方面，JSON 文件可能更具可移植性，并且可以帮助确保可重现性。*

最后，我们将向 `polygraphy run` 提供我们的自定义输入数据，并比较 ONNX-Runtime 和 TensorRT 之间的输出。

由于我们的模型具有动态形状，我们需要设置一个 TensorRT 优化配置文件。有关如何通过命令行进行设置的详细信息，请参阅 [`convert` 示例 03](../../convert/03_dynamic_shapes_in_tensorrt)。为了简单起见，我们将创建一个 `min` == `opt` == `max` 的配置文件。

> *注意：重要的是我们的优化配置文件要与自定义数据加载器提供的形状相匹配。在我们非常简单的案例中，数据加载器总是生成形状为 (1, 2, 28, 28) 的输入，所以我们只需要确保这个形状在 [`min`, `max`] 范围内。*

## Running The Example

1. 运行脚本将输入数据保存到磁盘。 

    > *注意：这只对选项 2 是必要的。*

    ```bash
    python3 data_loader.py
    ```

2. 使用自定义输入数据运行模型，比较 TensorRT 和 ONNX-Runtime 的输出：

    - 选项 1：使用数据加载器脚本：

    ```bash
    polygraphy run dynamic_identity.onnx --trt --onnxrt \
        --trt-min-shapes X:[1,2,28,28] --trt-opt-shapes X:[1,2,28,28] --trt-max-shapes X:[1,2,28,28] \
        --data-loader-script data_loader.py
    ```

    - 选项 2：使用包含保存输入的 JSON 文件：

    ```bash
    polygraphy run dynamic_identity.onnx --trt --onnxrt \
        --trt-min-shapes X:[1,2,28,28] --trt-opt-shapes X:[1,2,28,28] --trt-max-shapes X:[1,2,28,28] \
        --load-inputs custom_inputs.json
    ```
