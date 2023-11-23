# Int8 Calibration In TensorRT

使用TensorRT进行Int8校准.

## 简介

在[API示例04](../../../api/04_int8_calibration_in_tensorrt/)中，我们看到了如何利用Polygraphy中包含的校准器来轻松地在TensorRT中运行int8校准。

但是，如果我们想要在命令行上执行相同的操作怎么办？

要做到这一点，我们需要一种方法来向命令行工具提供自定义输入数据。
Polygraphy提供了多种方法来实现这一点，详细信息请参见[这里](../../../how-to/use_custom_input_data.md)。

在本示例中，我们将使用一个数据加载器脚本，通过在名为`data_loader.py`的Python脚本中定义`load_data`函数，然后使用`polygraphy convert`来构建TensorRT引擎。

*TIP: 我们可以使用类似的方法来使用`polygraphy run`来构建和运行引擎。*

## 运行示例

1. 转换模型，使用自定义数据加载器脚本来提供校准数据，保存校准缓存以供将来使用：

   ```bash
   polygraphy convert identity.onnx --int8 \
       --data-loader-script ./data_loader.py \
       --calibration-cache identity_calib.cache \
       -o identity.engine
   ```

2. **[可选]** 使用缓存重新构建引擎以跳过校准：

   ```bash
   polygraphy convert identity.onnx --int8 \
       --calibration-cache identity_calib.cache \
       -o identity.engine
   ```

   由于校准缓存已经填充，将跳过校准。
   因此，我们*不*需要提供输入数据。


3. **[可选]** 直接从API示例中使用数据加载器。

   这里概述的方法非常灵活，以至于我们甚至可以使用在API示例中定义的数据加载器！
   我们只需要指定函数名称，因为示例没有称其为 `load_data`：

   ```bash
   polygraphy convert identity.onnx --int8 \
       --data-loader-script ../../../api/04_int8_calibration_in_tensorrt/example.py:calib_data \
       -o identity.engine
   ```
