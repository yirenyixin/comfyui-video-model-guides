# LTX 2.3 + PinkCherry 无审查 Q5 GGUF：ComfyUI Desktop 8GB 显存安装配置教程

> 适用环境：Windows、ComfyUI Desktop、NVIDIA RTX 3070 Laptop 8GB（其他 8GB NVIDIA 显卡也可参考）  
> 本机工作区：`D:\ComfyUI\workspace\<YOUR_WORKSPACE>`  
> ComfyUI 根目录：`D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI`  
> 教程版本日期：2026-10-03

> [!CAUTION]
> 本教程使用的是 **PinkCherry 无审查（NSFW / uncensored）模型**。模型可能生成成人或敏感内容，仅限成年人在合法、合规并获得必要同意的前提下使用。禁止用于未成年人、非自愿内容、真人深度伪造、骚扰、剥削或其他违法用途。下载和使用前请阅读并遵守模型许可证、Hugging Face 仓库规则以及所在地法律法规。

## 1. 安装 Hugging Face 命令行工具

在 PowerShell 中执行：

```powershell
py -m pip install --upgrade huggingface_hub
hf --help
```

建议登录 Hugging Face，以减少限速并访问需要确认协议或敏感内容确认的仓库：

```powershell
hf auth login
```

不要把 Hugging Face Token 发给别人，也不要粘贴到公开聊天或截图中。

如果仓库页面要求确认许可或敏感内容，请先在浏览器登录 Hugging Face，并在对应模型页面完成确认。

## 2. 建立模型仓库目录

本教程使用独立的 D 盘模型仓库：

```powershell
$modelRoot = "D:\ComfyUI\models"

New-Item -ItemType Directory -Force -Path "$modelRoot\diffusion_models"
New-Item -ItemType Directory -Force -Path "$modelRoot\loras\LTX-2.3"
New-Item -ItemType Directory -Force -Path "$modelRoot\vae"
New-Item -ItemType Directory -Force -Path "$modelRoot\text_encoders"
New-Item -ItemType Directory -Force -Path "$modelRoot\latent_upscale_models"
```

## 3. 下载模型和工作流

### 3.1 下载 PinkCherry 无审查 Q5 GGUF 主模型

**模型类型：无审查（NSFW / uncensored）视频生成模型。**

以下命令下载 PinkCherry v1.8 的 Q5 GGUF 无审查主模型：

```powershell
hf download SexGod1979/PinkCherry_NSFW_LTX23 `
  "v1.8/PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf" `
  --local-dir "D:\ComfyUI\models\diffusion_models\PinkCherry-LTX23-v1.8"
```

最终路径应为：

```text
D:\ComfyUI\models\diffusion_models\PinkCherry-LTX23-v1.8\v1.8\PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf
```

该仓库被 Hugging Face 标记为敏感内容。访问或下载前可能需要登录并确认敏感内容提示。请遵守模型许可证、平台规则以及所在地法律法规。

### 3.2 下载无审查模型配套工作流

**工作流类型：PinkCherry 无审查模型配套工作流。** 工作流本身只是节点配置，但其中可能包含成人向示例提示词。

```powershell
hf download SexGod1979/PinkCherry_NSFW_LTX23 `
  --include "workflows/*.json" `
  --local-dir "D:\ComfyUI\ComfyUI-Workflows\PinkCherry-LTX23-v1.8"
```

### 3.3 下载 LTX 2.3 Distilled LoRA

```powershell
hf download Lightricks/LTX-2.3 `
  "ltx-2.3-22b-distilled-lora-384-1.1.safetensors" `
  --local-dir "D:\ComfyUI\models\loras\LTX-2.3"
```

最终路径：

```text
D:\ComfyUI\models\loras\LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors
```

官方文件页面显示该 LoRA 约为 7.61GB。

### 3.4 下载三个 VAE

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

最终路径：

```text
D:\ComfyUI\models\vae\LTX23_video_vae_bf16.safetensors
D:\ComfyUI\models\vae\taeltx2_3.safetensors
D:\ComfyUI\models\vae\LTX23_audio_vae_bf16.safetensors
```

### 3.5 下载 LTX 2.3 文本投影模型

