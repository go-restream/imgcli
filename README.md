# imgcli

统一的命令行图像处理工具，基于 Rust 构建，提供格式转换、缩放、增强、标注掩码生成、数据增强、裁剪、OCR 文字识别、AI 去背景、对象擦除和 AI 超分辨率放大等功能。

## 功能一览

| 命令 | 说明 |
|------|------|
| `convert` | 图片格式转换（JPG / PNG / BMP） |
| `resize` | 缩放图片分辨率 |
| `enhance` | 调整对比度、亮度、锐化，或检测感知亮度 |
| `mask` | 从 LabelMe JSON 标注生成二值掩码 |
| `augment` | 数据增强（翻转、旋转、色彩抖动、仿射变换） |
| `crop` | 网格裁剪，支持重叠切片 |
| `ocr` | 基于 PaddleOCR 的文字识别 |
| `rmbg` | 基于 BEN2 AI 模型的背景去除 |
| `erase` | 基于 MIGAN AI 模型的对象擦除/修复 |
| `upscale` | 基于 Real-ESRGAN 的 AI 超分辨率放大 |
| `seg` | 基于 SAM3 模型的图像分割（支持文本/框提示） |

完整命令参考请查看 [CLI_GUIDE.md](docs/CLI_GUIDE.md)。

## 安装

### 前置条件

- **Rust 工具链**：安装 [rustup](https://rustup.rs/) 以获取 `cargo` 编译工具
- **C++ 编译器**：用于编译 `clipper-sys` 依赖
- **网络连接**：首次编译时 ONNX Runtime 会自动下载预编译库

### 编译安装

```bash
git clone https://github.com/your-username/imgcli.git
cd imgcli
cargo install --path .

#Or build : target/release/imgcli 
cargo build --release

```

安装完成后，`imgcli` 二进制文件将位于 `~/.cargo/bin/`（确保该路径已加入 `PATH`）。

## 各平台构建注意事项

### macOS

需要安装 Xcode Command Line Tools：

```bash
xcode-select --install
```

### Linux

需要安装 GCC/G++ 编译器和相关系统库：

```bash
# Debian / Ubuntu
sudo apt install build-essential pkg-config libssl-dev

# Fedora / RHEL
sudo dnf install gcc gcc-c++ openssl-devel
```

### Windows

需要安装 [MSVC Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)，在安装程序中勾选「C++ 桌面开发」工作负载。

### ONNX Runtime

`ort` crate 在编译时默认自动下载对应平台的 ONNX Runtime 预编译库，无需手动安装。如果网络环境受限，可通过环境变量指定本地库路径。

## 快速开始

### 格式转换

将 PNG 转为 JPEG：

```bash
imgcli convert photo.png -f jpg
```

批量转换目录下所有图片为 PNG：

```bash
imgcli convert ./images/ -f png
```

### 缩放图片

```bash
imgcli resize photo.jpg -W 800 -H 600
```

### 背景去除

```bash
imgcli rmbg photo.jpg
```

输出为 RGBA 透明背景 PNG，保存在 `rmbg_output/` 目录。

### OCR 文字识别

```bash
imgcli ocr screenshot.png
```

识别结果直接输出到终端。使用 `-f json` 可输出 JSON 格式。

### 对象擦除

使用掩码指定要擦除的区域：

```bash
imgcli erase photo.jpg --mask mask.png
```

批量模式需要 `<name>-img.<ext>` + `<name>-mask.<ext>` 配对文件：

```bash
imgcli erase ./pairs/ --batch
```

### AI 超分辨率放大

```bash
imgcli upscale photo.jpg
```

默认 4 倍放大。可通过 `-m` 指定不同模型，通过 `--vram` 调整分块大小以适配显存。

### 图像分割

使用文本提示分割：

```bash
imgcli seg photo.jpg -t "person"
```

使用框提示分割（归一化坐标 cx,cy,w,h）：

```bash
imgcli seg photo.jpg -b 0.5,0.5,0.3,0.4
```

## AI 模型配置

`ocr`、`rmbg`、`erase`、`upscale` 和 `seg` 命令需要额外的 ONNX 模型文件。默认模型目录为 `~/.imgcli/models/`。

### 目录结构

```
~/.imgcli/models/
├── rmbg/                           # rmbg 模型目录
├── ocr/                            # OCR 模型目录
├── eraser/                         # erase 模型目录
├── upscale/                        # upscale 模型目录
└── seg/                            # seg 模型目录
```

### 模型下载

- **OCR**（文字识别）：[ocr_onnx_v5.zip](https://github.com/go-restream/imgcli/releases/download/v0.1.0/ocr_onnx_v5.zip)
- **REMOVEBG**（背景去除）：[rmbg.zip](https://github.com/go-restream/imgcli/releases/download/v0.1.0/rmbg.zip)
- **ERASER**（对象擦除）：[eraser_onnx_v1.zip](https://github.com/go-restream/imgcli/releases/download/v0.1.0/eraser_onnx_v1.zip)
- **UPSCALER**（超分辨率）：[upscaler_onnx.zip](https://github.com/go-restream/imgcli/releases/download/v0.1.0/upscaler_onnx.zip)
- **SAM3**（图像分割）：[sam3-onnx-models-v0.3.0](https://huggingface.co/wkentaro/sam3-onnx-models-v0.3.0)

下载后解压并放置到上述目录即可。也可通过命令行参数 `--model`（rmbg、erase）或 `--models-dir`（ocr、seg、upscale）指定自定义路径。

## 完整命令参考

详见 [CLI_GUIDE.md](docs/CLI_GUIDE.md)。
