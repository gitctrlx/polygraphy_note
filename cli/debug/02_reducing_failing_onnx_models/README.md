# Reducing Failing ONNX Models

减少失败的ONNX模型

## 简介

当模型因任何原因失败时（例如，在TensorRT中出现准确性问题），通常有必要将其缩减到触发故障的最小可能子图。这样可以更容易地确定故障的原因。

一种处理方法是生成原始ONNX模型的逐渐减小的子图。在每次迭代中，我们可以检查子图是否正常工作或仍然失败；一旦我们有一个正常工作的子图，就知道前一个迭代生成的子图是最小的失败子图。

`debug reduce` 子工具允许我们自动化这个过程。

## 运行示例

为了本示例的目的，我们假设我们的模型（`./model.onnx`）在TensorRT中存在准确性问题。由于实际上模型在TensorRT中可以工作（如果不行，请报告一个错误！），我们将概述通常运行的命令，然后是您可以运行的命令，以模拟失败以了解工具在实践中的外观。

我们的模拟失败将在模型中存在 `Mul` 节点时触发：

![./model.png](./model.png)

因此，最终的减少模型应只包含 `Mul` 节点（因为其他节点不会引起失败）。

1. 对于使用动态输入形状或包含形状操作的模型，需要冻结输入形状并折叠形状操作：
    
    ```bash
    polygraphy surgeon sanitize model.onnx -o folded.onnx --fold-constants \
        --override-input-shapes x0:[1,3,224,224] x1:[1,3,224,224]
    ```
    
2. 假设ONNX-Runtime为我们提供了正确的输出。我们将从头生成网络中每个张量的黄金值。我们还会保存我们使用的输入：
    
    ```bash
    polygraphy run folded.onnx --onnxrt \
        --save-inputs inputs.json \
        --onnx-outputs mark all --save-outputs layerwise_golden.json
    ```
    
    然后，我们将使用 `data to-input` 子工具将输入和逐层输出组合成单个逐层输入文件（我们将在下一步看到这是为什么必要的）：
    
    ```bash
    polygraphy data to-input inputs.json layerwise_golden.json -o layerwise_inputs.json
    ```


3. 接下来，我们将在 `debug reduce` 中使用 `bisect` 模式：

    ```bash
    polygraphy debug reduce folded.onnx -o initial_reduced.onnx --mode=bisect --load-inputs layerwise_inputs.json \
        --check polygraphy run polygraphy_debug.onnx --trt \
                --load-inputs layerwise_inputs.json --load-outputs layerwise_golden.json
    ```

    让我们来分解一下：

    - 与其他 `debug` 子工具一样，`debug reduce` 默认生成每次迭代的中间产物（默认情况下为 `./polygraphy_debug.onnx`）。这种情况下的产物是原始ONNX模型的某个子图。
        
    - 为了让 `debug reduce` 确定每个子图是否失败或通过，我们提供了一个 `--check` 命令。由于我们正在研究准确性问题，我们可以使用 `polygraphy run` 进行比较，与之前的黄金输出进行比较。
        
        > *提示：与其他 `debug` 子工具一样，还支持交互模式，只需省略 `--check` 参数即可使用。*

    - 在 `--check` 命令中，我们通过 `--load-inputs` 提供了逐层输入，否则 `polygraphy run` 会为子图张量生成新的输入，这些输入可能与我们生成黄金数据时这些张量的值不匹配。另一种方法是在 `debug reduce` 的每次迭代之前运行参考实现（这里是ONNX-Runtime），而不是提前运行。
        
    - 由于我们使用了非默认的输入数据，因此我们还通过 `--load-inputs` 直接提供了逐层输入给 `debug reduce` 命令（除了提供给 `--check` 命令）。在具有多个并行分支（*指的是模型中的路径而不是控制流*）的模型中，这一点很重要，比如：
        <!-- Polygraphy Test: Ignore Start -->
        
        ```
         inp0  inp1
          |     |
         Abs   Abs
            \ /
            Sum
             |
            out
        ```
        在这种情况下，`debug reduce` 需要能够将一个分支替换为常数。为此，它需要知道您正在使用的输入数据，以便可以用正确的值替换它。虽然我们在这里使用了文件，但可以通过[CLI用户指南](../../../how-to/use_custom_input_data.md)中涵盖的任何其他Polygraphy数据加载器参数提供输入数据。
        
        如果您不确定是否需要提供数据加载器，`debug reduce` 将在尝试替换分支时发出警告，如下所示：
        
        ```
        [W]     This model includes multiple branches/paths. In order to continue reducing, one branch needs to be folded away.
                Please ensure that you have provided a data loader argument to `debug reduce` if your `--check` command is using a non-default data loader.
                Not doing so may result in false negatives!
        ```
        <!-- Polygraphy Test: Ignore End -->
        
    - 我们通过 `-o` 选项指定了减小后的模型将写入 `initial_reduced.onnx`。

    **模拟失败：** 我们可以使用 `polygraphy inspect model` 与 `--fail-regex` 结合使用，以在模型包含 `Mul` 节点时触发失败：

    ```bash
    polygraphy debug reduce folded.onnx -o initial_reduced.onnx --mode=bisect \
        --fail-regex "Op: Mul" \
        --check polygraphy inspect model polygraphy_debug.onnx --show layers
    ```

4. **[可选]** 作为健全性检查，我们可以检查我们减小后的模型，以确保它确实包含 `Mul` 节点：

    ```bash
    polygraphy inspect model initial_reduced.onnx --show layers
    ```

5. 由于在上一步中使用了 `bisect` 模式，该模型可能不如它可能的那样最小。为了进一步完善它，我们将再次在 `linear` 模式下运行 `debug reduce`：
    
    ```bash
    polygraphy debug reduce initial_reduced.onnx -o final_reduced.onnx --mode=linear --load-inputs layerwise_inputs.json \
        --check polygraphy run polygraphy_debug.onnx --trt \
                --load-inputs layerwise_inputs.json --load-outputs layerwise_golden.json
    ```
    
    **模拟失败：** 我们将使用与之前相同的技术：
    
    ```bash
    polygraphy debug reduce initial_reduced.onnx -o final_reduced.onnx --mode=linear \
        --fail-regex "Op: Mul" \
        --check polygraphy inspect model polygraphy_debug.onnx --show layers
    ```
    
6. **[可选]** 在这个阶段，`final_reduced.onnx` 应该只包含失败的节点 - `Mul`。我们可以使用 `inspect model` 来验证这一点：
    
    ```bash
    polygraphy inspect model final_reduced.onnx --show layers
    ```

- ## 进一步阅读
    
    - 有关 `debug` 工具的详细信息，请参见帮助输出：`polygraphy debug -h` 和 `polygraphy debug reduce -h`。
    - 此外，还请查看 [`debug reduce` how-to guide](../../../how-to/use_debug_reduce_effectively.md)， 以获取更多信息、提示和技巧。

