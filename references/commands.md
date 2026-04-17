# imgcli Command Reference

Complete parameter documentation and advanced usage patterns for all imgcli commands.

## convert — Format Conversion

Convert images between JPG, PNG, and BMP formats.

```bash
imgcli convert <input> -f <format> [-o <dir>]
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `input` | Yes | File or directory path |
| `-f, --format` | Yes | Target format: `jpg`, `png`, `bmp` |
| `-o, --output` | No | Output directory (default: `convert_output/`) |

Examples:
```bash
imgcli convert photo.png -f jpg
imgcli convert ./images/ -f png -o ./converted/
```

## resize — Image Resize

Resize images using Lanczos3 resampling for high-quality downscaling/upscaling.

```bash
imgcli resize <input> -W <width> -H <height> [-o <dir>]
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `input` | Yes | File or directory path |
| `-W, --width` | Yes | Target width in pixels |
| `-H, --height` | Yes | Target height in pixels |
| `-o, --output` | No | Output directory (default: `resize_output/`) |

## enhance — Image Enhancement

Adjust contrast, brightness, apply sharpening, or detect perceptual brightness.

```bash
imgcli enhance <input> [--contrast <factor>] [--brightness <offset>] [--sharpen] [--detect-brightness] [-o <dir>]
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `input` | Yes | — | File or directory path |
| `--contrast` | No | 1.0 | Contrast multiplier (>1 increases) |
| `--brightness` | No | 0.0 | Brightness offset (>0 brightens) |
| `--sharpen` | No | false | Enable unsharp mask sharpening |
| `--detect-brightness` | No | false | Print perceptual brightness only (0.299R + 0.587G + 0.114B) |
| `-o, --output` | No | — | Output directory (default: `enhance_output/`) |

Examples:
```bash
imgcli enhance dark.jpg --brightness 30 --sharpen
imgcli enhance photo.jpg --contrast 1.5
imgcli enhance photo.jpg --detect-brightness   # prints value, no output file
```

## mask — LabelMe Mask Generation

Generate binary masks from LabelMe JSON annotation files.

```bash
imgcli mask <input> --label <name> [-o <dir>]
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `input` | Yes | LabelMe JSON file or directory of `.json` files |
| `--label` | Yes | Target label name to extract |
| `-o, --output` | No | Output directory (default: `mask_output/`) |

## augment — Data Augmentation

Apply data augmentation transforms for ML training data preparation.

```bash
imgcli augment <input> -a <type> [-a <type>] [-n <count>] [-d] [-o <dir>]
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `input` | Yes | — | File or directory path |
| `-a, --augment` | Yes | — | Augmentation type(s): `hflip`, `vflip`, `rotate`, `color_jitter`, `affine` |
| `-n, --num` | No | 1 | Samples per image per augmentation type |
| `-d, --distribute` | No | false | Distribute samples evenly across types |
| `-o, --output` | No | — | Output directory (default: `augment_output/`) |

Examples:
```bash
imgcli augment photo.jpg -a hflip rotate -n 3
imgcli augment ./dataset/ -a hflip vflip rotate color_jitter -n 5 -d
```

## crop — Grid Crop

Slice images into grid tiles with automatic overlap for full coverage.

```bash
imgcli crop <input> -W <tile_w> -H <tile_h> [-o <dir>]
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `input` | Yes | File or directory path |
| `-W, --width` | Yes | Tile width in pixels |
| `-H, --height` | Yes | Tile height in pixels |
| `-o, --output` | No | Output directory (default: `crop_output/`) |

## ocr — Text Recognition (PaddleOCR V5)

Recognize text in images using PaddleOCR V5 with a 3-stage pipeline: detection (DBNet) → classification (angle) → recognition (CRNN+CTC).