```powershell
hf download Kijai/LTX2.3_comfy `
  "text_encoders/ltx-2.3_text_projection_bf16.safetensors" `
  --local-dir "D:\ComfyUI\models"
```

最终路径：

```text
D:\ComfyUI\models\text_encoders\ltx-2.3_text_projection_bf16.safetensors
```

### 3.6 下载 Gemma 3 12B FP4 Mixed 文本编码器

8GB 显存不建议使用完整 BF16 Gemma。本配置使用约 9.45GB 的 FP4 Mixed 版本。

```powershell
curl.exe -L `
  "https://huggingface.co/Comfy-Org/ltx-2/resolve/main/split_files/text_encoders/gemma_3_12B_it_fp4_mixed.safetensors?download=true" `
  -o "D:\ComfyUI\models\text_encoders\gemma_3_12B_it_fp4_mixed.safetensors"
```

最终路径：

```text
D:\ComfyUI\models\text_encoders\gemma_3_12B_it_fp4_mixed.safetensors
```

### 3.7 下载 LTX 2.3 空间放大模型

```powershell
hf download Lightricks/LTX-2.3 `
  "ltx-2.3-spatial-upscaler-x2-1.1.safetensors" `
  --local-dir "D:\ComfyUI\models\latent_upscale_models"
```

最终路径：

```text
D:\ComfyUI\models\latent_upscale_models\ltx-2.3-spatial-upscaler-x2-1.1.safetensors
```

## 4. 最终目标

完成后，ComfyUI 可以通过同一份工作流运行：

- 文生视频（T2V）
- 图生视频（I2V）
- PinkCherry LTX 2.3 Q5 GGUF 主模型
- LTX 2.3 Distilled LoRA
- Gemma 3 12B FP4 文本编码器
- 视频和音频 VAE
- Tiny VAE 预览
- LTX 2.3 二次潜空间放大
- 适合 8GB 显存的低显存分块

已经修改好的工作流：

```text
D:\ComfyUI\ComfyUI-Workflows\PinkCherry-LTX23-v1.8\workflows\PinkCherry_LTX23_Q5_GGUF_8GB.json
```

该工作流已完成以下修改：

- 将普通 Checkpoint 加载器替换为 `Unet Loader (GGUF)`。
- 主模型替换为 `PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf`。
- 使用 LTX 2.3 Distilled LoRA。
- 使用 FP4 Gemma，避免完整 BF16 Gemma 带来的更高内存压力。
- 删除未连接的 `Fast Groups Bypasser (rgthree)` 节点。
- 将主要标题、说明和提示词中文化。
- 默认参数调整为 `832×480、3 秒、24 FPS`。
- 默认启用低显存分块。

### 4.1 如何手动把原始工作流修改成 Q5 GGUF 版本

如果直接使用本教程提供的 `PinkCherry_LTX23_Q5_GGUF_8GB.json`，以下修改已经完成，不需要再做一遍。本节用于说明如何从作者原始工作流手动修改。

修改前先打开作者原始工作流，然后使用“工作流 → 另存为”，保存成新文件：

```text
PinkCherry_LTX23_Q5_GGUF_8GB.json
```

不要直接覆盖作者原始工作流，方便修改失败时恢复。

#### 第一步：删除原来的 Checkpoint 加载器

1. 在画布的“模型”分组中找到原来的模型加载节点。
2. 节点类型通常是 `CheckpointLoaderSimple`，标题可能显示为“加载检查点”或 `Load Checkpoint`。
3. 记录该节点 `MODEL` 输出端口连接的紫色连线目标。
4. 选中原节点后按 `Delete` 删除。

原始节点通常会要求以下 BF16 文件：

```text
PinkCherry_FineTune_bf16_v1_8_LTX23.safetensors
```

8GB 显卡不使用这个完整 BF16 文件，改用 Q5 GGUF。

#### 第二步：添加 GGUF 主模型加载器

1. 在画布空白处双击，搜索 `Unet Loader (GGUF)`。
2. 添加由 `ComfyUI-GGUF` 提供的节点；内部节点类型应为 `UnetLoaderGGUF`。
3. 在 `unet_name` 下拉框中选择：

