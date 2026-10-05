# ComfyUI Video Model Guides

A bilingual collection of practical ComfyUI deployment and generation guides for local video models.

ComfyUI 本地视频模型部署与生成教程合集，按模型分类整理，并提供中英文版本。

## Guides / 教程目录

### LTX 2.3 + PinkCherry

- [中文教程](guides/ltx23-pinkcherry/README.md)
- [English Guide](guides/ltx23-pinkcherry/README_EN.md)

Covers the PinkCherry uncensored Q5 GGUF model on ComfyUI Desktop, including an 8 GB VRAM setup.

包含 PinkCherry 无审查 Q5 GGUF 模型在 ComfyUI Desktop 上的部署、8 GB 显存配置和工作流说明。

### MiniMax H3

- [中文教程](guides/minimax-h3/README.md)
- [English Guide](guides/minimax-h3/README_EN.md)

Covers the MiniMax H3 Fused Turbo INT8 ConvRot model, video/audio VAEs, Heretic text encoder, low-VRAM workflows, text-to-video, and image-to-video generation.

包含 MiniMax H3 Fused Turbo INT8 ConvRot 主模型、视频与音频 VAE、Heretic 文本编码器、低显存工作流、文生视频和图生视频教程。

## Repository Structure / 仓库结构

```text
.
├── README.md
└── guides
    ├── ltx23-pinkcherry
    │   ├── README.md
    │   └── README_EN.md
    └── minimax-h3
        ├── README.md
        └── README_EN.md
```

## Notes / 注意事项

Model licenses, platform rules, and local laws still apply. Check upstream model pages before downloading or using model weights.

使用或下载模型前，请阅读上游模型页面的许可证和使用要求，并遵守平台规则及所在地法律法规。
