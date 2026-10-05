# LTX 2.3 + PinkCherry Uncensored Q5 GGUF: ComfyUI Desktop Setup Guide for 8GB VRAM

> Platform: Windows and ComfyUI Desktop  
> Target hardware: NVIDIA GPU with 8GB VRAM; tested configuration is based on an RTX 3070 Laptop GPU  
> Example workspace: `D:\ComfyUI\workspace\<YOUR_WORKSPACE>`  
> Example ComfyUI root: `D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI`  
> Guide date: 2026-10-03

## 1. Install the Hugging Face CLI

Open PowerShell and run:

```powershell
py -m pip install --upgrade huggingface_hub
hf --help
```

Sign in to avoid anonymous rate limits and to access repositories that require license or sensitive-content confirmation:

```powershell
hf auth login
```

Never paste your Hugging Face token into a public issue, screenshot, or repository.

## 2. Create the model directories

This guide keeps the downloaded files in a central model library on drive D:

```powershell
$modelRoot = "D:\ComfyUI\models"

New-Item -ItemType Directory -Force -Path "$modelRoot\diffusion_models"
New-Item -ItemType Directory -Force -Path "$modelRoot\loras\LTX-2.3"
New-Item -ItemType Directory -Force -Path "$modelRoot\vae"
New-Item -ItemType Directory -Force -Path "$modelRoot\text_encoders"
New-Item -ItemType Directory -Force -Path "$modelRoot\latent_upscale_models"
```

Change the drive or base directory if your installation uses another location.

## 3. Download the models and workflow

### 3.1 Download the PinkCherry uncensored Q5 GGUF model

**Model type: uncensored / NSFW video-generation model.**

```powershell
hf download SexGod1979/PinkCherry_NSFW_LTX23 `
  "v1.8/PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf" `
  --local-dir "D:\ComfyUI\models\diffusion_models\PinkCherry-LTX23-v1.8"
```

Expected path:

```text
D:\ComfyUI\models\diffusion_models\PinkCherry-LTX23-v1.8\v1.8\PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf
```

The Hugging Face repository is marked as sensitive. You may need to sign in and acknowledge the content warning before downloading.

### 3.2 Download the uncensored model workflow

**Workflow type: workflow supplied for the PinkCherry uncensored model.** It may contain adult-oriented example prompts.

```powershell
hf download SexGod1979/PinkCherry_NSFW_LTX23 `
  --include "workflows/*.json" `
  --local-dir "D:\ComfyUI\ComfyUI-Workflows\PinkCherry-LTX23-v1.8"
```

### 3.3 Download the official LTX 2.3 distilled LoRA

```powershell
hf download Lightricks/LTX-2.3 `
  "ltx-2.3-22b-distilled-lora-384-1.1.safetensors" `
  --local-dir "D:\ComfyUI\models\loras\LTX-2.3"
```

Expected path:

```text
D:\ComfyUI\models\loras\LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors
```

### 3.4 Download the three VAE files

```powershell
hf download Kijai/LTX2.3_comfy `
  "vae/LTX23_video_vae_bf16.safetensors" `
  --local-dir "D:\ComfyUI\models"

hf download Kijai/LTX2.3_comfy `
  "vae/taeltx2_3.safetensors" `
  --local-dir "D:\ComfyUI\models"

hf download Kijai/LTX2.3_comfy `
  "vae/LTX23_audio_vae_bf16.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

Expected files:

```text
D:\ComfyUI\models\vae\LTX23_video_vae_bf16.safetensors
D:\ComfyUI\models\vae\taeltx2_3.safetensors
D:\ComfyUI\models\vae\LTX23_audio_vae_bf16.safetensors
```

### 3.5 Download the LTX 2.3 text projection

```powershell
hf download Kijai/LTX2.3_comfy `
  "text_encoders/ltx-2.3_text_projection_bf16.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

Expected path:

```text
D:\ComfyUI\models\text_encoders\ltx-2.3_text_projection_bf16.safetensors
```

