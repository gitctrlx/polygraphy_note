# Polygraphy Note - API/CLI

本仓库为 Polygraphy  API/CLI文档中文翻译笔记与代码样例。

## 文档链接

- Polygraphy CLI文档：[https://github.com/NVIDIA/TensorRT/tree/main/tools/Polygraphy/polygraphy/tools](https://github.com/NVIDIA/TensorRT/tree/main/tools/Polygraphy/polygraphy/tools)

- Polygraphy API文档：[https://docs.nvidia.com/deeplearning/tensorrt/polygraphy/docs/index.html#](https://docs.nvidia.com/deeplearning/tensorrt/polygraphy/docs/index.html#)

- Polygraphy API文档概述：[https://github.com/NVIDIA/TensorRT/tree/main/tools/Polygraphy/polygraphy](https://github.com/NVIDIA/TensorRT/tree/main/tools/Polygraphy/polygraphy)

## 获取 Polygraphy

您可以在以下 GitHub 仓库地址找到 Polygraphy 的源代码和最新更新：

[https://github.com/NVIDIA/TensorRT/tree/main/tools/Polygraphy](https://github.com/NVIDIA/TensorRT/tree/main/tools/Polygraphy)

## 翻译进度

+ [ ] API
+ [x] CLI
+ [ ] how-to


## 关于 Polygraphy

Polygraphy 是一个功能强大的工具包，旨在帮助您在各种深度学习框架中运行和调试模型。它提供了一个Python API和一个使用该API构建的命令行界面（CLI），使您能够轻松执行各种深度学习任务。

Polygraphy 不仅提供了丰富的功能，还具有以下优势：

- 在多个后端（例如 TensorRT 和 ONNX-Runtime）之间运行推理，并比较结果（例如：API、CLI） 

- 将模型转换为各种格式，例如具有训练后量化功能的 TensorRT 引擎（例如：API、CLI） 

- 查看各种类型模型的信息（例如：CLI） 

- 在命令行上修改 ONNX 模型：         
  - 提取子图（示例：CLI）         

  - 简化和清理（示例：CLI） 

- 隔离 TensorRT 中的错误策略（示例：CLI） 


![](./assets/Polygraphy%20CLI.png)

