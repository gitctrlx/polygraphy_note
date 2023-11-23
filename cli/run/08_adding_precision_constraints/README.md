# Adding Precision Constraints

添加精度约束。

## 引言

当使用在FP32中训练的模型构建使用[降低精度优化](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#reduced-precision)的TensorRT引擎时，模型中的某些层可能需要被约束为以FP32运行，以保持可接受的精度。

以下示例演示了如何有选择地约束网络中指定层的精度。提供的ONNX模型执行以下操作：

1. 通过右乘90度旋转的单位矩阵水平翻转其输入，
2. 将`FP16_MAX`添加到翻转后的输入，然后从结果中减去`FP16_MAX`，
3. 通过右乘旋转的单位矩阵水平翻转减法的输出。

如果`x`为正数，那么在这个过程中的步骤(2)需要以FP32进行，以获得可接受的精度，因为值将超过FP16可表示的范围（根据设计）。然而，当启用FP16优化而不进行约束时，TensorRT，由于不知道将用于`x`的值范围，通常会选择在此过程的所有步骤中以FP16运行：

* 步骤(1)和(3)中的GEMM操作在大问题规模下以FP16运行比以FP32运行更快

* 步骤(2)中的逐点操作在FP16中运行更快，并且将数据保留在FP16中可以消除额外的FP32格式转换的需要。

因此，您需要在TensorRT网络中限制允许的精度，以便在分配层精度时TensorRT可以做出适当的选择。

Polygraphy命令行工具提供多种约束层精度的方法：

1. `--layer-precisions`选项允许您设置单个层的精度。

2. 网络后处理脚本允许您以编程方式修改由Polygraphy解析或其他方式生成的TensorRT网络。

3. 网络加载器脚本允许您手动构建整个TensorRT网络，其中您可以根据需要设置层精度。

## 运行示例

**警告：**_此示例需要TensorRT 8.4或更高版本。_

### 使用`--layer-precisions`选项

运行以下命令以比较使用FP16优化的TensorRT运行模型与使用FP32的ONNX-Runtime：

<!-- Polygraphy Test: XFAIL Start -->

```bash
polygraphy run needs_constraints.onnx \
    --trt --fp16 --onnxrt --val-range x:[1,2] \
    --layer-precisions Add:float16 Sub:float32 --precision-constraints prefer \
    --check-error-stat median
```

<!-- Polygraphy Test: XFAIL End -->

为增加此命令因上述原因失败的机会，我们将强制`Add`以FP16精度运行，并将随后的`Sub`以FP32运行。这将阻止它们被融合并导致`Add`的输出超出FP16范围。

### 使用网络后处理脚本约束精度

另一个选择是使用TensorRT网络后处理脚本在解析的网络上应用精度约束。

使用提供的网络后处理脚本[add_constraints.py](./add_constraints.py)来约束模型中的精度：

```
polygraphy run needs_constraints.onnx --onnxrt --trt --fp16 --precision-constraints obey \
    --val-range x:[1,2] --check-error-stat median \
    --trt-network-postprocess-script ./add_constraints.py
```

*提示：您可以使用`--trt-npps`作为`--trt-network-postprocess-script`的缩写。*

默认情况下，Polygraphy会查找脚本中名为`postprocess`的函数以执行。要指定要使用的不同函数，请在脚本名称后缀冒号后跟函数名称，例如：

<!-- Polygraphy Test: Ignore Start -->

```
polygraphy run ... --trt-npps my_script.py:custom_func
```

<!-- Polygraphy Test: Ignore End -->

### 使用网络加载器脚本来约束精度

或者，您可以使用网络加载器脚本手动定义整个网络，其中可以设置层精度。

以下部分假定您已经阅读了关于[手动定义TensorRT网络或配置](../../../cli/run/04_defining_a_tensorrt_network_or_config_manually)的示例，并且对如何使用[TensorRT Python API](https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/index.html)有基本了解。

首先，在模型上运行ONNX-Runtime以生成参考输入和黄金输出：

```bash
polygraphy run needs_constraints.onnx --onnxrt --val-range x:[1,2] \
    --save-inputs inputs.json --save-outputs golden_outputs.json
```

接下来，运行提供的网络加载器脚本[constrained_network.py](./constrained_network.py)来约束模型中的精度，强制TensorRT遵守约束，使用保存的输入进行比较并与保存的黄金输出进行比较：

```bash
polygraphy run constrained_network.py --precision-constraints obey \
    --trt --fp16 --load-inputs inputs.json --load-outputs golden_outputs.json \
    --check-error-stat median
```

请注意，如果TensorRT没有符合请求的精度约束的

层实现，则TensorRT可能选择以FP32运行网络中的其他层，而不仅仅是明确约束的层。

**[可选]**：运行网络脚本，但如果TensorRT没有满足请求的精度约束的层实现，则允许TensorRT忽略精度约束。这可能是在运行网络时所必需的，如果TensorRT没有符合请求的精度约束的层实现：

```
polygraphy run constrained_network.py --precision-constraints prefer \
    --trt --fp16 --load-inputs inputs.json --load-outputs golden_outputs.json \
    --check-error-stat median
```

## See Also

* [使用降低精度](../../../how-to/work_with_reduced_precision.md)：有关使用Polygraphy调试降低精度优化的更一般指南。
* [手动定义TensorRT网络或配置](../../../cli/run/04_defining_a_tensorrt_network_or_config_manually)：有关如何创建网络脚本模板的说明。
* [TensorRT Python API参考](https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/index.html)。