### 3.6 Download the Gemma 3 12B FP4 Mixed text encoder

The full BF16 Gemma is not recommended for an 8GB GPU. This configuration uses the smaller FP4 Mixed variant.

```powershell
curl.exe -L `
  "https://huggingface.co/Comfy-Org/ltx-2/resolve/main/split_files/text_encoders/gemma_3_12B_it_fp4_mixed.safetensors?download=true" `
  -o "D:\ComfyUI\models\text_encoders\gemma_3_12B_it_fp4_mixed.safetensors"
```

### 3.7 Download the LTX 2.3 spatial upscaler

```powershell
hf download Lightricks/LTX-2.3 `
  "ltx-2.3-spatial-upscaler-x2-1.1.safetensors" `
  --local-dir "D:\ComfyUI\models\latent_upscale_models"
```

## 4. What the modified workflow changes

Recommended workflow name:

```text
PinkCherry_LTX23_Q5_GGUF_8GB.json
```

The modified workflow:

- Replaces the regular checkpoint loader with `Unet Loader (GGUF)`.
- Loads `PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf`.
- Applies the official LTX 2.3 distilled LoRA.
- Uses the FP4 Mixed Gemma encoder instead of the much larger BF16/Heretic encoder.
- Removes the unconnected `Fast Groups Bypasser (rgthree)` node.
- Enables low-VRAM feed-forward chunking.
- Defaults to `832×480`, 3 seconds, and 24 FPS.
- Supports both text-to-video and image-to-video.

### 4.1 Replace the checkpoint loader

1. Open the original workflow and save it under a new filename.
2. Find and delete the `CheckpointLoaderSimple` node that expects the full BF16 PinkCherry checkpoint.
3. Add `Unet Loader (GGUF)` from the `ComfyUI-GGUF` node pack.
4. Select:

```text
PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf
```

5. Connect its purple `MODEL` output to the `model` input of `LoraLoaderModelOnly`.

The main model path in the active workspace should be:

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\models\diffusion_models\PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf
```

### 4.2 Configure the distilled LoRA

In `LoraLoaderModelOnly`, select:

```text
LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors
```

The model path should follow this order:

```text
Unet Loader (GGUF)
        │ MODEL
        ▼
LoraLoaderModelOnly
        │ MODEL
        ▼
Low-VRAM patching and sampler nodes
```

Keep the LoRA strength from the original workflow unless you are deliberately testing another value.

### 4.3 Configure the two text-encoder inputs

In `DualCLIPLoader`, select:

```text
Encoder 1: gemma_3_12B_it_fp4_mixed.safetensors
Encoder 2: ltx-2.3_text_projection_bf16.safetensors
Type: ltxv
```

Keep the existing `CLIP` output connection intact.

### 4.4 Configure VAE models

| Purpose | Node | File |
|---|---|---|
| Video encode/decode | `VAELoader` | `LTX23_video_vae_bf16.safetensors` |
| Audio encode/decode | `VAELoaderKJ` | `LTX23_audio_vae_bf16.safetensors` |
| Sampler previews | `VAELoader` | `taeltx2_3.safetensors` |

For the audio VAE, start with:

```text
device: main_device
weight_dtype: bf16
```

### 4.5 Configure spatial upscaling

In `LatentUpscaleModelLoader`, select:

```text
ltx-2.3-spatial-upscaler-x2-1.1.safetensors
```

Keep its connection to `LTXVLatentUpsampler`. The second pass costs extra time but improves final clarity.

### 4.6 Enable the low-VRAM node

Find `LTXV Chunk FeedForward`. If it is bypassed or greyed out, select it and press `Ctrl+B`, or use the node's context menu to set its mode to normal/always.

Recommended initial values:

```text
chunks: 2
dim_threshold: 4096
```

Chunking lowers peak VRAM use but can reduce speed slightly.

### 4.7 Remove the optional rgthree node

If the workflow reports a missing `Fast Groups Bypasser (rgthree)` node, delete the node titled `ENABLE PROMPT ENHANCER`. In the original workflow it has no data links and only provides a group-toggle shortcut.

### 4.8 Set safe 8GB defaults

In the Video Settings group, use:

```text
Width: 832
Height: 480
Duration: 3 seconds
Frame rate: 24 FPS
```

Do not use the original `1280×1280`, 13-second configuration for the first run. At 24 FPS it produces roughly 313 frames and is far too heavy for an 8GB GPU.

### 4.9 Choose I2V or T2V

Use the boolean control named `Text To Video (no image ref)`:

- `false`: image-to-video; select an input image.
- `true`: text-to-video; the reference image is not used.

### 4.10 Save and reload

1. Save the modified workflow under the new Q5 filename.
2. Close the currently open graph.
3. Reopen the saved JSON from disk.
4. Check Workflow Overview for missing nodes or models.
5. If model lists are stale, fully quit ComfyUI Desktop, including its tray process, and restart it.

## 5. Identify the active ComfyUI Desktop workspace

ComfyUI Desktop may contain several workspaces. Models placed in an old workspace are not automatically visible in a new one.

Example locations:

```text
Desktop application: D:\Program Files\Comfy Desktop
Workspace:           D:\ComfyUI\workspace\<YOUR_WORKSPACE>
ComfyUI root:        D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI
```

Confirm the real paths in:

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\user\comfyui.log
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\logs\comfyui.log
```

