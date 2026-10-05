# 工作流说明

本目录拟上传三个已经匹配本教程模型文件名的 ComfyUI 工作流。仓库不会包含模型权重、输出视频或本地备份文件。

## 文件清单

| 文件 | 模式 | 主要修改 |
|---|---|---|
| `01_T2V_10s_低显存_续片优化.json` | 文生视频 | 本地模型名、低显存分块、自动保存最后一帧 |
| `04_I2V_10s_低显存_续片优化.json` | 图生视频 | 上一段最后一帧作为首帧、自动保存新的最后一帧 |
| `02_T2I_ImageStudio_本地模型适配.json` | 文生图片 | 本地主模型、Heretic 编码器、INT8 VAE、4 步配置 |

## 统一模型配置

```text
主模型：minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors
文本编码器：qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors
视频 VAE：minimax_h3_video_vae_int8_convrot.safetensors
音频 VAE：minimax_h3_audio_vae_fp32.safetensors
```

## 自定义节点

视频工作流：

- ComfyUI-KJNodes
- ComfyUI-H3-SLA-Attention

文生图片工作流：

- MiniMax H3 Image Studio（astropuzzo）

## 默认输出

```text
视频：output/video/MiniMax_H3/
续片帧：output/MiniMax_H3/chain/
图片：output/MiniMax_H3_Image/
```

## 上传时不包含

- `*.safetensors`
- `*.gguf`
- `.before-model-path-fix.bak`
- 生成的视频、图片和音频
- Hugging Face 缓存目录
- 用户本地绝对路径或账号凭据

## 上游来源

- 视频低显存工作流：[MATLOWAI/minimax-h3-fused-turbo-int8-convrot](https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/workflows/lowvram)
- 文生图片工作流：[astropuzzo/ComfyUI-MiniMax-H3-Image-Studio](https://github.com/astropuzzo/ComfyUI-MiniMax-H3-Image-Studio)

这些文件是适配版本，不应暗示为上游官方发布。提交前应再次检查上游许可证及归属说明。
