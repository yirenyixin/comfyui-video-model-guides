# MiniMax H3 ComfyUI 部署与生成教程

本文档按照当前电脑的实际目录编写：

```text
ComfyUI 数据目录：D:\ComfyUI
当前工作区：D:\ComfyUI\workspace\test1
ComfyUI 程序目录：D:\ComfyUI\workspace\test1\ComfyUI
共享模型目录：D:\ComfyUI\models
工作流目录：D:\ComfyUI\ComfyUI-Workflows\MinMax_H3
```

本方案使用 MiniMax H3 Fused Turbo INT8 ConvRot 主模型、视频/音频 VAE、Heretic 文本编码器和低显存工作流，支持文生视频、图生视频和参考图生视频。

## 1. 部署前准备

建议准备：

- Windows 10 或 Windows 11。
- 最新版 ComfyUI。
- NVIDIA 显卡；低显存工作流可降低峰值显存，但显存越小，生成越慢。
- 至少 32 GB 系统内存，建议 64 GB。
- D 盘至少预留 50～70 GB 空间；Hugging Face 下载缓存还会暂时占用额外空间。
- Python 和 Hugging Face CLI。

如果 PowerShell 中无法识别 `hf` 命令，执行：

```powershell
py -m pip install -U huggingface_hub
```

国内网络建议在每个新 PowerShell 窗口中先设置镜像：

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

此设置只对当前 PowerShell 窗口生效。

## 2. 下载模型

### 2.1 主扩散模型

来源：

<https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/diffusion_models>

执行：

```powershell
hf download MATLOWAI/minimax-h3-fused-turbo-int8-convrot `
  "diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

最终位置：

```text
D:\ComfyUI\models\diffusion_models\minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors
```

文件在 Windows 中约为 19.54 GB；Hugging Face 页面按十进制标注约 21 GB，两者是同一个大小。

### 2.2 视频 VAE

来源：

<https://huggingface.co/Kijai/MiniMax-H3-experimental>

执行：

```powershell
hf download Kijai/MiniMax-H3-experimental `
  "minimax_h3_video_vae_int8_convrot.safetensors" `
  --local-dir "D:\ComfyUI\models\vae"
```

最终位置：

```text
D:\ComfyUI\models\vae\minimax_h3_video_vae_int8_convrot.safetensors
```

Windows 中约为 2.95 GB。

### 2.3 音频 VAE

执行：

```powershell
hf download Comfy-Org/MiniMax-H3 `
  "vae/minimax_h3_audio_vae_fp32.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

最终位置：

```text
D:\ComfyUI\models\vae\minimax_h3_audio_vae_fp32.safetensors
```

Windows 中约为 0.56 GB。

### 2.4 Heretic 文本编码器

来源：

<https://huggingface.co/Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4>

执行：

```powershell
hf download Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4 `
  "qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors" `
  --local-dir "D:\ComfyUI\models\text_encoders"
```

最终位置：

```text
D:\ComfyUI\models\text_encoders\qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors
```

Windows 中约为 14.61 GB。工作流加载后，需要在 `CLIPLoader` 中选择这个文件，并将类型设为 `minimax`。

> 如果 Heretic NVFP4 编码器在当前显卡上提示量化格式不支持，可改用 `Comfy-Org/MiniMax-H3` 仓库中的 `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`。

## 3. 下载低显存工作流

工作流来源：

<https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/workflows/lowvram>

可使用以下命令下载全部低显存工作流：

```powershell
hf download MATLOWAI/minimax-h3-fused-turbo-int8-convrot `
  --include "workflows/lowvram/*.json" `
  --local-dir "D:\ComfyUI\ComfyUI-Workflows"
```

`hf download` 会保留仓库子目录，文件默认位于：

```text
D:\ComfyUI\ComfyUI-Workflows\workflows\lowvram
```

也可以将非 API 版本复制到：

```text
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3
```

推荐使用以下三个不带 `.api` 的文件：

```text
01_reference_4step_sla_lowvram.json      文生视频
04_i2v_fl2v_4step_sla_lowvram.json      图片生视频
05_ref2va_4step_sla_lowvram.json         参考图生视频
```

普通 ComfyUI 界面应使用 `.json`，不要选 `.api.json`。

## 4. 配置 test1 工作区读取共享模型

当前 ComfyUI 实例实际运行于：

```text
D:\ComfyUI\workspace\test1\ComfyUI
```

模型存放于：

```text
D:\ComfyUI\models
```

因此必须创建：

```text
D:\ComfyUI\workspace\test1\ComfyUI\extra_model_paths.yaml
```