Look for `ComfyUI Path`, `Python executable`, and `User directory`.

## 6. Install required custom nodes

Preferred method: use ComfyUI Manager to install:

- `ComfyUI-GGUF`
- `ComfyUI-KJNodes`
- `ComfyUI-VideoHelperSuite`

Restart ComfyUI Desktop after installation.

Manual installation example:

```powershell
Set-Location "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\custom_nodes"

git clone https://github.com/city96/ComfyUI-GGUF.git
git clone https://github.com/kijai/ComfyUI-KJNodes.git comfyui-kjnodes
git clone https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite.git comfyui-videohelpersuite
```

Install the GGUF requirements with the Python executable shown in your ComfyUI log:

```powershell
& "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\standalone-env\python.exe" `
  -m pip install -r `
  "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\custom_nodes\ComfyUI-GGUF\requirements.txt"
```

## 7. Expose the central model library to the active workspace

The most reliable option for ComfyUI Desktop is to place models directly inside the active workspace. When the central library and workspace are on the same drive, hard links avoid duplicate disk usage.

Run PowerShell as Administrator:

```powershell
$srcRoot = "D:\ComfyUI\models"
$dstRoot = "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\models"

New-Item -ItemType Directory -Force -Path "$dstRoot\diffusion_models"
New-Item -ItemType Directory -Force -Path "$dstRoot\loras\LTX-2.3"
New-Item -ItemType Directory -Force -Path "$dstRoot\vae"
New-Item -ItemType Directory -Force -Path "$dstRoot\text_encoders"
New-Item -ItemType Directory -Force -Path "$dstRoot\latent_upscale_models"

New-Item -ItemType HardLink `
  -Path "$dstRoot\diffusion_models\PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf" `
  -Target "$srcRoot\diffusion_models\PinkCherry-LTX23-v1.8\v1.8\PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf"

New-Item -ItemType HardLink `
  -Path "$dstRoot\loras\LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors" `
  -Target "$srcRoot\loras\LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors"

New-Item -ItemType HardLink `
  -Path "$dstRoot\vae\LTX23_video_vae_bf16.safetensors" `
  -Target "$srcRoot\vae\LTX23_video_vae_bf16.safetensors"

New-Item -ItemType HardLink `
  -Path "$dstRoot\vae\taeltx2_3.safetensors" `
  -Target "$srcRoot\vae\taeltx2_3.safetensors"

New-Item -ItemType HardLink `
  -Path "$dstRoot\vae\LTX23_audio_vae_bf16.safetensors" `
  -Target "$srcRoot\vae\LTX23_audio_vae_bf16.safetensors"

New-Item -ItemType HardLink `
  -Path "$dstRoot\text_encoders\gemma_3_12B_it_fp4_mixed.safetensors" `
  -Target "$srcRoot\text_encoders\gemma_3_12B_it_fp4_mixed.safetensors"

New-Item -ItemType HardLink `
  -Path "$dstRoot\text_encoders\ltx-2.3_text_projection_bf16.safetensors" `
  -Target "$srcRoot\text_encoders\ltx-2.3_text_projection_bf16.safetensors"

New-Item -ItemType HardLink `
  -Path "$dstRoot\latent_upscale_models\ltx-2.3-spatial-upscaler-x2-1.1.safetensors" `
  -Target "$srcRoot\latent_upscale_models\ltx-2.3-spatial-upscaler-x2-1.1.safetensors"
```

