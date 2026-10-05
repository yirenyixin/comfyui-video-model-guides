# MiniMax H3 ComfyUI Deployment and Generation Guide

This guide is written for the following Windows directory layout:

```text
ComfyUI data directory: D:\ComfyUI
Active workspace: D:\ComfyUI\workspace\test1
ComfyUI application: D:\ComfyUI\workspace\test1\ComfyUI
Shared model library: D:\ComfyUI\models
Workflow directory: D:\ComfyUI\ComfyUI-Workflows\MinMax_H3
```

The setup uses the MiniMax H3 Fused Turbo INT8 ConvRot diffusion model, separate video and audio VAEs, a Heretic text encoder, and low-VRAM workflows. It supports text-to-video, image-to-video, and reference-to-video generation.

## 1. Requirements

- Windows 10 or Windows 11.
- An up-to-date ComfyUI installation.
- An NVIDIA GPU. The low-VRAM workflows reduce peak VRAM use, but smaller GPUs run more slowly.
- At least 32 GB of system RAM; 64 GB is recommended.
- At least 50–70 GB of free disk space. Hugging Face download caches may temporarily require additional space.
- Python and the Hugging Face CLI.

Install or update the Hugging Face CLI if the `hf` command is unavailable:

```powershell
py -m pip install -U huggingface_hub
```

For networks that cannot reliably reach Hugging Face, set a mirror in every new PowerShell window:

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

This environment variable applies only to the current PowerShell session.

## 2. Download the Models

### 2.1 Fused diffusion model

Source:

<https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/diffusion_models>

```powershell
hf download MATLOWAI/minimax-h3-fused-turbo-int8-convrot `
  "diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

Expected location:

```text
D:\ComfyUI\models\diffusion_models\minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors
```

Windows reports about 19.54 GiB, while the Hugging Face page displays approximately 21 GB in decimal units.

### 2.2 Video VAE

Source:

<https://huggingface.co/Kijai/MiniMax-H3-experimental>

```powershell
hf download Kijai/MiniMax-H3-experimental `
  "minimax_h3_video_vae_int8_convrot.safetensors" `
  --local-dir "D:\ComfyUI\models\vae"
```

Expected location:

```text
D:\ComfyUI\models\vae\minimax_h3_video_vae_int8_convrot.safetensors
```

Windows reports about 2.95 GiB.

### 2.3 Audio VAE

```powershell
hf download Comfy-Org/MiniMax-H3 `
  "vae/minimax_h3_audio_vae_fp32.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

Expected location:

```text
D:\ComfyUI\models\vae\minimax_h3_audio_vae_fp32.safetensors
```

Windows reports about 0.56 GiB.

### 2.4 Heretic text encoder

Source:

<https://huggingface.co/Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4>

```powershell
hf download Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4 `
  "qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors" `
  --local-dir "D:\ComfyUI\models\text_encoders"
```

Expected location:

```text
D:\ComfyUI\models\text_encoders\qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors
```

Windows reports about 14.61 GiB. Select this file in `CLIPLoader`, set the type to `minimax`, and leave the device at `default`.

If the Heretic NVFP4 encoder is unsupported on your GPU, use `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` from `Comfy-Org/MiniMax-H3` instead.

## 3. Download the Low-VRAM Workflows

Source:

<https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/workflows/lowvram>

```powershell
hf download MATLOWAI/minimax-h3-fused-turbo-int8-convrot `
  --include "workflows/lowvram/*.json" `
  --local-dir "D:\ComfyUI\ComfyUI-Workflows"
```

The CLI preserves repository subdirectories, so downloaded files initially appear under:

```text
D:\ComfyUI\ComfyUI-Workflows\workflows\lowvram
```

You may copy the non-API workflows to:

```text
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3
```

Recommended workflows:

```text
01_reference_4step_sla_lowvram.json   Text to video
04_i2v_fl2v_4step_sla_lowvram.json   Image to video
05_ref2va_4step_sla_lowvram.json      Reference image to video
```

Use the normal `.json` files in the ComfyUI interface, not the `.api.json` variants.

