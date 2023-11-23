# Modifying Input Shapes

修改输入形状.

## 简介

`surgeon sanitize`子工具可用于修改ONNX模型的输入形状。这不会更改模型的中间层，因此如果模型对输入形状有假设（例如，具有硬编码新形状的`Reshape`节点），可能会导致问题。

输出形状可以被推断出来，因此不会被修改（也不需要被修改）。

*注意: 强烈建议重新导出具有所需形状的ONNX模型。只有在不可能重新导出模型时才应使用此方法。*

## 运行示例

1. 将模型的输入形状更改为具有动态批次维度的形状，保持其他维度不变：

   ```bash
   polygraphy surgeon sanitize identity.onnx \
       --override-input-shapes x:['batch',1,2,2] \
       -o dynamic_identity.onnx
   ```

2. **[可选]** 您可以使用 `inspect model` 来确认它是否看起来正确：

   ```bash
   polygraphy inspect model dynamic_identity.onnx --show layers
   ```