Hard links require source and destination to be on the same volume. If a destination already exists, inspect it before replacing anything.

## 8. Recommended 8GB settings

| Use case | Resolution | Duration | FPS | Notes |
|---|---:|---:|---:|---|
| Fast diagnostics | 640×384 | 3 s | 24 | Highest chance of success |
| Stable test | 832×480 | 3 s | 24 | Recommended default |
| Balanced | 832×480 | 5 s | 24 | Noticeably slower |
| Higher clarity | 960×544 | 3–5 s | 24 | May require heavier offloading |
| Not recommended on 8GB | 1280×720+ | 10 s+ | 24+ | Very slow or likely to fail |

The workflow converts duration to a frame count compatible with `8n + 1`:

- 3 seconds at 24 FPS: about 73 frames.
- 5 seconds at 24 FPS: about 121 frames.
- 10 seconds at 24 FPS: about 241 frames.

Duration scales generation time approximately linearly. Increasing both width and height is substantially more expensive. Avoid 48, 50, or 60 FPS on an 8GB GPU.

## 9. Prompt examples

### Text-to-video

Describe the full subject, environment, lighting, motion, camera, and audio:

```text
A rainy city street at night. An adult woman in a red raincoat stands beneath a transparent umbrella beside a warmly lit noodle shop. She slowly turns toward the distant street as her wet hair and coat move in a light breeze. Rain falls on the umbrella and pavement, blue and amber neon reflections ripple in shallow water, and steam rises from a street vent. The camera slowly pushes forward. Fine rain, distant tires passing through water, and quiet dish sounds from the shop are audible.
```

### Image-to-video

Describe changes from the reference image instead of repeating its static appearance:

```text
The woman slowly turns toward the distant street and blinks once. Her wet hair and the hem of her coat move naturally in a light breeze. She slightly adjusts the transparent umbrella. Rain continues to strike the umbrella and pavement, neon reflections shimmer in shallow water, and white steam drifts across the right side of the street. Background pedestrians walk naturally while the camera slowly pushes forward. Fine rain, distant vehicles passing through water, and quiet dish sounds from the shop are audible.
```

## 10. Disk and memory expectations

Approximate major file sizes:

| File | Approximate size |
|---|---:|
| PinkCherry Q5 GGUF | 15.9 GB |
| LTX 2.3 distilled LoRA | 7.61 GB |
| Gemma 3 12B FP4 Mixed | 9.45 GB |
| LTX text projection | 2.31 GB |
| Video VAE | 1.45 GB |
| Audio VAE | 365 MB |
| Tiny VAE | 23.5 MB |
| Spatial upscaler | 996 MB |

Keep at least 50GB free; 70GB or more is preferable for caches and output videos. An 8GB GPU should ideally be paired with at least 32GB of system RAM and a properly sized Windows page file.

## 11. Troubleshooting

### Models are downloaded but still reported missing

The files are probably in a different ComfyUI Desktop workspace. Confirm the active `ComfyUI Path` in the log and place or link the models under that workspace's `ComfyUI\models` directory. Fully restart the Desktop application afterward.

