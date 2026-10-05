# MiniMax H3 ComfyUI 本地部署与工作流指南

本目录提供一套面向 Windows + ComfyUI 的 MiniMax H3 本地方案，覆盖：

- 文生视频（T2V）
- 图生视频与分段续片（I2V）
- 文生图片（T2I）
- 3 段 × 约 10 秒的 30 秒视频制作流程
- 原生音频、连续背景音乐和后期拼接

工作流已经按以下本地目录适配：

```text
ComfyUI 数据目录：D:\ComfyUI
ComfyUI 程序目录：D:\ComfyUI\workspace\test1\ComfyUI
共享模型目录：D:\ComfyUI\models
工作流目录：D:\ComfyUI\ComfyUI-Workflows\MinMax_H3
```

如果你的目录不同，请替换本文中的路径。仓库不包含任何模型权重。

## 目录结构

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

## 1. 环境要求

- Windows 10/11
- NVIDIA GPU
- ComfyUI 0.37.0 或更高版本（文生图片插件要求）
- 建议 32 GB 以上内存，64 GB 更稳
- 至少预留 50–70 GB 磁盘空间
- Git 与 Hugging Face CLI

安装 Hugging Face CLI：

```powershell
py -m pip install -U huggingface_hub
```

网络无法直接访问 Hugging Face 时，可在当前 PowerShell 窗口设置镜像：

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

## 2. 下载模型

### 2.1 主扩散模型

来源：[MATLOWAI/minimax-h3-fused-turbo-int8-convrot](https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/diffusion_models)

```powershell
hf download MATLOWAI/minimax-h3-fused-turbo-int8-convrot `
  "diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

目标文件：

```text
D:\ComfyUI\models\diffusion_models\minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors
```

### 2.2 视频 VAE

