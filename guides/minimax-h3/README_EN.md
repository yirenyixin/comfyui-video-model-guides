# MiniMax H3 ComfyUI Local Setup and Workflow Guide

This directory contains a Windows-oriented MiniMax H3 setup for ComfyUI, including text-to-video, image-to-video continuation, text-to-image, native audio, and a three-shot 30-second production workflow.

The prepared workflows assume:

```text
ComfyUI data: D:\ComfyUI
ComfyUI app: D:\ComfyUI\workspace\test1\ComfyUI
Shared models: D:\ComfyUI\models
Workflows: D:\ComfyUI\ComfyUI-Workflows\MinMax_H3
```

Replace these paths if your installation differs. Model weights are not included.

## Repository layout

```text
minimax-h3/
├─ README.md
├─ README_EN.md
├─ workflows/
│  ├─ README.md
│  ├─ 01_T2V_10s_低显存_续片优化.json
│  ├─ 02_T2I_ImageStudio_本地模型适配.json
│  └─ 04_I2V_10s_低显存_续片优化.json
└─ examples/
   ├─ t2i-desert-observatory.md
   └─ video-30s-last-tram.md
```

## Requirements

- Windows 10/11 and an NVIDIA GPU
- ComfyUI 0.37.0 or newer
- 32 GB RAM minimum; 64 GB recommended
- Approximately 50–70 GB of free disk space
- Git and the Hugging Face CLI

```powershell
py -m pip install -U huggingface_hub
```

If Hugging Face is unreachable from your network:

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

## Required model files

| Component | File | Folder |
|---|---|---|
| Diffusion model | `minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors` | `models/diffusion_models/` |
| Text encoder | `qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors` | `models/text_encoders/` |
| Video VAE | `minimax_h3_video_vae_int8_convrot.safetensors` | `models/vae/` |
| Audio VAE | `minimax_h3_audio_vae_fp32.safetensors` | `models/vae/` |

Download commands and source links are provided in the [Chinese guide](README.md).

## Shared model path

Create `D:\ComfyUI\workspace\test1\ComfyUI\extra_model_paths.yaml`:

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

Restart the ComfyUI backend after changing this file.

## Custom nodes

Install these packages through ComfyUI's Custom Node Manager:

| Package | Used by |
|---|---|
| ComfyUI-KJNodes | T2V and I2V |
| ComfyUI-H3-SLA-Attention | T2V and I2V |
| MiniMax H3 Image Studio by astropuzzo | T2I |

Restart ComfyUI and hard-refresh the browser after installation.

## Prepared workflows

| File | Purpose | Defaults |
|---|---|---|
| `01_T2V_10s_低显存_续片优化.json` | First text-to-video shot | 1152×640, 24 fps, about 10.1 s, 4 steps |
| `04_I2V_10s_低显存_续片优化.json` | Continuation shots | Uses the previous shot's final frame |
| `02_T2I_ImageStudio_本地模型适配.json` | Text-to-image | 16:9, about 0.98 MP, selects from five decoded frames |

The video workflows automatically extract and save their final frame for the next I2V shot.

## Making a 30-second video

1. Generate shot 1 with the T2V workflow.
2. Load its automatically saved final frame into the I2V workflow.
3. Generate shots 2 and 3 in the same way.
4. Give each prompt an explicit timeline and reserve the last 0.8–1 second for a stable continuation anchor.
5. Join the shots in an NLE such as DaVinci Resolve, Premiere Pro, or CapCut.
6. Remove the duplicated first frame from shots 2 and 3 if the cut appears to pause.
7. Crossfade generated audio for 0.15–0.3 seconds at each cut.
8. Add one continuous music track after joining the shots.

See [the complete 30-second example](examples/video-30s-last-tram.md).

## Resolution guidance

- Use 960×544 or 1152×640 for testing.
- H3's native canvas is roughly one megapixel; 1344×768 is a suitable 16:9 target.
- Generate first and upscale the completed edit to 1920×1080 afterward.
- Long videos should be generated as shorter continuation shots.

## Troubleshooting

- Missing models: verify folders, filenames, and `extra_model_paths.yaml`, then restart the backend.
- Missing nodes: install the matching package in Custom Node Manager, restart, and hard-refresh.
- CUDA out of memory: lower resolution and duration before changing other parameters.
- Tiny downloaded model: it is likely an incomplete file or Git LFS pointer; download it again with `hf download`.

## Sources and attribution

- [Official ComfyUI MiniMax H3 guide](https://docs.comfy.org/tutorials/video/minimax/minimax-h3)
- [MATLOWAI fused model and low-VRAM workflows](https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot)
- [Kijai MiniMax H3 experimental files](https://huggingface.co/Kijai/MiniMax-H3-experimental)
- [Heretic text encoder](https://huggingface.co/Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4)
- [MiniMax H3 Image Studio](https://github.com/astropuzzo/ComfyUI-MiniMax-H3-Image-Studio)

The workflows are adapted from upstream community files. Review the licenses of every upstream workflow, model, and plugin before redistribution or commercial use.