```text
PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf
```

4. 将该节点右侧的紫色 `MODEL` 输出连接到 `LoraLoaderModelOnly` 左侧的 `model` 输入。
5. 如果下拉框中没有这个模型，确认文件位于当前工作区的以下目录，然后彻底重启 ComfyUI：

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\models\diffusion_models\PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf
```

#### 第三步：设置 LTX 2.3 Distilled LoRA

找到 `LoraLoaderModelOnly` 节点，在 LoRA 下拉框中选择：

```text
LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors
```

连接顺序应为：

```text
Unet Loader (GGUF)
        │ MODEL
        ▼
LoraLoaderModelOnly
        │ MODEL
        ▼
后续模型处理和采样节点
```

保持作者工作流原有的 LoRA 强度。不要把 LoRA 连接到文本编码器端口；该文件在此工作流中只应用到模型。

#### 第四步：替换文本编码器

找到标题类似 `CLIPLoader (Gemma + LTX Embeddings)` 的 `DualCLIPLoader` 节点，依次设置：

```text
第一个模型：gemma_3_12B_it_fp4_mixed.safetensors
第二个模型：ltx-2.3_text_projection_bf16.safetensors
类型：ltxv
```

不要选择作者原工作流中的完整 Heretic/BF16 Gemma：

```text
gemma-3-12b-it-heretic-v2.safetensors
```

FP4 Mixed 版本更适合本机 8GB 显卡和 32GB 系统内存。修改模型选择即可，不要断开 `CLIP` 输出连线。

#### 第五步：设置三个 VAE

分别找到对应的 VAE 加载节点并选择：

| 用途 | 节点 | 选择的文件 |
|---|---|---|
| 视频编码与解码 | `VAELoader` | `LTX23_video_vae_bf16.safetensors` |
| 音频编码与解码 | `VAELoaderKJ` | `LTX23_audio_vae_bf16.safetensors` |
| 采样过程预览 | `VAELoader` | `taeltx2_3.safetensors` |

音频 VAE 的其他选项可先保持：

```text
device：main_device
weight_dtype：bf16
```

如果不需要音频，不要随意删除音频节点，因为作者工作流的输出链路可能仍引用它。应先完成一次正常生成，再单独制作无音频精简版。

#### 第六步：设置二次潜空间放大模型

找到 `LatentUpscaleModelLoader` 节点，选择：

```text
ltx-2.3-spatial-upscaler-x2-1.1.safetensors
```

保持它与 `LTXVLatentUpsampler` 的原有连接。这个阶段会增加运行时间，但能改善最终清晰度。

#### 第七步：启用低显存分块

找到：

```text
LTXV Chunk FeedForward
```

如果节点呈灰色或标题栏显示为旁路状态，选中节点后按 `Ctrl+B` 取消旁路，或右键节点选择“模式 → 始终运行”。

建议参数：

```text
chunks：2
dim_threshold：4096
```

启用后的节点模式应为正常运行，而不是 `Bypass/旁路`。分块会降低显存峰值，但可能稍微减慢生成速度。

#### 第八步：删除不必要的 rgthree 节点

如果加载工作流时提示缺少：

```text
Fast Groups Bypasser (rgthree)
```

可以在画布中找到标题为 `ENABLE PROMPT ENHANCER` 的该节点并删除。这个节点在原工作流中没有数据连线，只是用于批量切换分组，不参与模型生成。

删除后再次打开“工作流总览”，确认不再提示缺少 rgthree 节点。

#### 第九步：设置 8GB 稳定参数

在“视频设置”分组中修改：

```text
宽度：832
高度：480
时长：3
帧率：24
```

对应节点通常是：

| 显示标题 | 节点类型 | 值 |
|---|---|---:|
| 宽度 | `INTConstant` | 832 |
| 高度 | `INTConstant` | 480 |
| 时长（秒） | `INTConstant` | 3 |
| 帧率 | `PrimitiveFloat` | 24 |

不要使用原始工作流中的 `1280×1280、13 秒` 作为首次测试参数。该组合约为 313 帧，对 8GB 显卡负担过大。

#### 第十步：选择图生视频或文生视频

找到布尔开关：

```text
文生视频（不使用参考图）
```

- 设置为 `false`：图生视频；必须在“加载图像”节点选择参考图片。
- 设置为 `true`：文生视频；不会使用参考图片。

首次测试图生视频时建议设为 `false`，并选择教程提供的测试图片。

#### 第十一步：中文化标题和提示词（可选）

节点标题、分组标题和 Markdown 说明可以翻译成中文，因为它们只影响界面显示。

以下内容不要翻译或改名：

- 模型文件名。
- `model`、`vae`、`clip`、`fps` 等 Set/Get 节点内部变量名。
- 节点的内部类型名称。
- 采样器名称、调度器名称和公式。

修改这些技术字符串可能导致模型找不到、Set/Get 节点断开或工作流无法执行。

#### 第十二步：保存并重新加载

1. 使用“工作流 → 保存”保存新 JSON。
2. 关闭当前画布。
3. 重新打开保存后的 `PinkCherry_LTX23_Q5_GGUF_8GB.json`。
4. 打开“工作流总览”，确认没有缺失节点和缺失模型。
5. 如果模型仍显示缺失，完全退出 ComfyUI Desktop（包括系统托盘）后重新启动。

最终关键连接可以概括为：

```text
PinkCherry Q5 GGUF
        │
        ▼