来源：[Kijai/MiniMax-H3-experimental](https://huggingface.co/Kijai/MiniMax-H3-experimental)

```powershell
hf download Kijai/MiniMax-H3-experimental `
  "minimax_h3_video_vae_int8_convrot.safetensors" `
  --local-dir "D:\ComfyUI\models\vae"
```

### 2.3 音频 VAE

```powershell
hf download Comfy-Org/MiniMax-H3 `
  "vae/minimax_h3_audio_vae_fp32.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

### 2.4 Heretic 文本编码器

来源：[Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4](https://huggingface.co/Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4)

```powershell
hf download Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4 `
  "qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors" `
  --local-dir "D:\ComfyUI\models\text_encoders"
```

最终应具备：

```text
D:\ComfyUI\models\diffusion_models\minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors
D:\ComfyUI\models\vae\minimax_h3_video_vae_int8_convrot.safetensors
D:\ComfyUI\models\vae\minimax_h3_audio_vae_fp32.safetensors
D:\ComfyUI\models\text_encoders\qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors
```

## 3. 让工作区读取共享模型

创建：

```text
D:\ComfyUI\workspace\test1\ComfyUI\extra_model_paths.yaml
```

内容：

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

保存后必须完全重启 ComfyUI 后端，仅刷新浏览器不会重新加载模型路径。

## 4. 安装自定义节点

使用 ComfyUI 的“自定义节点管理器”安装：

| 节点包 | 用途 | 哪些工作流需要 |
|---|---|---|
| ComfyUI-KJNodes | 数学表达式、分辨率与低显存分块节点 | T2V、I2V |
| ComfyUI-H3-SLA-Attention | `H3SLAAttention` | T2V、I2V |
| MiniMax H3 Image Studio（作者 astropuzzo） | H3 文生图片节点 | T2I |

安装后完全重启 ComfyUI，并在浏览器中按 `Ctrl+F5`。

文生图片插件也可以手动安装：

```cmd
cd /d D:\ComfyUI\workspace\test1\ComfyUI\custom_nodes
git clone https://github.com/astropuzzo/ComfyUI-MiniMax-H3-Image-Studio.git
```

## 5. 工作流清单

详见 [workflows/README.md](workflows/README.md)。

| 文件 | 用途 | 默认设置 |
|---|---|---|
| `01_T2V_10s_低显存_续片优化.json` | 第一段文生视频 | 1152×640、24 fps、约 10.1 秒、4 步 |
| `04_I2V_10s_低显存_续片优化.json` | 第 2/3 段图生视频 | 与 T2V 相同，并读取上一段最后一帧 |
| `02_T2I_ImageStudio_本地模型适配.json` | 文生图片 | 16:9、约 0.98 MP、5 帧择优 |

T2V 和 I2V 优化版会在保存视频的同时自动执行：

```text
VAEDecode → Get Image from Batch（-1）→ Save Image
```

生成的最后一帧位于 ComfyUI 的输出目录下，可上传到下一段 I2V 工作流。

## 6. 视频参数

默认视频参数：

```text
宽高比：16:9
目标像素量：0.7 MP
实际分辨率：1152 × 640
时长输入：10.0 秒
实际帧数：243 帧
帧率：24 fps
实际时长：10.125 秒
采样器：res_multistep
调度器：simple
步数：4
Seed：42
```

帧数会被表达式调整到 MiniMax H3 使用的 `17k+5` 序列。不要直接把节点改成任意帧数。

### 分辨率建议

- 调试：960×544 或 1152×640
- H3 原生约 1 MP 画布：1344×768
- 不建议直接生成 1920×1080
- 需要 1080p 时，先生成稳定版本，再统一放大并裁切到 1920×1080

ComfyUI 官方说明中，H3 开放权重主要面向约 1 MP、24 fps、最长约 15 秒的视频。长视频应拆分生成。

## 7. 30 秒连续视频流程

1. 使用 T2V 工作流生成第 1 段。
2. 找到自动保存的 `t2v_last_frame_*.png`。
3. 将该图上传到 I2V 工作流，生成第 2 段。
4. 将第 2 段自动保存的 `i2v_last_frame_*.png` 用于第 3 段。
5. 每段提示词都写出时间轴，并规定最后约 0.8–1 秒的固定结束构图。
6. 在剪辑软件中顺序拼接三段。
7. 如果接缝短暂停顿，删除第 2、3 段开头重复的第一帧（24 fps 下约 0.0417 秒）。
8. 原声音频接缝使用 0.15–0.3 秒交叉淡化。
9. 背景音乐只添加一次，覆盖完整时间线，不要每段分别生成音乐。

完整案例见 [examples/video-30s-last-tram.md](examples/video-30s-last-tram.md)。

## 8. 文生图片

先在 ComfyUI 中打开 **自定义节点管理器**，搜索并安装：

```text
MiniMax H3 Image Studio
作者：astropuzzo
```

请确认选择的是作者为 `astropuzzo` 的节点包，不要安装名称相似的其他插件。安装完成后，必须完全关闭并重新启动 ComfyUI 客户端和后端；只刷新工作流页面不会使新节点生效。重新启动后，可以在浏览器中再按一次 `Ctrl+F5` 强制刷新前端资源。

节点包生效后，导入：

```text
02_T2I_ImageStudio_本地模型适配.json
```

工作流已经匹配本教程的 fused 主模型、Heretic 编码器和 INT8 视频 VAE。它使用 Image Studio 的兼容节点；首次载入若提示以下节点缺失，说明插件没有安装或浏览器尚未刷新：

```text
H3ImageDecode
H3ImageFrameSelector
H3ImageResolutionPreset
H3ImageSamplingPreset
H3TextToImagePrepare
```

测试案例见 [examples/t2i-desert-observatory.md](examples/t2i-desert-observatory.md)。

## 9. 拼接与背景音乐

最简单的方法是使用剪映专业版、Premiere 或 DaVinci Resolve：

- 时间线设为 1152×640、24 fps。
- 三段按顺序排列，必要时删除续片段开头的重复帧。
- 画面优先硬切；只有轻微跳动时才使用 2–4 帧的短叠化。
- 原声音频接缝做短交叉淡化。
- 背景音乐使用一条完整音轨覆盖全片。
- 全部剪完后再统一放大至 1920×1080。

也可以使用 FFmpeg：

```text
file 'segment01.mp4'
file 'segment02.mp4'
file 'segment03.mp4'
```

将以上内容保存为 `list.txt`，然后执行：

```cmd
ffmpeg -f concat -safe 0 -i list.txt -c copy merged_model_audio.mp4
```

如果编码参数不同，统一重编码：

```cmd
ffmpeg -f concat -safe 0 -i list.txt -c:v libx264 -preset medium -crf 18 -c:a aac -b:a 192k merged_model_audio.mp4
```

## 10. 常见问题

### Hugging Face 连接超时

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

优先使用 `hf download`，它支持缓存、校验和断点续传。

### 模型只有几 KB 或几 MB

通常下载到了 Git LFS 指针或错误页面。删除错误文件后重新使用 `hf download`。

### 工作流提示缺模型

1. 检查文件是否在正确的模型目录。
2. 检查 `extra_model_paths.yaml`。
3. 工作流中只填写文件名，不添加不存在的 `minimax_h3/` 前缀。
4. 完全重启 ComfyUI。

### 工作流提示缺节点

通过节点名称判断对应插件，在自定义节点管理器中安装，然后重启并 `Ctrl+F5`。

### CUDA out of memory

按顺序尝试：

1. 降低到 1152×640 或 960×544。
2. 将时长降到 4–6 秒。
3. 关闭占用显存的其他程序。
4. 保持 4 步低显存工作流。
5. 分段生成，最后统一放大。

## 11. 来源与说明

- [ComfyUI MiniMax H3 官方指南](https://docs.comfy.org/tutorials/video/minimax/minimax-h3)
- [MATLOWAI fused 模型与低显存工作流](https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot)
- [Kijai MiniMax H3 experimental](https://huggingface.co/Kijai/MiniMax-H3-experimental)
- [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3)
- [Heretic 文本编码器](https://huggingface.co/Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4)
- [MiniMax H3 Image Studio](https://github.com/astropuzzo/ComfyUI-MiniMax-H3-Image-Studio)

工作流根据上游社区文件修改。使用、再分发或商业使用前，请分别检查模型、插件和工作流上游项目的许可证。