文件内容：

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

该文件在当前电脑上已经配置完成。

修改 `extra_model_paths.yaml` 后必须完全停止并重启 ComfyUI 后端；只刷新网页不会重新读取模型路径。

## 5. 安装工作流需要的节点

打开 ComfyUI Manager，更新 ComfyUI，然后搜索并安装：

```text
ComfyUI-KJNodes
ComfyUI-MAINodes
ComfyUI-PlagueKind-Nodes-only-sparse
```

作用：

- `ComfyUI-KJNodes`：提供低显存工作流中的 `MiniMaxChunkFeedForward` 等节点。
- `ComfyUI-MAINodes`：提供 MiniMax H3 Motion Lab、去抖动及相关节点。
- `ComfyUI-PlagueKind-Nodes-only-sparse`：提供 `H3SLAAttention` 稀疏注意力节点。

安装完成后完全重启 ComfyUI。如果打开工作流时仍显示红色节点，使用 Manager 的“安装缺失节点”功能再次检查。

## 6. 工作流模型配置

工作流中的模型选择应为：

```text
UNETLoader
minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors

视频 VAELoader
minimax_h3_video_vae_int8_convrot.safetensors

音频 VAELoader
minimax_h3_audio_vae_fp32.safetensors

CLIPLoader
qwen3vl_32b_heretic_minimax_h3_nvfp4.safetensors
类型：minimax
设备：default
```

不要在文件名前增加不存在的 `minimax_h3/` 子目录。

当前电脑中的以下工作流已经修正为上述文件名：

```text
D:\ComfyUI\ComfyUI-Workflows\01_reference_4step_sla_lowvram.json
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3\01_reference_4step_sla_lowvram.json
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3\04_i2v_fl2v_4step_sla_lowvram.json
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3\05_ref2va_4step_sla_lowvram.json
```

原始文件备份使用以下后缀：

```text
.before-model-path-fix.bak
```

## 7. 运行工作流

### 7.1 文生视频

1. 完全重启 `test1` 工作区的 ComfyUI。
2. 将下面的文件拖进 ComfyUI 页面：

   ```text
   D:\ComfyUI\ComfyUI-Workflows\MinMax_H3\01_reference_4step_sla_lowvram.json
   ```

3. 检查四个模型加载节点是否都已识别。
4. 在顶部 `PROMPT (feeds every stage)` 文本框中粘贴提示词。
5. 检查分辨率、时长、随机种子和采样步数。
6. 点击 `Queue` 开始生成。
7. 默认输出通常位于：

   ```text
   D:\ComfyUI\workspace\test1\ComfyUI\output
   ```

### 7.2 图生视频

图生视频使用：

```text
D:\ComfyUI\ComfyUI-Workflows\MinMax_H3\04_i2v_fl2v_4step_sla_lowvram.json
```

该工作流已经修改为使用本教程下载的主模型、两个 VAE 和 Heretic 文本编码器。

操作步骤：

1. 完全启动 `test1` 工作区的 ComfyUI。
2. 将 `04_i2v_fl2v_4step_sla_lowvram.json` 拖进 ComfyUI 页面。
3. 在标题为 `首帧图像（FL2V 可在 last_frame 添加第二个加载图像节点）` 的 `LoadImage` 节点中上传起始图片。
4. 在顶部 `PROMPT (feeds every stage)` 中描述图片接下来如何运动、镜头如何移动以及需要什么声音。
5. 在 `ResolutionSelector` 中选择输出宽高比和像素量。
6. 在 `PrimitiveFloat` 中设置目标秒数。
7. 检查 `BasicScheduler` 的采样步数为 4。
8. 点击 `Queue` 开始生成。

首次测试建议：

```text
输入图片比例：16:9
输出分辨率：1152 × 640
时长：4～6 秒
采样步数：4
帧率：24 FPS
```

输入图片最好提前裁切成与输出一致的宽高比。首帧会适配到工作流画布；如果输入图与输出比例差异较大，人物和物体可能被拉伸。

#### 首帧和尾帧控制

默认工作流只连接了一张首帧图片。如果需要从一张图片平滑过渡到另一张图片：

1. 再添加一个 `LoadImage` 节点并上传目标尾帧。
2. 将第二个 `LoadImage` 的图像输出连接到 `MiniMax H3 图像转视频` 节点的 `last_frame` 输入。
3. 第一张图片继续连接到 `first_frame`。
4. 在提示词中描述从首帧到尾帧之间的动作和镜头变化，不要描述互相冲突的场景。

#### 图生视频提示词示例

