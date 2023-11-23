# Defining A TensorRT Network Or Config Manually


## Introduction

在某些情况下，使用 Python API 从头开始定义一个 TensorRT 网络，或修改由其他方式创建的网络（例如，解析器）可能是有用的。通常，这会限制您使用 CLI 工具，至少在您构建引擎之前，因为网络不能序列化到磁盘上并在命令行上加载。

Polygraphy CLI 工具提供了一个解决方法 - 如果您的 Python 脚本定义了一个名为 `load_network` 的函数，该函数不接受参数并返回一个 TensorRT 构建器、网络，以及可选的解析器，那么您可以提供您的 Python 脚本代替模型参数。

类似地，我们可以使用定义了一个名为 `load_config` 的函数的脚本创建一个自定义的 TensorRT 构建器配置，该函数接受一个构建器和网络，并返回一个构建器配置。

在这个示例中，包含的 `define_network.py` 脚本解析一个 ONNX 模型并在其后附加一个恒等层。由于它在一个名为 `load_network` 的函数中返回构建器、网络和解析器，我们可以使用单个命令从中构建并运行一个 TensorRT 引擎。`create_config.py` 脚本创建一个新的 TensorRT 构建器配置并启用 FP16 模式。


### TIP: Generating Script Templates Automatically

你可以使用 `polygraphy template trt-network` 来自动创建网络脚本的起始点，而不是从头开始编写：

```bash
polygraphy template trt-network -o my_define_network.py
```

如果您想从一个模型开始并修改生成的 TensorRT 网络，而不是从头创建，只需将模型作为参数提供给 `template trt-network`：

```bash
polygraphy template trt-network identity.onnx -o my_define_network.py
```

类似地，您可以使用 `polygraphy template trt-config` 来生成配置的模板脚本：

```bash
polygraphy template trt-config -o my_create_config.py
```

您还可以指定构建器配置选项来预填充脚本。例如，启用 FP16 模式：

```bash
polygraphy template trt-config --fp16 -o my_create_config.py
```


## Running The Example

1. 运行 `define_network.py` 中定义的网络：

    ```bash
    polygraphy run --trt define_network.py --model-type=trt-network-script
    ```

2. 使用 `create_config.py` 中定义的构建器配置运行第 (1) 步中的网络：

    ```bash
    polygraphy run --trt define_network.py --model-type=trt-network-script --trt-config-script=create_config.py
    ```

    请注意，我们可以在同一个脚本中定义 `load_network` 和 `load_config`。 实际上，我们可以从任意脚本或甚至模块中检索这些函数。

> **提示：我们可以使用相同的方法与 `polygraphy convert` 来构建但不运行引擎。**
