# Deterministic Engine Building In TensorRT

在TensorRT中进行确定性引擎构建.

## 简介

在引擎构建期间，TensorRT会运行和计时多个内核以选择最优的内核。由于内核的计时可能在每次运行时略有不同，因此这个过程本质上是不确定的。

在许多情况下，可能希望进行确定性引擎构建。实现这一目标的一种方法是使用`IAlgorithmSelector` API，以确保每次都选择相同的内核。

为了简化这个过程，Polygraphy提供了两个内置的算法选择器：`TacticRecorder`和`TacticReplayer`。前者可用于记录引擎构建过程中选择的策略，而后者可用于在后续构建中播放这些策略。CLI工具包括与这些选项对应的`--save-tactics`和`--load-tactics`选项。

## 运行示例

1. 构建一个引擎并保存一个回放文件：

   ```bash
   polygraphy convert identity.onnx \
       --save-tactics replay.json \
       -o 0.engine
   ```

   生成的`replay.json`文件是人类可读的。可选地，我们可以使用`inspect tactics`以友好的格式查看它：

   ```bash
   polygraphy inspect tactics replay.json
   ```

2. 使用回放文件进行另一个引擎构建：

   ```bash
   polygraphy convert identity.onnx \
       --load-tactics replay.json \
       -o 1.engine
   ```

3. 验证引擎完全相同：

   ```bash
   diff -sa 0.engine 1.engine
   ```
