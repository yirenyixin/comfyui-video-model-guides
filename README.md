# ComfyUI Video Model Guides

A bilingual collection of practical ComfyUI deployment and generation guides for local video models.

ComfyUI 本地视频模型部署与生成教程合集，按模型分类整理，并提供中英文版本、测试案例和可导入工作流。

## Guides / 教程目录

### LTX 2.3 + PinkCherry

- [中文教程](guides/ltx23-pinkcherry/README.md)
- [English Guide](guides/ltx23-pinkcherry/README_EN.md)

Covers the PinkCherry uncensored Q5 GGUF model on ComfyUI Desktop, including an 8 GB VRAM setup.

包含 PinkCherry 无审查 Q5 GGUF 模型在 ComfyUI Desktop 上的部署、8 GB 显存配置和工作流说明。

### MiniMax H3

- [中文教程](guides/minimax-h3/README.md)
- [English Guide](guides/minimax-h3/README_EN.md)
- [Workflows / 工作流](guides/minimax-h3/workflows/README.md)
- [Text-to-image example / 文生图案例](guides/minimax-h3/examples/t2i-desert-observatory.md)
- [30-second continuation example / 30 秒续片案例](guides/minimax-h3/examples/video-30s-last-tram.md)

Covers MiniMax H3 fused INT8 models, video/audio VAEs, the Heretic text encoder, low-VRAM workflows, text-to-video, image-to-video continuation, text-to-image, native audio, and multi-shot editing.

包含 MiniMax H3 Fused Turbo INT8 ConvRot 主模型、视频与音频 VAE、Heretic 文本编码器、低显存工作流、文生视频、图生视频续片、文生图片和多段剪辑教程。

## Repository Structure / 仓库结构

```text
.
├─ README.md
└─ guides/
   ├─ ltx23-pinkcherry/
   │  ├─ README.md
   │  └─ README_EN.md
   └─ minimax-h3/
      ├─ README.md
      ├─ README_EN.md
      ├─ examples/
      └─ workflows/
```

## Notes / 注意事项

Model weights and generated media are not stored in this repository. Model licenses, platform rules, and local laws still apply. Check upstream model and workflow pages before downloading, redistributing, or using them commercially.

本仓库不存放模型权重和生成媒体。使用、再分发或商业使用前，请阅读上游模型、工作流和插件的许可证，并遵守平台规则及所在地法律法规。
