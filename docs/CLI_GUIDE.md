# CLI 命令参考

imgcli 提供了 11 个子命令，覆盖图像格式转换、缩放、增强、标注掩码、数据增强、裁剪、OCR、背景去除、对象擦除、AI 超分辨率放大和图像分割。

## 通用约定

### 输入规则

- 所有命令的 `input` 参数支持**文件路径**或**目录路径**
- 传入目录时，自动扫描其中的图片文件（非递归，`ocr` 命令可通过 `-r` 开启递归，`upscale` 命令默认递归）
- `mask` 命令传入目录时扫描 `.json` 文件

### 输出规则

- 未指定 `-o` 时，自动在输入目录下创建 `<命令>_output/` 子目录（如 `convert_output/`、`resize_output/`）
- 输出文件名默认与输入文件名相同（保持 stem），扩展名按命令逻辑决定
- 指定 `-o` 时，输出到指定目录

### 支持的图片格式

通用命令（convert/resize/enhance/augment/crop/rmbg）：jpg、jpeg、png、bmp

OCR 命令额外支持：tiff、tif、webp

Upscale 命令支持：png、jpg、jpeg、bmp、tiff、tif、webp

---

## convert

图片格式转换，支持 JPG / PNG / BMP 互转。JPEG 输出质量固定为 95。

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `-f, --format` | 是 | string | — | 目标格式：jpg、png、bmp |
| `-o, --output` | 否 | string | `<input_dir>/convert_output/` | 输出目录 |

### 示例

```bash
# 单文件转换
imgcli convert photo.png -f jpg

# 批量转换
imgcli convert ./images/ -f png

# 指定输出目录
imgcli convert photo.jpg -f bmp -o ./output/
```

---

## resize

缩放图片到指定分辨率，使用 Lanczos3 重采样算法保持画质。

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `-W, --width` | 是 | u32 | — | 目标宽度（像素） |
| `-H, --height` | 是 | u32 | — | 目标高度（像素） |
| `-o, --output` | 否 | string | `<input_dir>/resize_output/` | 输出目录 |

### 示例

```bash
# 缩放到 800x600
imgcli resize photo.jpg -W 800 -H 600

# 批量缩放
imgcli resize ./images/ -W 640 -H 480
```

---

## enhance

调整图片对比度、亮度、锐化，或仅检测感知亮度。

`--detect-brightness` 与调整参数互斥：指定检测模式时仅打印亮度值，不生成输出文件。不指定检测模式时，必须至少提供一个调整参数（`--contrast`、`--brightness` 或 `--sharpen`）。

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `--contrast` | 否 | f64 | 1.0 | 对比度系数（>1 增强，<1 降低） |
| `--brightness` | 否 | f64 | 0.0 | 亮度偏移（>0 变亮，<0 变暗） |
| `--sharpen` | 否 | flag | false | 启用 Unsharp Mask 锐化 |
| `--detect-brightness` | 否 | flag | false | 仅检测并打印感知亮度 |
| `-o, --output` | 否 | string | `<input_dir>/enhance_output/` | 输出目录 |

### 亮度检测

感知亮度计算公式：`0.299R + 0.587G + 0.114B`，输出值为 0-255 范围的均值。

### 示例

```bash
# 增强对比度
imgcli enhance photo.jpg --contrast 1.5

# 提亮并锐化
imgcli enhance dark.jpg --brightness 30 --sharpen

# 检测亮度
imgcli enhance photo.jpg --detect-brightness
```

---

## mask

从 LabelMe JSON 标注文件生成二值掩码图片（PNG 格式）。

### LabelMe JSON 格式

输入文件为 LabelMe 标注的 JSON 文件，需包含以下字段：

```json
{
  "imageWidth": 1920,
  "imageHeight": 1080,
  "shapes": [
    {
      "label": "cat",
      "points": [[100.0, 200.0], [300.0, 400.0], ...]
    }
  ]
}
```

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | LabelMe JSON 文件或目录 |
| `--label` | 是 | string | — | 要提取的目标标签名 |
| `-o, --output` | 否 | string | `<input_dir>/mask_output/` | 输出目录 |