## 4. Connect the test1 Workspace to the Shared Model Library

The active ComfyUI application is located at:

```text
D:\ComfyUI\workspace\test1\ComfyUI
```

The models are stored separately at:

```text
D:\ComfyUI\models
```

Create this file:

```text
D:\ComfyUI\workspace\test1\ComfyUI\extra_model_paths.yaml
```

Use the following content:

```yaml
d_models:
  base_path: D:/ComfyUI/models
  checkpoints: checkpoints
  diffusion_models: diffusion_models
  unet: diffusion_models
  text_encoders: text_encoders
  vae: vae
  loras: loras
  latent_upscale_models: latent_upscale_models
  upscale_models: upscale_models
```

Completely stop and restart the ComfyUI backend after changing this file. Refreshing the browser alone does not reload model paths.

## 5. Install Required Custom Nodes

Open ComfyUI Manager, update ComfyUI, and install:

```text
ComfyUI-KJNodes
ComfyUI-MAINodes
ComfyUI-PlagueKind-Nodes-only-sparse
```

- `ComfyUI-KJNodes` provides `MiniMaxChunkFeedForward` for the low-VRAM workflows.
- `ComfyUI-MAINodes` provides MiniMax H3 Motion Lab and de-roping nodes.
- `ComfyUI-PlagueKind-Nodes-only-sparse` provides the `H3SLAAttention` sparse-attention node.

Restart ComfyUI after installation. If red nodes remain, use Manager's missing-node installer.

## 6. Model Selections Inside the Workflows

The loader nodes should use:

```text
UNETLoader:
minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors

Video VAELoader:
minimax_h3_video_vae_int8_convrot.safetensors

Audio VAELoader:
minimax_h3_audio_vae_fp32.safetensors

CLIPLoader:
qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors
Type: minimax
Device: default
```

Do not add a nonexistent `minimax_h3/` prefix to these file names.

## 7. Text-to-Video

Open:

```text
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3\01_reference_4step_sla_lowvram.json
```

1. Restart the `test1` ComfyUI backend.
2. Drag the workflow JSON onto the ComfyUI canvas.
3. Confirm that all four model loaders resolve correctly.
4. Paste a prompt into `PROMPT (feeds every stage)`.
5. Check resolution, duration, seed, and sampling steps.
6. Click `Queue`.

Output normally appears under:

```text
D:\ComfyUI\workspace\test1\ComfyUI\output
```

## 8. Image-to-Video

Open:

```text
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3\04_i2v_fl2v_4step_sla_lowvram.json
```

1. Drag the workflow JSON onto the ComfyUI canvas.
2. Upload the starting frame in the `LoadImage` node titled `First frame image`.
3. Describe the intended motion, camera movement, and sound in the main prompt box.
4. Set the output aspect ratio and pixel target in `ResolutionSelector`.
5. Set the target duration in `PrimitiveFloat`.
6. Keep `BasicScheduler` at 4 steps for the first test.
7. Click `Queue`.

Recommended first test:

```text
Input image: 16:9
Output: 1152 × 640
Duration: 4–6 seconds
Sampling steps: 4
Frame rate: 24 FPS
```

Crop the source image to the same aspect ratio as the output. A significantly different input ratio may stretch people or objects when the first frame is adapted to the video canvas.

### First-and-last-frame interpolation

To guide the video from one image to another:

1. Add a second `LoadImage` node and load the target ending frame.
2. Connect its image output to `last_frame` on `MiniMax H3 Image to Video`.
3. Keep the first image connected to `first_frame`.
4. Describe a plausible transition between the two images without conflicting scene instructions.

Image-to-video prompts should focus on what happens next while explicitly preserving the subject, clothes, and environment from the input image. One main action plus one camera movement is generally more stable than many simultaneous actions.

## 9. Default Workflow Parameters

The current `01_reference_4step_sla_lowvram.json` settings are:

```text
Aspect ratio: 16:9
Pixel target: 0.7 MP
Alignment multiple: 32
Actual resolution: 1152 × 640
Duration input: 10.0 seconds
Actual frame count: 243
Output frame rate: 24 FPS
Actual duration: about 10.125 seconds
Sampling steps: 4
Scheduler: simple
Seed: 42
FFN chunks: 4
```