LTX 2.3 Distilled LoRA
        │
        ▼
低显存分块
        │
        ▼
第一遍采样 → 潜空间 ×2 放大 → 第二遍采样
        │
        ▼
视频 VAE / 音频 VAE 解码 → 保存视频
```

## 5. 本机目录说明

### 5.1 ComfyUI Desktop 安装目录

```text
D:\Program Files\Comfy Desktop
```

### 5.2 当前有效工作区

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>
```

### 5.3 当前有效 ComfyUI 根目录

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI
```

注意：ComfyUI Desktop 可以创建多个工作区。模型放进旧工作区并不代表新工作区可以看到。判断实际工作区时，以日志中的 `ComfyUI Path` 为准。

日志文件通常位于：

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\user\comfyui.log
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\logs\comfyui.log
```

## 6. 安装必需的自定义节点

优先在 ComfyUI Manager 中搜索并安装：

- `ComfyUI-GGUF`
- `ComfyUI-KJNodes`
- `ComfyUI-VideoHelperSuite`

安装后必须完全退出并重新启动 ComfyUI Desktop。

也可以手动安装。PowerShell 示例：

```powershell
Set-Location "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\custom_nodes"

git clone https://github.com/city96/ComfyUI-GGUF.git
git clone https://github.com/kijai/ComfyUI-KJNodes.git comfyui-kjnodes
git clone https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite.git comfyui-videohelpersuite
```

安装 GGUF 依赖：

```powershell
& "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\standalone-env\python.exe" `
  -m pip install -r `
  "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\custom_nodes\ComfyUI-GGUF\requirements.txt"
```

如果 `standalone-env\python.exe` 不存在，请在日志中查找 `Python executable:`，使用日志显示的 Python 路径。

`ComfyUI-GGUF` 官方说明要求使用 `Unet Loader (GGUF)` 加载 GGUF 扩散模型，并将文件放入 `models/unet` 或 `models/diffusion_models`。

## 7. 让 test1 工作区识别 D 盘模型

ComfyUI Desktop 的不同工作区拥有各自的 `ComfyUI\models`。最稳定的方法是把模型直接放进去，或者创建硬链接。

硬链接不会复制模型内容，不会额外占用几十 GB，但源文件和目标文件必须位于同一个磁盘分区。本教程中的源和目标都位于 D 盘。

以管理员身份打开 PowerShell，然后执行：

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

如果再次执行时提示目标已存在，先检查现有文件是否正确，不要盲目删除。

## 8. 正确配置工作流节点

### 8.1 主模型

使用节点：

```text
Unet Loader (GGUF)
```

选择：

```text
PinkCherry_FineTune_Q5_K_M_v18_LTX23.gguf
```

将它的 `MODEL` 输出连接到原工作流中 LoRA 加载器的模型输入。

不要使用 `CheckpointLoaderSimple` 加载 GGUF 文件。