### 示例

```bash
# 从标注文件生成 cat 标签的掩码
imgcli mask annotation.json --label cat

# 批量处理
imgcli mask ./annotations/ --label building
```

---

## augment

数据增强，支持 5 种增强类型。可组合多种增强同时作用于每张图片，也可通过 distribute 模式分别生成。

### 增强类型

| 类型 | 说明 |
|------|------|
| `hflip` | 水平翻转 |
| `vflip` | 垂直翻转 |
| `rotate` | 随机旋转（0-345 度，以 15 度为步长） |
| `color_jitter` | HSL 色彩抖动（亮度 ±0.2、对比度 ×0.8~1.2、饱和度 ±0.2、色相 ±0.1） |
| `affine` | 仿射变换（平移 15% + 剪切 ±10 度） |

### 模式说明

- **组合模式**（默认）：对每张图片依次应用所有选定的增强类型，生成 N 个样本
- **distribute 模式**（`-d`）：将 N 个样本均匀分配给各增强类型，每种类型独立生成

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `-a, --augment` | 是 | 多选 | — | 增强类型，可指定多个 |
| `-n, --num` | 否 | u32 | 1 | 每张图片生成的样本数 |
| `-d, --distribute` | 否 | flag | false | distribute 模式 |
| `-o, --output` | 否 | string | `<input_dir>/augment_output/` | 输出目录 |

### 输出命名

- 组合模式：`{stem}_aug_{i}.{ext}`
- distribute 模式：`{stem}_{augment_type}_{i}.{ext}`

### 示例

```bash
# 水平翻转 + 垂直翻转，各生成 3 张
imgcli augment photo.jpg -a hflip vflip -n 3

# distribute 模式：3 种增强各生成 5 张
imgcli augment photo.jpg -a hflip rotate color_jitter -n 5 -d

# 组合增强
imgcli augment ./images/ -a hflip color_jitter affine -n 10
```

---

## crop

将图片裁剪为网格切片，自动计算步长确保完整覆盖。

### 切片逻辑

- 根据图片尺寸和切片大小自动计算行列数
- 自动计算步长（step），确保边缘区域也被覆盖
- 如果切片大于步长则产生重叠（overlap）
- 输出覆盖率（coverage）表示切片面积总和与原图面积之比

### 输出命名

格式：`{stem}_r{row}_c{col}.{ext}`（如 `photo_r0_c0.png`）

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `-W, --width` | 是 | u32 | — | 切片宽度（像素） |
| `-H, --height` | 是 | u32 | — | 切片高度（像素） |
| `-o, --output` | 否 | string | `<input_dir>/crop_output/` | 输出目录 |

### 示例

```bash
# 切成 512x512 的网格
imgcli crop photo.jpg -W 512 -H 512

# 批量裁剪
imgcli crop ./images/ -W 256 -H 256
```

---

## ocr

基于 PaddleOCR V5 的文字识别，使用 ONNX Runtime 推理。

### OCR 流水线

三阶段处理流程：

1. **检测（Det）**：`ch_PP-OCRv5_mobile_det.onnx` — 检测文本区域
2. **方向分类（Cls）**：`ch_ppocr_mobile_v2.0_cls_infer.onnx` — 判断文本方向
3. **识别（Rec）**：`ch_PP-OCRv5_rec_mobile_infer.onnx` — 识别文本内容

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `--models-dir` | 否 | string | `~/.imgcli/models/ocr/` | OCR 模型目录 |
| `-f, --format` | 否 | string | text | 输出格式：text 或 json |
| `-o, --output` | 否 | string | — | 输出文件路径（不指定则打印到终端） |
| `-r, --recursive` | 否 | flag | false | 递归搜索子目录 |
| `--confidence` | 否 | flag | false | 显示置信度分数 |
| `--pretty` | 否 | flag | false | JSON 格式化输出 |
| `-q, --quiet` | 否 | flag | false | 静默模式 |
| `--box-thresh` | 否 | f32 | 0.3 | 检测框阈值 |
| `--unclip-ratio` | 否 | f32 | 1.6 | 文本框扩展比例 |
| `--use-angle-cls` | 否 | flag | false | 启用方向分类 |

