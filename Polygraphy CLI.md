# Polygraphy 命令行工具包用户指南

## 目录

- [简介](#简介)
- [常见用例](#常见用例)
  - [检查模型](#检查模型)
  - [将模型转换为TensorRT](#将模型转换为TensorRT)
  - [清理ONNX模型](#清理ONNX模型)
  - [跨框架比较模型](#跨框架比较模型)
  - [在ONNX模型中修改输入形状](#在ONNX模型中修改输入形状)
- [高级主题](#高级主题)
  - [确定性引擎构建](#确定性引擎构建)
  - [定义自定义TensorRT网络或构建配置](#定义自定义TensorRT网络或构建配置)
  - [提取ONNX模型的子图](#提取ONNX模型的子图)
  - [调试TensorRT间歇性失败](#调试TensorRT间歇性失败)
  - [缩减失败的ONNX模型](#缩减失败的ONNX模型)
- [示例](#示例)
- [添加新工具](#添加新工具)


## 简介

Polygraphy命令行工具包包括多个工具，涵盖广泛的原型制作和调试用例。本指南提供了包含的功能的广泛概述。

有关特定工具的更多信息，请参阅这里相应目录下的README文件。

Polygraphy提供的所有工具都可以通过polygraphy二进制文件调用：`polygraphy`。

有关特定工具的使用信息，您可以参考帮助输出：`polygraphy <subtool> -h`。

> 注意：一些包含的工具仍在实验阶段。任何标记为[EXPERIMENTAL]的工具可能随时都会有向后不兼容的更改，甚至可能完全移除。

## 常见用例


### 检查模型

检查模型的各个方面，如层名称、属性或整体结构经常是有用的。`inspect model`子工具旨在以有助于程序化分析的方式提供此功能；即，它提供模型的人类可读文本表示（并且是`grep`友好的！）。

更多详情，请参考示例，其中展示了如何检查：

- [TensorRT网络](./cli/inspect/01_inspecting_a_tensorrt_network/)
- [TensorRT引擎](./cli/inspect/02_inspecting_a_tensorrt_engine/)
- [ONNX模型](./cli/inspect/03_inspecting_an_onnx_model/)

您可以在[这里](./cli/inspect)找到完整的`inspect`示例清单。


### 将模型转换为TensorRT

`convert`工具可用于将各种类型的模型转换为其他格式；例如，将ONNX模型转换为TensorRT。

有关更多信息，请参考示例，其中展示了如何：

- [将具有动态形状的模型转换为TensorRT](./cli/convert/03_dynamic_shapes_in_tensorrt/)
- [在转换过程中运行TensorRT INT8校准](./cli/convert/01_int8_calibration_in_tensorrt)

您可以在[这里](./cli/convert/)找到完整的`convert`示例清单。


### 清理ONNX模型

有时，TensorRT可能无法导入ONNX模型。在这些情况下，通常有助于清理ONNX模型，删除多余的节点和折叠常量。`surgeon sanitize`子工具使用ONNX-GraphSurgeon和ONNX-Runtime提供了这种能力。

有关更多详情，请参考[`surgeon`示例02](./cli/surgeon/02_folding_constants/)。

您可以在[这里](./cli/surgeon/)找到完整的`surgeon`示例清单。


### 跨框架比较模型

`run`工具可以运行推理并比较一个或多个框架生成的输出。这可以用来检查框架对特定模型的准确性，并在准确性较差时进行调试。

此外，它还可用于生成执行相同操作的Python脚本。

有关更多信息，请参考示例，其中展示了如何：

- [比较不同框架之间的输出](./cli/run/01_comparing_frameworks/)
- [保存并加载输出以跨运行进行比较](./cli/run/02_comparing_across_runs)
- [生成比较脚本](./cli/run/03_generating_a_comparison_script/)

您可以在[这里](./cli/run/)找到完整的`run`示例清单。


### 在ONNX模型中修改输入形状

修改ONNX模型中输入形状的最佳方式是使用所需的输入形状重新导出模型。当这不可能时，您可以使用`surgeon sanitize`工具来做到这一点。

有关更多信息，请参考[`surgeon`示例03](./cli/surgeon/03_modifying_input_shapes/)。


## 高级主题

### 确定性引擎构建

由于TensorRT的工作方式，引擎构建过程是不确定的。即使使用相同的模型和相同的构建器配置，引擎也可能略有不同。

为了使引擎构建可重现，所有Polygraphy CLI工具涉及构建TensorRT引擎时接受`--save-tactics`和`--load-tactics`选项，这允许您保存引擎选择的策略并在随后的构建中重新加载它们。

有关更多信息，请参考[`convert`示例02](./cli/convert/02_deterministic_engine_builds_in_tensorrt/)。


### 定义自定义TensorRT网络或构建配置

许多命令行工具涉及创建TensorRT网络。大多数时候，网络是通过解析来自框架的模型（通常是ONNX格式）创建的。然而，也可能使用Python脚本手动定义TensorRT网络。

如果您想以TensorRT Python API的某种方式修改网络；例如，设置层精度或逐层设备偏好。类似地，也可以提供自定义构建配置。这在Polygraphy尚不支持某些TensorRT构建器配置选项的情况下很有用。

有关更多信息，请参考[`run`示例04](./cli/run/04_defining_a_tensorrt_network_or_config_manually)。


### 提取ONNX模型的子图

在调试ONNX模型时，提取子图以孤立查看它们可能是有用的。`surgeon extract`工具允许您这样做。

有关更多信息，请参考[`surgeon`示例01](./cli/surgeon/01_isolating_subgraphs/)。


### 调试TensorRT间歇性失败

由于TensorRT优化器本质上是不确定的，重建引擎可能导致选择不同的策略。如果某个策略有缺陷，这可能表现为间歇性失败。`debug build`工具在这种情况下帮助发现有缺陷的策略并可靠地复现失败。

有关更多信息，请参考[`debug`示例01](./cli/debug/01_debugging_flaky_trt_tactics/)。


### 缩减失败的ONNX模型

在调查涉及ONNX模型的错误时，将模型减少到最小失败子图可以帮助我们确定问题并使进一步调试变得更容易。`debug reduce`工具帮助我们自动化这个过程。

## 示例

要查看示例，请参见 [examples/cli](./cli) 中的相应子目录。

## 添加新工具

您可以通过在此目录中添加一个新文件，创建一个扩展了[`Tool` 基类](https://github.com/NVIDIA/TensorRT/blob/main/tools/Polygraphy/polygraphy/tools/base/tool.py)的类，并将新工具添加到[注册表](https://github.com/NVIDIA/TensorRT/blob/main/tools/Polygraphy/polygraphy/tools/registry.py)中来添加一个新工具。

有关开发工具的详细信息，请参见[此示例](./dev/01_writing_cli_tools/)。