### 8.2 Distilled LoRA

使用：

```text
LoraLoaderModelOnly
```

选择：

```text
LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors
```

### 8.3 文本编码器

`DualCLIPLoader` 中选择：

```text
gemma_3_12B_it_fp4_mixed.safetensors
ltx-2.3_text_projection_bf16.safetensors
```

类型选择：

```text
ltxv
```

### 8.4 VAE

视频 VAE：

```text
LTX23_video_vae_bf16.safetensors
```

音频 VAE：

```text
LTX23_audio_vae_bf16.safetensors
```

采样预览 Tiny VAE：

```text
taeltx2_3.safetensors
```

### 8.5 空间放大模型

```text
ltx-2.3-spatial-upscaler-x2-1.1.safetensors
```

### 8.6 低显存分块

启用：

```text
LTXV Chunk FeedForward（低显存）
```

建议初始值：

```text
chunks = 2
dim_threshold = 4096
```

分块可以降低峰值显存，但可能略微降低速度。8GB 显卡建议优先保证稳定性。

## 9. 文生视频与图生视频切换

工作流中的开关：

```text
文生视频（不使用参考图）
```

- `true`：文生视频，不读取参考图片。
- `false`：图生视频，需要在“加载图像”节点中选择图片。

### 9.1 文生视频提示词

文生视频提示词需要完整说明：

- 人物或主体
- 环境
- 光线和风格
- 动作顺序
- 镜头运动
- 环境声、对白或音乐

示例：

```text
夜晚的城市街道被细雨打湿，一名穿红色雨衣的成年女子撑着透明雨伞站在温暖的面馆灯光旁。她缓慢转头看向远处，湿润的头发和衣摆被微风吹动。雨水落在伞面和路面上，蓝色与橙色霓虹倒影随着积水轻微晃动，街边蒸汽缓慢升起。镜头平稳地向前推进，可以听见雨声、远处车辆驶过积水的声音和面馆内轻微的餐具声。
```

### 9.2 图生视频提示词

图生视频时不要重复描述图片已经清楚呈现的外观，重点写：

- 主体如何移动
- 衣物、头发、水、烟雾等如何变化
- 镜头是否移动
- 声音如何随动作发生

示例：

```text
女子轻轻转头看向街道远处，眨了一下眼睛，湿润的头发被微风轻轻吹动。她稍微调整手中的透明雨伞，红色雨衣的衣摆随风摆动。雨水持续落在伞面和街道上，地面的霓虹倒影随着水波轻微晃动，右侧白色蒸汽缓慢升起。背景行人自然走动，镜头缓慢向前推进。可以听见细密雨声、车辆经过积水的声音以及面馆中轻微的餐具碰撞声。
```

## 10. 8GB 显存参数建议

当前稳定版工作流默认值：

```text
宽度：832
高度：480
时长：3 秒
帧率：24 FPS
模式：图生视频
低显存分块：启用
```

建议逐级测试：

| 用途 | 分辨率 | 时长 | FPS | 说明 |
|---|---:|---:|---:|---|
| 快速排错 | 640×384 | 3 秒 | 24 | 最容易成功 |
| 稳定测试 | 832×480 | 3 秒 | 24 | 8GB 默认档 |
| 均衡效果 | 832×480 | 5 秒 | 24 | 明显更慢 |
| 更清晰 | 960×544 | 3–5 秒 | 24 | 可能需要更强卸载 |
| 不建议 | 1280×720 以上 | 10 秒以上 | 24+ | 8GB 很容易失败或极慢 |

24 FPS 下，工作流会将帧数调整为 `8n + 1`：

- 3 秒约 73 帧。
- 5 秒约 121 帧。
- 10 秒约 241 帧。

时长翻倍时，生成时间通常接近翻倍；宽度和高度同时增加时，计算量增长得更快。

## 11. 打开本地工作流

推荐文件：

```text
D:\ComfyUI\ComfyUI-Workflows\PinkCherry-LTX23-v1.8\workflows\PinkCherry_LTX23_Q5_GGUF_8GB.json
```

打开方法：

