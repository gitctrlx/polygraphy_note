# Working With Models With Dynamic Shapes In TensorRT

在TensorRT中处理具有动态形状的模型

## 简介

为了在TensorRT中使用动态输入形状，我们必须在构建引擎时指定可能的形状范围（或多个范围）。
有关工作原理的详细信息，请参阅[API示例07](../../../api/07_tensorrt_and_dynamic_shapes/)。

在使用CLI时，我们可以为每个输入指定一次或多次最小、最佳和最大形状。如果为每个输入指定了多次形状，将创建多个优化配置文件。

## 运行示例

1. 使用3个单独的配置文件构建引擎：

   ```bash
   polygraphy convert dynamic_identity.onnx -o dynamic_identity.engine \
       --trt-min-shapes X:[1,3,28,28] --trt-opt-shapes X:[1,3,28,28] --trt-max-shapes X:[1,3,28,28] \
       --trt-min-shapes X:[1,3,28,28] --trt-opt-shapes X:[4,3,28,28] --trt-max-shapes X:[32,3,28,28] \
       --trt-min-shapes X:[128,3,28,28] --trt-opt-shapes X:[128,3,28,28] --trt-max-shapes X:[128,3,28,28]
   ```

   对于具有多个输入的模型，只需为每个`--trt-*-shapes`参数提供多个参数。例如：`--trt-min-shapes input0:[10,10] input1:[10,10] input2:[10,10] ...`

   *提示：如果我们只想使用一个最小==最佳==最大的配置文件，我们可以使用运行时输入形状选项：`--input-shapes`作为设置最小/最佳/最大的简便方法。*


2. **[可选]** 检查生成的引擎：

   ```bash
   polygraphy inspect model dynamic_identity.engine
   ```

## 进一步阅读

有关在TensorRT中使用动态形状的更多信息，请参阅[开发者指南](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#work_dynamic_shapes)。
