# MiniMax H3 文生图测试案例：沙漠观测站

这个案例用于测试人物面部、双手、服装材质、复杂光照和远近景层次。生成出来的图片也可以直接作为图生视频首帧。

使用工作流：[`02_T2I_ImageStudio_本地模型适配.json`](../workflows/02_T2I_ImageStudio_本地模型适配.json)

前置条件：已经安装自定义节点管理器中的 **MiniMax H3 Image Studio（astropuzzo）**，并在安装后重启 ComfyUI、刷新浏览器。

## 推荐设置

- 画幅：`16:9 landscape`
- 分辨率档位：`native detail | 0.98 MP`
- 尺寸：`1344 × 768`
- 帧数：`recommended | 5 frames`
- Seed：`42`，先保持固定，方便比较设置变化
- 当前适配工作流：`res_multistep / simple / 4 steps / shift 12,3`

如果 4 步结果细节不足，可以先尝试 8 步，再尝试 12 步。其他参数不要同时改变，这样容易判断差异来自哪里。

## 测试提示词

把下面整段复制到 `H3TextToImagePrepare` 的提示词输入框：

```text
A finished cinematic production still, 16:9 landscape composition. An adult East Asian female desert observatory engineer stands on an elevated metal platform at blue hour, three-quarter body portrait, facing slightly toward the camera. She has a consistent natural face, short wind-swept black hair, calm focused eyes, and realistic skin texture. She wears a sand-colored technical jacket with dark navy utility panels, a narrow red scarf, fitted work gloves, and a compact radio clipped to her chest. Her left hand naturally holds the railing and her right hand holds a small silver astronomical calibration device; both hands are clearly visible with correct anatomy.

Behind her is a large white radio telescope dish angled toward a sky filled with the first visible stars. Warm amber work lights illuminate the platform while cool cobalt twilight surrounds the distant desert mesas. Fine windblown dust catches the side light. Include realistic brushed metal, painted steel, fabric seams, subtle scratches, bolts, cables, and atmospheric depth. Use a 35mm cinema lens, eye-level camera, restrained teal-and-amber color contrast, soft volumetric light, natural depth of field, crisp subject focus, physically plausible lighting, high detail without over-sharpening.

Keep the scene grounded and photorealistic. One person only. No duplicated limbs, no extra fingers, no extra tools, no floating objects, no text, no logos, no watermark, no frame border. Leave visible open space on the right side of the platform so the image can later be animated into a slow camera move.
```

## 判断生成是否成功

优先挑选满足以下条件的一张：

1. 面部自然，没有明显左右不对称或五官漂移。
2. 左手确实扶着栏杆，右手确实拿着银色仪器。
3. 只有一个人物，没有重复手臂、工具或望远镜结构。
4. 暖色平台灯和蓝色天空形成明确但自然的冷暖关系。
5. 右侧保留了可供镜头运动使用的空间。

如果双手明显错误，优先更换 seed；如果所有候选图普遍缺少细节，再将步数从 4 提高到 8。不要同时修改 seed、步数、分辨率和提示词。

## 可直接用于图生视频的动作提示词

如果静态图效果满意，可以把它上传到图生视频工作流，并使用下面的动作描述：

```text
A single continuous cinematic shot. Preserve the exact same adult female engineer, face, short black hair, sand-colored technical jacket, navy utility panels, red scarf, gloves, radio, silver calibration device, platform, telescope and blue-hour lighting from the input image. She slowly looks from the calibration device toward the radio telescope while the telescope dish rotates a few degrees toward the starry sky. A light desert wind moves only her scarf and short hair. Fine dust drifts through the warm platform lights. The camera performs a very slow push-in with no cuts, no sudden motion, no new people and no changes to clothing or equipment. Natural restrained motion, stable identity, coherent hands, photorealistic cinematic lighting.

overall_soundscape:
Soft desert wind, a low observatory machinery hum, two quiet metallic relay clicks, and a brief radio static pulse. No dialogue.

non_diegetic_music:
none
```

图生视频时建议使用 [`04_I2V_10s_低显存_续片优化.json`](../workflows/04_I2V_10s_低显存_续片优化.json)，时长先设置为 4–6 秒验证人物稳定性，再尝试约 10 秒。