1. 启动 ComfyUI Desktop，并确认工作区是 `test1`。
2. 把 JSON 文件直接拖入 ComfyUI 画布；或者使用“工作流 → 打开”。
3. 如果画布中已经打开旧版本，关闭时不要保存。
4. 重新打开上面的 JSON。
5. 点击“工作流总览”中的刷新按钮。

工作流也可复制到应用内目录：

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\user\default\workflows\
```

## 12. 测试图像

本次生成的测试图片已放入 ComfyUI Desktop 公共输入目录：

```text
C:\Users\<YOUR_WINDOWS_USER>\AppData\Local\Comfy-Desktop\ComfyUI-Shared\input\ltx23_rainy_city_test.png
```

图生视频测试时：

1. 将“文生视频（不使用参考图）”设为 `false`。
2. 在“加载图像”中选择 `ltx23_rainy_city_test.png`。
3. 使用 `832×480、3 秒、24 FPS`。
4. 先测试固定镜头或轻微动作，再增加镜头运动。

## 13. 硬件和磁盘建议

### 13.1 8GB 显存的现实预期

LTX 2.3 是 22B 级别的视频模型。Q5 GGUF 可以降低模型权重占用，但 8GB 显存仍需要依赖内存与显存之间的动态卸载，因此：

- 第一次加载模型通常很慢。
- 生成视频明显比生成图片慢。
- 过高分辨率或过长视频容易导致显存不足。
- 运行时系统内存占用可能很高。
- 建议至少 32GB 系统内存，并保证系统盘有足够虚拟内存空间。

### 13.2 磁盘空间

主要文件大致包括：

| 文件 | 大小（约） |
|---|---:|
| PinkCherry Q5 GGUF | 15.9 GB |
| LTX 2.3 Distilled LoRA | 7.61 GB |
| Gemma 3 12B FP4 Mixed | 9.45 GB |
| 文本投影模型 | 2.31 GB |
| 视频 VAE | 1.45 GB |
| 音频 VAE | 365 MB |
| Tiny VAE | 23.5 MB |
| 空间放大模型 | 996 MB |

建议至少预留 50GB，最好预留 70GB 以上，以容纳缓存、输出视频和后续模型。

## 14. 常见问题排查

### 14.1 明明下载了模型，仍提示“缺失模型”

常见原因：模型放在了另一个 ComfyUI 工作区。

检查日志：

```powershell
Select-String `
  -Path "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\user\comfyui.log" `
  -Pattern "ComfyUI Path|Python executable|Adding extra search path"
```

确认模型最终出现在：

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\models\对应分类
```

模型目录变化后，必须彻底退出 ComfyUI Desktop，包括右下角托盘，然后重新启动。

### 14.2 缺少 `UnetLoaderGGUF`

原因：未安装或未成功加载 `ComfyUI-GGUF`。

检查目录：

```text
D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\custom_nodes\ComfyUI-GGUF
```

安装依赖并重启。如果节点存在，应能在节点菜单的 GGUF/bootleg 分类中找到 `Unet Loader (GGUF)`。

### 14.3 提示 `Fast Groups Bypasser (rgthree)` 缺失

该节点只是工作流分组的快捷开关，不参与生成链路。本教程提供的 Q5 工作流已经将其移除。

如果界面仍显示：

1. 不要保存当前旧画布。
2. 完全退出 ComfyUI Desktop。
3. 重新打开修改后的 Q5 JSON。

### 14.4 `No module named 'triton'`

如果日志显示：

```text
PatchTritonVAE requires triton
ModuleNotFoundError: No module named 'triton'
```

这是 KJNodes 的可选 Triton VAE 节点加载警告。只要工作流没有使用 `PatchTritonVAE`，通常可以忽略，不是 LTX 生成失败的核心原因。

### 14.5 CUDA Out of Memory / 显存不足

依次处理：

1. 改成 `640×384、3 秒、24 FPS`。
2. 确认启用 `LTXV Chunk FeedForward`。
3. 关闭其他占用显存的软件、浏览器硬件加速页面和游戏。
4. 不要使用 48/50/60 FPS。
5. 不要一开始就使用 10 秒以上视频。
6. 确认使用 Q5 GGUF 主模型和 FP4 Gemma。
7. 完全重启 ComfyUI，释放此前模型占用。