### 示例

```bash
# 识别单张图片
imgcli ocr screenshot.png

# JSON 格式输出
imgcli ocr screenshot.png -f json

# 递归识别目录，输出到文件
imgcli ocr ./documents/ -r -f json -o results.json --confidence

# 调整检测阈值
imgcli ocr blurry.png --box-thresh 0.2 --unclip-ratio 2.0
```

---

## rmbg

基于 BEN2 AI 模型的背景去除，输出为 RGBA 透明背景 PNG。

### 模型路径解析优先级

1. `--model` 参数指定的路径
2. `~/.imgcli/models/rmbg/BEN2_Base.onnx`
3. 当前工作目录下的 `./BEN2_Base.onnx`

### Provider 选项

| Provider | 说明 |
|----------|------|
| `auto`（默认） | 自动选择最佳可用 Provider |
| `cpu` | 仅使用 CPU 推理 |
| `cuda` | 使用 NVIDIA GPU（需 CUDA 运行时） |
| `coreml` | 使用 Apple CoreML（macOS） |

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `-o, --output` | 否 | string | `<input_dir>/rmbg_output/` | 输出目录 |
| `--model` | 否 | string | 见上方优先级 | BEN2 ONNX 模型文件路径 |
| `--provider` | 否 | enum | auto | 推理 Provider |

### 示例

```bash
# 去除背景
imgcli rmbg photo.jpg

# 使用 GPU 加速
imgcli rmbg photo.jpg --provider cuda

# 指定模型路径
imgcli rmbg photo.jpg --model /path/to/BEN2_Base.onnx
```

---

## erase

基于 MIGAN AI 模型的对象擦除/修复，通过掩码指定要移除的区域，AI 自动修复背景。

### 两种模式

- **单文件模式**（默认）：`input` 为图片路径，必须通过 `--mask` 指定对应的掩码图片（灰度图，白色区域为擦除目标）
- **批量模式**（`--batch`）：`input` 为目录，自动匹配 `<name>-img.<ext>` + `<name>-mask.<ext>` 配对文件

### 模型路径解析优先级

1. `--model` 参数指定的路径
2. `~/.imgcli/models/eraser/migan.onnx`
3. 当前工作目录下的 `./models/eraser/migan.onnx`

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入图片路径（单文件）或目录（批量） |
| `--mask` | 条件 | string | — | 掩码图片路径（单文件模式必需） |
| `--batch` | 否 | flag | false | 启用批量模式 |
| `-o, --output` | 否 | string | `<input_dir>/erase_output/` | 输出目录 |
| `--model` | 否 | string | 见上方优先级 | MIGAN ONNX 模型文件路径 |
| `--provider` | 否 | enum | auto | 推理 Provider（auto/cpu/cuda/coreml） |

### 输出命名

- 单文件模式：`{stem}_erase.png`
- 批量模式：`{name}-out.png`

### 示例

```bash
# 擦除图片中的对象
imgcli erase photo.jpg --mask mask.png

# 指定输出目录
imgcli erase photo.jpg --mask mask.png -o ./output/

# 批量处理配对文件
imgcli erase ./pairs/ --batch

# 使用 GPU 加速
imgcli erase photo.jpg --mask mask.png --provider cuda
```

---

## upscale

基于 Real-ESRGAN 的 AI 超分辨率放大，支持分块推理以适配不同显存大小。

### 支持的模型

模型文件名需包含 `x2` 或 `x4` 以自动检测放大倍率：

| 模型名 | 说明 |
|--------|------|
| `RealESR_Gx4` | 通用 4x 放大（默认） |
| `RealESR_Gx2` | 通用 2x 放大 |
| `RealESR_Animex4` | 动漫风格 4x 放大 |
| `RealESRGANx4` | 经典 Real-ESRGAN 4x 放大 |

