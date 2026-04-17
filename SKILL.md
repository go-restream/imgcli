---
name: imgcli
description: "Unified CLI image processing tool. Covers format conversion, resize, enhance, LabelMe mask generation, data augmentation, grid crop, OCR (PaddleOCR V5), background removal (BEN2), object erasing (MIGAN), AI upscaling (Real-ESRGAN), and segmentation (SAM3). TRIGGER when: user asks to convert, resize, crop, enhance, augment, erase, upscale, or process images via CLI; user asks to run OCR on images; user asks to remove image backgrounds; user asks to erase/remove objects from images; user asks to upscale/enlarge/super-resolve images; user asks to segment images with text/box prompts; user asks to generate masks from LabelMe annotations; user mentions imgcli commands (convert, resize, enhance, mask, augment, crop, ocr, rmbg, erase, upscale, seg). DO NOT TRIGGER for: general image editing GUI tools, web-based image processing, developing/building the imgcli Rust project itself."
---

# imgcli

Unified CLI image processing tool. Supports traditional image operations and AI-powered tasks (OCR, background removal, object erasing, AI upscaling, segmentation).

## Usage

```bash
imgcli <command> <input> [options]
```

- `input` — file or directory path
- Output defaults to `<command>_output/`, override with `-o`
- Supports batch processing when `input` is a directory

## Commands

| Command | Purpose | Required Flags |
|---------|---------|---------------|
| `convert` | Format conversion (jpg/png/bmp) | `-f <format>` |
| `resize` | Resize to target resolution | `-W <width> -H <height>` |
| `enhance` | Contrast/brightness/sharpen | `--contrast` `--brightness` `--sharpen` `--detect-brightness` |
| `mask` | LabelMe JSON → binary mask | `--label <name>` |
| `augment` | Data augmentation | `-a <type>` (multi-valued) `-n <count>` `-d` |
| `crop` | Grid-slice into tiles | `-W <tile_w> -H <tile_h>` |
| `ocr` | Text recognition (PaddleOCR V5) | `-f json\|text` `--pretty` `--confidence` `-r` |
| `rmbg` | Background removal (BEN2) | `--provider <auto\|cpu\|cuda\|coreml>` |
| `erase` | Object erasing/inpainting (MIGAN) | `--mask <path>` `--batch` `--provider <auto\|cpu\|cuda\|coreml>` |
| `upscale` | AI super-resolution (Real-ESRGAN) | `-m <model>` `--vram <gb>` `-b <blend>` `--output-format <fmt>` |
| `seg` | Segmentation (SAM3) | `-t "<text>"` or `-b <cx,cy,w,h>` `--labelme` |

### Augment Types

`hflip`, `vflip`, `rotate`, `color_jitter`, `affine`

## Quick Examples

```bash
# Traditional image operations
imgcli convert photo.png -f jpg
imgcli resize photo.jpg -W 800 -H 600
imgcli enhance dark.jpg --brightness 30 --sharpen
imgcli mask annotation.json --label cat
imgcli augment photo.jpg -a hflip rotate -n 3
imgcli crop photo.jpg -W 512 -H 512

# AI-powered operations (require ONNX models)
imgcli ocr screenshot.png
imgcli ocr screenshot.png -f json --pretty
imgcli rmbg photo.jpg
imgcli rmbg photo.jpg --provider cuda
imgcli erase photo.jpg --mask mask.png
imgcli erase ./pairs/ --batch
imgcli upscale photo.jpg
imgcli upscale photo.jpg -m RealESR_Gx2 --vram 2
imgcli seg photo.jpg -t "person"
imgcli seg photo.jpg -b 0.5,0.5,0.3,0.4
```

## AI Model Setup

OCR, rmbg, erase, upscale, and seg commands require ONNX models in `~/.imgcli/models/`:

```
~/.imgcli/models/
├── ocr/
│   ├── ch_PP-OCRv5_mobile_det.onnx
│   ├── ch_ppocr_mobile_v2.0_cls_infer.onnx
│   └── ch_PP-OCRv5_rec_mobile_infer.onnx
├── rmbg/
│   └── BEN2_Base.onnx
├── eraser/
│   └── migan.onnx
├── upscale/
│   ├── RealESR_Gx4_fp16.onnx
│   └── ...
└── seg/
    ├── sam3_image_encoder.onnx
    ├── sam3_language_encoder.onnx
    └── sam3_decoder.onnx
```

Override paths with `--models-dir` (ocr, seg, upscale) or `--model` (rmbg, erase).

## Detailed Reference

For complete parameter documentation, usage patterns, and advanced options for each command, see [commands.md](references/commands.md).