### 14.6 生成速度非常慢

这在 8GB 显存运行 22B 模型时是正常现象，尤其首次加载需要把模型从磁盘读入内存并在 CPU、内存和显存之间卸载。

重点检查：

- 是否误设为 `1280×1280`。
- 是否误设为 10～20 秒。
- 是否使用 48/50 FPS。
- 是否同时开启了不必要的高分辨率二次处理。
- 模型是否存放在速度较慢的机械硬盘。
- Windows 虚拟内存是否不足。

### 14.7 工作流文件已经修改，但界面还是旧内容

ComfyUI 前端会把当前画布保存在内存中。磁盘上的 JSON 修改后，已经打开的画布不会自动刷新。

正确操作：

1. 不要保存旧画布。
2. 关闭当前工作流。
3. 重新打开磁盘上的 JSON。
4. 必要时完全重启 ComfyUI Desktop。

### 14.8 Hugging Face 下载很慢或提示未认证

执行：

```powershell
hf auth login
```

也可以设置环境变量，但不要把 Token 写入公开脚本：

```powershell
$env:HF_TOKEN = "你的本地 Token"
```

该环境变量只对当前 PowerShell 会话有效。

## 15. 验证模型文件

列出关键模型：

```powershell
$root = "D:\ComfyUI\workspace\<YOUR_WORKSPACE>\ComfyUI\models"

Get-ChildItem -LiteralPath $root -Recurse -File |
  Where-Object {
    $_.Name -match "PinkCherry|ltx-2.3|LTX23|taeltx|gemma_3_12B"
  } |
  Select-Object FullName, Length
```

计算文件 SHA256：

```powershell
Get-FileHash `
  -LiteralPath "D:\ComfyUI\models\loras\LTX-2.3\ltx-2.3-22b-distilled-lora-384-1.1.safetensors" `
  -Algorithm SHA256
```

Lightricks 官方 LoRA 页面公布的 SHA256 为：

```text
f5d4953f3386197a4b4f5abdb17616ff256171e8075c111d6e7d2dfa6e823b3a
```

## 16. 参考来源

- LTX 2.3 官方模型仓库：<https://huggingface.co/Lightricks/LTX-2.3>
- LTX 2.3 Distilled LoRA：<https://huggingface.co/Lightricks/LTX-2.3/blob/main/ltx-2.3-22b-distilled-lora-384-1.1.safetensors>
- Kijai ComfyUI VAE：<https://huggingface.co/Kijai/LTX2.3_comfy/tree/main/vae>
- Kijai 文本投影模型：<https://huggingface.co/Kijai/LTX2.3_comfy/tree/main/text_encoders>
- Comfy-Org Gemma FP4：<https://huggingface.co/Comfy-Org/ltx-2/blob/main/split_files/text_encoders/gemma_3_12B_it_fp4_mixed.safetensors>
- ComfyUI-GGUF：<https://github.com/city96/ComfyUI-GGUF>
- PinkCherry LTX 2.3：<https://huggingface.co/SexGod1979/PinkCherry_NSFW_LTX23>

## 17. 最终检查清单

- [ ] ComfyUI Desktop 当前工作区为 `D:\ComfyUI\workspace\<YOUR_WORKSPACE>`。
- [ ] `ComfyUI-GGUF`、`ComfyUI-KJNodes`、`ComfyUI-VideoHelperSuite` 已安装。
- [ ] Q5 GGUF 主模型可在 `Unet Loader (GGUF)` 中选择。
- [ ] Distilled LoRA 可被 `LoraLoaderModelOnly` 识别。
- [ ] FP4 Gemma 与 LTX 文本投影均可被 `DualCLIPLoader` 识别。
- [ ] 视频、音频和 Tiny VAE 均可选择。
- [ ] 空间放大模型可选择。
- [ ] 工作流不再提示 `Fast Groups Bypasser (rgthree)`。
- [ ] 首次测试使用 `832×480、3 秒、24 FPS`。
- [ ] 图生视频时开关为 `false`，文生视频时开关为 `true`。
- [ ] 修改模型目录或插件后已彻底重启 ComfyUI Desktop。