### 分块推理

大图片会自动拆分为小块分别推理再拼接，分块大小根据 `--vram`（可用显存 GB）和模型类型自动计算。如显存不足，可降低 `--vram` 值以使用更小的分块。

### 模型路径解析

默认查找 `~/.imgcli/models/upscale/{model_name}_fp16.onnx`，可通过 `--models-dir` 指定自定义模型目录。

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `-m, --model` | 否 | string | `RealESR_Gx4` | AI 模型名称 |
| `-i, --input-scale` | 否 | u32 | 100 | 输入缩放百分比（100 = 不缩放） |
| `-o, --output-scale` | 否 | u32 | 100 | 输出缩放百分比（100 = 完整 AI 放大） |
| `-b, --blend` | 否 | f64 | 0.0 | 混合系数：原图与 AI 结果的权重（0.0 = 纯 AI，0.7 = 主要原图） |
| `--output-format` | 否 | enum | — | 输出格式（png/jpg/bmp/tiff/webp） |
| `--output-dir` | 否 | string | `<input_parent>/upscale_output/` | 输出目录 |
| `--models-dir` | 否 | string | — | ONNX 模型文件目录 |
| `--vram` | 否 | u32 | 4 | 可用 GPU 显存（GB），用于计算分块大小 |
| `--tile-overlap` | 否 | u32 | 8 | 分块重叠像素数 |
| `-t, --threads` | 否 | usize | 3 | 推理线程数（3、6 或 9） |
| `--overwrite` | 否 | flag | false | 覆盖已存在的输出文件 |

### 输出命名

格式：`{stem}_{model}_InputR-{input_scale}_OutputR-{output_scale}[._Blending-{blend}].{ext}`

### 示例

```bash
# 默认 4x 放大
imgcli upscale photo.jpg

# 使用 2x 模型
imgcli upscale photo.jpg -m RealESR_Gx2

# 动漫风格放大
imgcli upscale anime.png -m RealESR_Animex4

# 显存有限时降低分块大小
imgcli upscale large.jpg --vram 2

# 混合原图细节
imgcli upscale photo.jpg -b 0.3

# 批量处理目录
imgcli upscale ./images/

# 指定输出格式和目录
imgcli upscale photo.jpg --output-format webp --output-dir ./out/

# 使用更多线程加速
imgcli upscale photo.jpg -t 9
```

---

## seg

基于 SAM3 模型的图像分割，支持文本提示（text prompt）和框提示（box prompt）两种模式。

### 提示模式

- **文本提示**（`-t`）：输入自然语言描述要分割的目标（如 "person"、"car"）
- **框提示**（`-b`）：输入归一化坐标 `cx,cy,w,h`，格式为相对于图片尺寸的比例值（0.0~1.0）

两种模式互斥，必须指定其中一种。

### 输出

每张输入图片生成两个文件：
- `{stem}_mask.png` — 分割掩码
- `{stem}_overlay.png` — 原图叠加可视化

使用 `--labelme` 时额外生成 `{stem}.json`（LabelMe 标注格式）。

### 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | 是 | 位置参数 | — | 输入文件或目录 |
| `-t, --text` | 条件 | string | — | 文本提示（与 -b 互斥） |
| `-b, --box` | 条件 | string | — | 框提示，格式：cx,cy,w,h（与 -t 互斥） |
| `--models-dir` | 否 | string | `~/.imgcli/models/seg` | 模型目录 |
| `--labelme` | 否 | flag | false | 导出 LabelMe JSON 格式 |
| `-o, --output` | 否 | string | `<input_dir>/seg_output/` | 输出目录 |

### 示例

```bash
# 文本提示分割
imgcli seg photo.jpg -t "person"

# 框提示分割
imgcli seg photo.jpg -b 0.5,0.5,0.3,0.4

# 导出 LabelMe 标注
imgcli seg photo.jpg -t "car" --labelme

# 指定模型目录
imgcli seg photo.jpg -t "building" --models-dir /path/to/models

# 批量处理
imgcli seg ./images/ -t "person"
```