```text
integrated_multimodal_description:
保持输入图片中的成年女性外貌、发型、服装、身体比例、海边木栈道和晨光环境一致。生成一段约6秒的连续镜头。女性先安静地望向海面，随后海风逐渐吹动她的头发和外套衣角。她缓慢转过身看向镜头，自然微笑，然后抬起右手整理耳边被风吹乱的头发。镜头以非常缓慢、稳定的速度向前推进，背景海浪持续运动，远处一只海鸥从左向右飞过。动作自然连贯，人物身份和服装始终稳定，符合真实物理。不要镜头切换，不要突然缩放，不要人物复制，不要面部漂移，不要手指或肢体畸形，不要改变背景结构，不要文字、字幕或水印。

overall_soundscape:
自然的海浪声、轻柔海风声、远处海鸥鸣叫和衣料被风吹动的细微声音，空间感真实，声音变化与画面动作同步。

non_diegetic_music:
非常轻柔的钢琴与环境氛围音乐，音量较低，不遮盖海浪和环境声音，在结尾自然减弱。
```

图生视频提示词应重点描述“接下来发生什么”，同时明确要求保持输入图中的人物、服装和场景一致。动作不要一次安排过多，短视频通常用一个主要动作配合一个镜头运动更稳定。

## 8. 当前工作流默认参数

当前 `01_reference_4step_sla_lowvram.json` 的设置为：

```text
宽高比：16:9
目标像素量：0.7 MP
对齐倍数：32
实际分辨率：1152 × 640
时长输入：10.0 秒
实际帧数：243 帧
输出帧率：24 FPS
实际视频时长：约 10.125 秒
采样步数：4
调度器：simple
Seed：42
FFN chunks：4
```

### 8.1 修改时长

找到左侧的 `PrimitiveFloat` 节点：

```text
4.0  → 约 4 秒
6.0  → 约 6 秒
8.0  → 约 8 秒
10.0 → 约 10 秒
```

该值经过 `ComfyMathExpression` 自动转换为 MiniMax H3 所需的合法帧数 `17k+5`。

### 8.2 修改分辨率

找到 `ResolutionSelector` 节点。当前值：

```text
Aspect Ratio：16:9 (Widescreen)
Megapixels：0.7
Multiple：32
```

它会自动得到 `1152×640`。

不要修改 `MiniMaxH3ImageToVideo` 节点里显示的备用宽高，因为它的宽高输入已经被 `ResolutionSelector` 连线覆盖。

### 8.3 1920×1080 的处理

MiniMax H3 节点的宽高按 32 像素步进。`1920` 可以被 32 整除，但 `1080` 不可以：

```text
1920 ÷ 32 = 60
1080 ÷ 32 = 33.75
1088 ÷ 32 = 34
```

因此推荐原生生成：

```text
Aspect Ratio：16:9
Megapixels：2.0
Multiple：32
结果：1920 × 1088
```

如果最终必须交付标准 `1920×1080`，生成后从顶部和底部各裁掉 4 像素。

`1920×1088` 的像素量约为 `1152×640` 的 2.8 倍，10 秒视频可能显著增加显存和时间开销。低显存显卡建议先生成 `1152×640`，然后再做视频放大和裁切。

## 9. 测试提示词

下面的提示词可直接粘贴到工作流顶部的 `PROMPT` 节点：

```text
integrated_multimodal_description:
一段约10秒的写实电影感视频，清晨金色阳光下的海边木栈道，全程为一个连续镜头，不切镜。海面闪烁着自然的金色反光，天空清澈，远处有几艘小帆船缓慢移动。

0-2秒：镜头从贴近木栈道的低机位开始，一名穿浅蓝色运动外套、白色长裤的年轻成年女性踩着滑板，从画面左侧平稳进入。她的头发和外套衣角被海风轻轻吹动，滑板轮子在木板接缝处产生细微震动。

2-5秒：镜头向后平稳移动并保持跟拍。女性略微俯身加速，滑板沿栈道自然滑行。一只金毛犬从后方跑来，在她旁边开心地奔跑，毛发随动作和海风自然摆动。远处几只海鸥从海面起飞。

5-8秒：前方出现一个浅水洼，女性轻巧地控制滑板绕过水洼，金毛犬直接踩过水面，溅起清晰而自然的水花。镜头稍微向右侧环绕，保持人物和金毛犬处于画面中央，阳光在镜头中形成轻微而真实的光晕。

8-10秒：女性逐渐减速，在栈道栏杆旁停下，弯腰轻轻摸了摸金毛犬的头。她抬头望向海面，微笑着用中文说：“今天一定会是个好天气。”镜头缓慢升高，最后同时展示人物、金毛犬、海面和远处帆船。

人物、服装、滑板和金毛犬的外观在整个视频中保持一致。滑板运动、人物重心变化、狗的奔跑、水花、毛发和衣物运动符合真实物理。自然电影摄影，温暖晨光，真实肤色，细腻但不过度磨皮，轻微景深，流畅稳定的跟拍镜头。不要镜头切换，不要慢动作，不要人物或动物复制，不要面部漂移，不要肢体、手指或狗腿畸形，不要滑板变形，不要文字、字幕、水印或品牌标志。

overall_soundscape:
持续而柔和的海浪声与海风声，滑板轮子滚过木栈道的连续声音，经过木板缝隙时有轻微节奏变化，金毛犬奔跑时的脚步声和呼吸声，踩入水洼时清晰的水花声，远处海鸥鸣叫，女性最后一句中文台词清楚自然，声音与口型基本同步。所有声音具有真实的海边空间感和距离变化。

non_diegetic_music:
轻快温暖的原声吉他与柔和钢琴，节奏舒缓，音量保持较低，不遮盖海浪、滑板、动物和人物对白。在女性停下望向海面的最后两秒稍微增强，随后自然结束。
```