### Change duration

Edit the `PrimitiveFloat` node:

```text
4.0  → about 4 seconds
6.0  → about 6 seconds
8.0  → about 8 seconds
10.0 → about 10 seconds
```

The connected expression converts seconds to MiniMax H3's valid `17k+5` frame grid.

### Change resolution

The current `ResolutionSelector` values are:

```text
Aspect Ratio: 16:9 (Widescreen)
Megapixels: 0.7
Multiple: 32
```

This calculates `1152×640`. Do not edit the fallback width and height displayed inside `MiniMaxH3ImageToVideo`; connected inputs override them.

### 1920×1080 output

MiniMax H3 dimensions advance in 32-pixel steps. `1920` is divisible by 32, but `1080` is not:

```text
1920 / 32 = 60
1080 / 32 = 33.75
1088 / 32 = 34
```

Use this native generation size:

```text
Aspect Ratio: 16:9
Megapixels: 2.0
Multiple: 32
Result: 1920 × 1088
```

Crop four pixels from both the top and bottom for an exact `1920×1080` delivery. Because `1920×1088` contains about 2.8 times as many pixels as `1152×640`, low-VRAM systems should generate at `1152×640` first and upscale afterward.

## 10. Troubleshooting

### Hugging Face connection timeout

If `curl` or the CLI cannot reach `huggingface.co:443`, use:

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

Then retry with `hf download`, which supports caching, validation, and resuming.

### Incorrect model size

Approximate Windows sizes:

```text
Fused diffusion model: 19.54 GiB
Video VAE: 2.95 GiB
Audio VAE: 0.56 GiB
Heretic text encoder: 14.61 GiB
```

A `.safetensors` file that is only a few KB or MB is usually an error page, a Git LFS pointer, or an incomplete download. Run the corresponding `hf download` command again.

### Workflow reports missing models

Check that:

1. The files exist under the correct `D:\ComfyUI\models` subdirectories.
2. `extra_model_paths.yaml` exists in the active workspace's ComfyUI directory.
3. Workflow file names exactly match the downloaded files.
4. The ComfyUI backend was fully restarted after the path configuration changed.
5. You re-imported the corrected JSON instead of using an older browser-cached workflow.

### Missing or red nodes

Install the three custom-node packs listed in Section 5, then restart ComfyUI.

### CUDA out of memory

Reduce load in this order:

1. Use the low-VRAM workflow.
2. Reduce duration from 10 seconds to 4 seconds.
3. Use `960×544` or `1152×640`.
4. Close other GPU-intensive applications.
5. Keep sampling at 4 steps.
6. Generate at low resolution and upscale afterward.

### Duplicated directories after `hf download`

The CLI preserves repository-relative paths. For example:

```powershell
hf download OWNER/REPO "diffusion_models/model.safetensors" --local-dir "D:\ComfyUI\models"
```

creates:

```text
D:\ComfyUI\models\diffusion_models\model.safetensors
```

Do not use `D:\ComfyUI\models\diffusion_models` as `--local-dir` for that command, or you may create a duplicated `diffusion_models\diffusion_models` path.

## 11. References

- Original Chinese tutorial: <https://www.freedidi.com/25385.html>
- Fused Turbo model: <https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot>
- Low-VRAM workflows: <https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/workflows/lowvram>
- Kijai MiniMax H3 experiments: <https://huggingface.co/Kijai/MiniMax-H3-experimental>
- Official ComfyUI model package: <https://huggingface.co/Comfy-Org/MiniMax-H3>
- Heretic text encoder: <https://huggingface.co/Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4>
- ComfyUI-KJNodes: <https://github.com/kijai/ComfyUI-KJNodes>
- ComfyUI-MAINodes: <https://github.com/matlowai/ComfyUI-MAINodes>
- H3 SLA sparse-attention node: <https://github.com/ethanfel/ComfyUI-PlagueKind-Nodes-only-sparse>


