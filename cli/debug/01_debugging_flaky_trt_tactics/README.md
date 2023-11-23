# Debugging Flaky TensorRT Tactics

调试不稳定的TensorRT策略


## 简介

有时，TensorRT中的某个策略可能会产生不正确的结果，或者表现出其他的错误行为。由于TensorRT构建器依赖于计时策略，因此引擎构建是不确定性的，这可能会导致策略错误表现为不稳定或间歇性的失败。

解决问题的一种方法是多次运行构建器，保存每次运行的策略重放文件。一旦我们有了一组已知好和已知坏的策略，我们就可以将它们进行比较，以确定哪个策略可能是错误的根源。

`debug build` 子工具允许您自动化这个过程。

有关`debug`工具的详细信息，请参见帮助输出：`polygraphy debug -h` 和 `polygraphy debug build -h`。


## 运行示例

1. 从ONNX-Runtime生成黄金输出：

   ```bash
   polygraphy run identity.onnx --onnxrt \
       --save-outputs golden.json
   ```

2. 使用 `debug build` 多次构建TensorRT引擎，并将结果与黄金输出进行比较，每次保存一个策略重放文件：

   ```bash
   polygraphy debug build identity.onnx --fp16 --save-tactics replay.json \
       --artifacts-dir replays --artifacts replay.json --until=10 \
       --check polygraphy run polygraphy_debug.engine --trt --load-outputs golden.json
   ```

   让我们来分解一下：

   - 与其他 `debug` 子工具一样，`debug build` 默认生成每次迭代的中间产物（`./polygraphy_debug.engine`）。 在这种情况下，这个产物是一个TensorRT引擎。

     *提示：`debug build` 支持所有其他工具支持的TensorRT构建器配置选项，比如 `convert` 或 `run`。*

   - 为了让 `debug build` 确定每个引擎是失败还是成功，我们提供了一个 `--check` 命令。 由于我们正在查看一个（虚假的）准确性问题，我们可以使用 `polygraphy run` 来比较引擎的输出和我们的黄金值。

     *提示：与其他 `debug` 子工具不同，`debug build` 没有自动终止条件，因此我们需要提供 `--until` 选项，以便工具知道何时停止。 这可以是迭代次数，也可以是 "good" 或 "bad"。 在后一种情况下，工具将在找到第一个通过或失败的迭代后停止。

   - 由于我们最终要比较好和坏的策略重放，我们指定 `--save-tactics` 以保存每次迭代的策略重放文件，然后使用 `--artifacts` 告诉 `debug build` 来管理它们，这涉及将它们分类到主要的工件目录下的 `good` 和 `bad` 子目录中，这个目录由 `--artifacts-dir` 指定。


3. 使用 `inspect diff-tactics` 确定哪些策略可能有问题：

   ```bash
   polygraphy inspect diff-tactics --dir replays
   ```

   *注意：最后一步应该报告无法确定可能有问题的策略，因为我们的 `bad` 目录此时应该是空的（否则请报告TensorRT问题！）：*

   <!-- Polygraphy Test: Ignore Start -->

   ```
   [I] Loaded 2 good tactic replays.
   [I] Loaded 0 bad tactic replays.
   [I] Could not determine potentially bad tactics. Try generating more tactic replay files?
   ```

   <!-- Polygraphy Test: Ignore End -->


## 进一步阅读

有关 `debug` 工具的更多信息，以及适用于所有 `debug` 子工具的提示和技巧，请参阅 [how-to guide for `debug` subtools](../../../how-to/use_debug_subtools_effectively.md)。
