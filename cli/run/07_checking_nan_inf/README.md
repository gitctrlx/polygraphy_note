# Checking for Intermediate NaN or Infinities

## Introduction

在 Polygraphy 中调试模型精度问题时，检查逐层输出以发现潜在问题可能会有所帮助。Polygraphy 的 `run` 子工具提供了一个有用的标志 `--validate`，它可以快速诊断出问题的中间输出。

这个示例演示了如何使用这个标志与一个故意生成无限输出的模型，通过在输入张量中添加无穷大。

## Running The Example

 <!-- Polygraphy Test: XFAIL Start -->
```bash
polygraphy run add_infinity.onnx --onnx-outputs mark all --onnxrt --validate
```
 <!-- Polygraphy Test: XFAIL End -->

 <!-- Polygraphy Test: Ignore Start -->
您应该会看到如下输出：

```
[I] onnxrt-runner-N0-05/13/22-22:35:48  | Completed 1 iteration(s) in 0.1326 ms | Average inference time: 0.1326 ms.
[I] Output Validation | Runners: ['onnxrt-runner-N0-05/13/22-22:35:48']
[I]     onnxrt-runner-N0-05/13/22-22:35:48  | Validating output: B (check_inf=True, check_nan=True)
[I]         mean=inf, std-dev=nan, var=nan, median=inf, min=inf at (0,), max=inf at (0,), avg-magnitude=inf
[E]         Inf Detected | One or more non-finite values were encountered in this output
[I]         Note: Use -vv or set logging verbosity to EXTRA_VERBOSE to display non-finite values
[E]         FAILED | Errors detected in output: B
[E]     FAILED | Output Validation
```
 <!-- Polygraphy Test: Ignore End -->

```
[I] onnxrt-runner-N0-05/13/22-22:35:48  | 完成了 1 次迭代，用时 0.1326 毫秒 | 平均推理时间：0.1326 毫秒。
[I] 输出验证 | 运行器：['onnxrt-runner-N0-05/13/22-22:35:48']
[I]     onnxrt-runner-N0-05/13/22-22:35:48  | 正在验证输出：B (check_inf=True, check_nan=True)
[I]         平均值=无穷大, 标准差=nan, 方差=nan, 中位数=无穷大, 最小值=无穷大 在 (0,)，最大值=无穷大 在 (0,)，平均幅度=无穷大
[E]         检测到无穷大 | 这个输出中遇到了一个或多个非有限值
[I]         注意：使用 -vv 或将日志详细级别设置为 EXTRA_VERBOSE 来显示非有限值
[E]         失败 | 输出中检测到错误：B
[E]     失败 | 输出验证
```



## See Also

* [Debugging TensorRT Accuracy Issues](../../../how-to/debug_accuracy.md)