```bash
imgcli ocr <input> [-f <format>] [-o <file>] [-r] [--confidence] [--pretty] [-q] [--box-thresh <f>] [--unclip-ratio <f>] [--use-angle-cls] [--models-dir <dir>]
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `input` | Yes | — | File or directory path |
| `-f, --format` | No | `text` | Output format: `text` or `json` |
| `-o, --output` | No | stdout | Output file path |
| `-r, --recursive` | No | false | Recurse into subdirectories |
| `--confidence` | No | false | Show confidence scores |
| `--pretty` | No | false | Pretty-print JSON output |
| `-q, --quiet` | No | false | Suppress progress output |
| `--box-thresh` | No | 0.3 | Detection box confidence threshold |
| `--unclip-ratio` | No | 1.6 | Text box expansion ratio |
| `--use-angle-cls` | No | false | Enable 0/180 degree direction classification |
| `--models-dir` | No | `~/.imgcli/models/ocr/` | Model directory path |

Examples:
```bash
imgcli ocr screenshot.png
imgcli ocr screenshot.png -f json --pretty
imgcli ocr ./documents/ -r -f json -o results.json
imgcli ocr receipt.jpg --confidence --box-thresh 0.5
```

### OCR Pipeline

1. **Detection** (DBNet): Locates text regions in the image
2. **Classification** (AngleNet, optional): Detects text orientation (0° or 180°)
3. **Recognition** (CRNN): Reads text from detected regions using CTC decoding

## rmbg — Background Removal (BEN2)

Remove image backgrounds using the BEN2 model via ONNX Runtime.

```bash
imgcli rmbg <input> [-o <dir>] [--model <path>] [--provider <provider>]
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `input` | Yes | — | File or directory path |
| `-o, --output` | No | — | Output directory (default: `rmbg_output/`) |
| `--model` | No | `~/.imgcli/models/rmbg/BEN2_Base.onnx` | Path to BEN2 ONNX model |
| `--provider` | No | `auto` | ONNX Runtime provider: `auto`, `cpu`, `cuda`, `coreml` |

Examples:
```bash
imgcli rmbg photo.jpg
imgcli rmbg photo.jpg --provider cuda       # GPU acceleration
imgcli rmbg ./portraits/ --provider coreml  # Apple Silicon
```

### Processing Pipeline

1. Resize input to 1024x1024 (preprocessing)
2. Run BEN2 model inference
3. Bilinear interpolation to restore original size (postprocessing)
4. Normalize mask and apply as alpha channel
5. Save as PNG with transparency

## erase — Object Erasing (MIGAN)

Remove objects from images using MIGAN inpainting model. Provide a grayscale mask (white = erase area) and the model automatically repairs the background.

```bash
imgcli erase <input> --mask <mask_path> [--batch] [-o <dir>] [--model <path>] [--provider <provider>]
```

### Single Mode

`input` is an image file, `--mask` is required to specify the grayscale mask.

### Batch Mode

Add `--batch` flag, `input` becomes a directory. Automatically matches `<name>-img.<ext>` + `<name>-mask.<ext>` pairs.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `input` | Yes | — | Image file (single) or directory (batch) |
| `--mask` | Yes* | — | Grayscale mask path (required in single mode) |
| `--batch` | No | false | Enable batch mode |
| `-o, --output` | No | — | Output directory (default: `erase_output/`) |
| `--model` | No | `~/.imgcli/models/eraser/migan.onnx` | Path to MIGAN ONNX model |
| `--provider` | No | `auto` | ONNX Runtime provider: `auto`, `cpu`, `cuda`, `coreml` |

*`--mask` is required in single mode. Ignored in batch mode (auto-matched by filename).

### Output Naming

- Single: `{stem}_erase.png`
- Batch: `{name}-out.png`

Examples:
```bash
imgcli erase photo.jpg --mask mask.png
imgcli erase photo.jpg --mask mask.png --provider cuda
imgcli erase ./pairs/ --batch
```

### Processing Pipeline

1. Load image (RGB) and mask (grayscale)
2. Validate dimensions match
3. Crop image and mask to masked regions
4. Run MIGAN inpainting model
5. Paste inpainted result back into original image
6. Save as PNG

## upscale — AI Super-Resolution (Real-ESRGAN)

Upscale images using Real-ESRGAN models with automatic tile-based inference for VRAM adaptation.

```bash
imgcli upscale <input> [-m <model>] [-i <scale>] [-o <scale>] [-b <blend>] [--output-format <fmt>] [--output-dir <dir>] [--models-dir <dir>] [--vram <gb>] [--tile-overlap <px>] [-t <threads>] [--overwrite]
```

