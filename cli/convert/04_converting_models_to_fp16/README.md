# Converting ONNX Models To FP16

# 将ONNX模型转换为FP16

## 简介

当使用TensorRT降低精度优化（`--fp16`和`--tf32`标志）在FP32中训练的ONNX模型时，用于调试精度问题时，将模型转换为FP16并在ONNX-Runtime下运行可以帮助检查是否可能存在与降低精度运行模型相关的问题。

## 运行示例

1. 将模型转换为FP16：

   ```bash
   polygraphy convert --fp-to-fp16 -o identity_fp16.onnx identity.onnx
   ```

2. **[可选]** 检查生成的模型：

   ```bash
   polygraphy inspect model identity_fp16.onnx
   ```

3. **[可选]** 在ONNX-Runtime下运行FP32和FP16模型，然后比较结果：

   ```bash
   polygraphy run --onnxrt identity.onnx \
      --save-inputs inputs.json --save-outputs outputs_fp32.json
   ```

   ```bash
   polygraphy run --onnxrt identity_fp16.onnx \
      --load-inputs inputs.json --load-outputs outputs_fp32.json \
      --atol 0.001 --rtol 0.001
   ```

4. **[可选]** 检查FP16模型的任何中间输出是否包含NaN或无穷大（请参阅[检查中间NaN或无穷大](../../../cli/run/07_checking_nan_inf)）：

   ```bash
   polygraphy run --onnxrt identity_fp16.onnx --validate
   ```

## 另请参阅

* [跨运行比较](../../../cli/run/02_comparing_across_runs)
* [检查中间NaN或无穷大](../../../cli/run/07_checking_nan_inf)
* [调试TensorRT精度问题](../../../how-to/debug_accuracy.md)