## 10. 常见问题

### 10.1 `curl: (28) Failed to connect to huggingface.co:443`

这是网络无法连接 Hugging Face，不是文件名错误。使用：

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
hf download ...
```

`hf download` 支持缓存、校验和断点续传，下载大模型通常比直接使用 `curl` 更稳。

### 10.2 模型文件大小不对

正确的大致大小：

```text
主模型：19.54 GiB（页面约标 21 GB）
视频 VAE：2.95 GiB（页面约标 3.17 GB）
音频 VAE：0.56 GiB
Heretic 文本编码器：14.61 GiB（页面约标 15.7 GB）
```

如果 `.safetensors` 只有几 KB 或几 MB，通常下载到的是错误页面、Git LFS 指针或未完成文件。重新执行对应的 `hf download`。

### 10.3 打开工作流仍提示缺少模型

依次确认：

1. 模型文件确实位于 `D:\ComfyUI\models` 的对应子目录。
2. `test1` 的 `extra_model_paths.yaml` 已存在。
3. 文件名与工作流选择完全一致。
4. 修改路径配置后已彻底重启 ComfyUI 后端，而不是只刷新浏览器。
5. 重新拖入磁盘上的已修改 JSON，避免浏览器仍使用旧工作流副本。

### 10.4 红色节点或提示缺少节点

这不是模型问题。在 ComfyUI Manager 中安装：

```text
ComfyUI-KJNodes
ComfyUI-MAINodes
ComfyUI-PlagueKind-Nodes-only-sparse
```

然后重启 ComfyUI。

### 10.5 显存不足或 `CUDA out of memory`

按以下顺序降低压力：

1. 使用 `lowvram` 工作流。
2. 将时长从 10 秒降到 4 秒。
3. 使用 `960×544` 或默认 `1152×640`。
4. 关闭其他占用显存的程序。
5. 保持采样步数为 4。
6. 先低分辨率生成，再单独放大视频。

### 10.6 下载后出现重复目录

`hf download` 会保留仓库内的相对路径。例如：

```powershell
hf download 仓库 "diffusion_models/模型文件" --local-dir "D:\ComfyUI\models"
```

最终会自动放入：

```text
D:\ComfyUI\models\diffusion_models\模型文件
```

如果把 `--local-dir` 直接写成 `D:\ComfyUI\models\diffusion_models`，可能得到重复的：

```text
D:\ComfyUI\models\diffusion_models\diffusion_models\模型文件
```

## 11. 参考链接

- 部署教程：<https://www.freedidi.com/25385.html>
- Fused Turbo 主模型：<https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot>
- 低显存工作流：<https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/tree/main/workflows/lowvram>
- Kijai MiniMax H3 实验模型：<https://huggingface.co/Kijai/MiniMax-H3-experimental>
- ComfyUI 官方打包模型：<https://huggingface.co/Comfy-Org/MiniMax-H3>
- Heretic 文本编码器：<https://huggingface.co/Momoking/Qwen3-VL-32B-Heretic-MiniMax-H3-NVFP4>
- ComfyUI-KJNodes：<https://github.com/kijai/ComfyUI-KJNodes>
- ComfyUI-MAINodes：<https://github.com/matlowai/ComfyUI-MAINodes>
- H3 SLA 稀疏注意力节点：<https://github.com/ethanfel/ComfyUI-PlagueKind-Nodes-only-sparse>