### Supported Models

Model name must contain `x2` or `x4` for automatic upscale factor detection:

| Model | Description |
|-------|-------------|
| `RealESR_Gx4` | General-purpose 4x upscale (default) |
| `RealESR_Gx2` | General-purpose 2x upscale |
| `RealESR_Animex4` | Anime-oriented 4x upscale |
| `RealESRGANx4` | Classic Real-ESRGAN 4x upscale |

### Tile-Based Inference

Large images are automatically split into tiles based on available VRAM. Lower `--vram` values produce smaller tiles for limited GPU memory.

### Model Path

Defaults to `~/.imgcli/models/upscale/{model_name}_fp16.onnx`. Override with `--models-dir`.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `input` | Yes | — | File or directory path (recursive) |
| `-m, --model` | No | `RealESR_Gx4` | Model name |
| `-i, --input-scale` | No | 100 | Input resize percentage (100 = no resize) |
| `-o, --output-scale` | No | 100 | Output resize percentage (100 = full AI upscale) |
| `-b, --blend` | No | 0.0 | Blend factor: original vs AI result (0.0 = pure AI, 0.7 = mostly original) |
| `--output-format` | No | — | Output format: `png`, `jpg`, `bmp`, `tiff`, `webp` |
| `--output-dir` | No | — | Output directory (default: `<input_parent>/upscale_output/`) |
| `--models-dir` | No | — | ONNX model directory |
| `--vram` | No | 4 | Available GPU VRAM in GB |
| `--tile-overlap` | No | 8 | Tile overlap in pixels |
| `-t, --threads` | No | 3 | Inference threads (3, 6, or 9) |
| `--overwrite` | No | false | Overwrite existing output files |

### Output Naming

`{stem}_{model}_InputR-{input_scale}_OutputR-{output_scale}[_Blending-{blend}].{ext}`

Examples:
```bash
imgcli upscale photo.jpg                          # Default 4x upscale
imgcli upscale photo.jpg -m RealESR_Gx2           # 2x upscale
imgcli upscale anime.png -m RealESR_Animex4       # Anime model
imgcli upscale large.jpg --vram 2                 # Limited VRAM
imgcli upscale photo.jpg -b 0.3                   # Blend with original
imgcli upscale ./images/                          # Batch (recursive)
imgcli upscale photo.jpg --output-format webp -t 9
```

### Processing Pipeline

1. Discover input files (recursive for directories)
2. Load model, create inference sessions (thread count)
3. For each image: split into tiles based on VRAM and model type
4. Run Real-ESRGAN inference on each tile
5. Stitch tiles back together with overlap blending
6. Apply input/output scale and optional blend with original
7. Save in specified format

## seg — Image Segmentation (SAM3)

Segment images using SAM3 with text or box prompts via ONNX Runtime.

```bash
imgcli seg <input> -t "<text>" | -b <cx,cy,w,h> [--models-dir <dir>] [--labelme] [-o <dir>]
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `input` | Yes | — | File or directory path |
| `-t, --text` | Yes* | — | Text prompt (mutually exclusive with `-b`) |
| `-b, --box` | Yes* | — | Box prompt: `cx,cy,w,h` in normalized [0,1] coords (mutually exclusive with `-t`) |
| `--models-dir` | No | `~/.imgcli/models/seg/` | Model directory path |
| `--labelme` | No | false | Also export LabelMe JSON annotations |
| `-o, --output` | No | — | Output directory (default: `seg_output/`) |

*One of `-t` or `-b` is required.

Examples:
```bash
imgcli seg photo.jpg -t "person"
imgcli seg photo.jpg -t "car" --labelme
imgcli seg photo.jpg -b 0.5,0.5,0.3,0.4
```

### Processing Pipeline

1. Encode image via SAM3 image encoder
2. Encode text prompt via SAM3 language encoder (for `-t` mode)
3. Run decoder with image + text/box embeddings
4. Postprocess: resize mask to original dimensions, generate overlay visualization
5. Optional: export LabelMe JSON with simplified polygon (Douglas-Peucker)