### `UnetLoaderGGUF` is missing

Install `ComfyUI-GGUF`, install its requirements using ComfyUI's own Python environment, and restart. The Q5 GGUF model must be loaded with `Unet Loader (GGUF)`, not `CheckpointLoaderSimple`.

### `Fast Groups Bypasser (rgthree)` is missing

The supplied Q5 workflow removes this unconnected UI-only node. If an old copy still reports it, close the old graph without saving and reopen the modified workflow.

### `No module named 'triton'`

KJNodes may report that `PatchTritonVAE` is unavailable. This is an optional-node warning. It can normally be ignored when the workflow does not use `PatchTritonVAE`.

### CUDA out of memory

1. Use `640×384`, 3 seconds, and 24 FPS.
2. Enable `LTXV Chunk FeedForward`.
3. Close other GPU-heavy applications.
4. Keep FPS at 24.
5. Use the Q5 GGUF transformer and FP4 Gemma.
6. Restart ComfyUI to release previously loaded models.

### Generation is extremely slow

This is expected when an 8GB GPU runs a 22B model with CPU/RAM offloading. Check that you did not accidentally select `1280×1280`, 10–20 seconds, or 48/50 FPS. SSD storage and sufficient page-file space also matter.

### The JSON changed but the open graph did not

The current graph is cached in the browser frontend. Close it without saving, reopen the JSON from disk, and restart ComfyUI Desktop if necessary.

## 12. Verify installed files

```powershell
$root = "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\models"

Get-ChildItem -LiteralPath $root -Recurse -File |
  Where-Object {
    $_.Name -match "PinkCherry|ltx-2.3|LTX23|taeltx|gemma_3_12B"
  } |
  Select-Object FullName, Length
```

Verify the official distilled LoRA:

```powershell
Get-FileHash `
  -LiteralPath "D:\ComfyUI\models\loras\LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors" `
  -Algorithm SHA256
```

Expected SHA256:

```text
f5d4953f3386197a4b4f5abdb17616ff256171e8075c111d6e7d2dfa6e823b3a
```

## 13. Sources

- LTX 2.3 official repository: <https://huggingface.co/Lightricks/LTX-2.3>
- LTX 2.3 distilled LoRA: <https://huggingface.co/Lightricks/LTX-2.3/blob/main/ltx-2.3-22b-distilled-lora-384-1.1.safetensors>
- Kijai LTX 2.3 ComfyUI VAE files: <https://huggingface.co/Kijai/LTX2.3_comfy/tree/main/vae>
- Kijai text projection: <https://huggingface.co/Kijai/LTX2.3_comfy/tree/main/text_encoders>
- Comfy-Org Gemma FP4: <https://huggingface.co/Comfy-Org/ltx-2/blob/main/split_files/text_encoders/gemma_3_12B_it_fp4_mixed.safetensors>
- ComfyUI-GGUF: <https://github.com/city96/ComfyUI-GGUF>
- PinkCherry LTX 2.3: <https://huggingface.co/SexGod1979/PinkCherry_NSFW_LTX23>

## 14. Final checklist

- [ ] The active ComfyUI Desktop workspace is known.
- [ ] `ComfyUI-GGUF`, `ComfyUI-KJNodes`, and `ComfyUI-VideoHelperSuite` are installed.
- [ ] The Q5 GGUF model appears in `Unet Loader (GGUF)`.
- [ ] The distilled LoRA appears in `LoraLoaderModelOnly`.
- [ ] FP4 Gemma and the LTX text projection appear in `DualCLIPLoader`.
- [ ] Video, audio, and Tiny VAE files are available.
- [ ] The spatial upscaler is available.
- [ ] The workflow no longer requires `Fast Groups Bypasser (rgthree)`.
- [ ] The first test uses `832×480`, 3 seconds, and 24 FPS.
- [ ] I2V uses `false`; T2V uses `true` for the no-image-reference switch.
- [ ] ComfyUI Desktop was fully restarted after model or custom-node changes.

