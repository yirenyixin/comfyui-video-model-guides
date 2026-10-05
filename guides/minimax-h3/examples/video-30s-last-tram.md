# MiniMax H3 30 秒连续测试案例：雨夜末班电车

这套案例按 **3 段 × 约 10.1 秒** 生成。第 1 段使用文生视频优化工作流，第 2、3 段使用图生视频优化工作流，并把上一段自动保存的最后一帧作为下一段首帧。

使用工作流：

- 第 1 段：[`01_T2V_10s_低显存_续片优化.json`](../workflows/01_T2V_10s_低显存_续片优化.json)
- 第 2、3 段：[`04_I2V_10s_低显存_续片优化.json`](../workflows/04_I2V_10s_低显存_续片优化.json)

## 固定元素

- 主角：成年女性快递员，短黑色波波头，黄色雨衣，红色帆布斜挎包。
- 道具：银色旧自行车，车把挂着一盏暖白小灯。
- 风格：写实电影感，35mm 镜头，雨夜蓝紫霓虹，湿地反光，人物服装和面部保持一致。
- 声音策略：生成阶段只要环境声、脚步、车铃和一句短对白；`non_diegetic_music` 始终设为 `none`。三段拼好后再叠加同一条 30 秒音乐，音乐最稳、不会在分段处重启。
- 固定参数：1152×640，24 fps，4 steps，seed 42。

## 时间控制写法

每段按实际约 **10.1 秒** 编排。提示词使用明确的时间区间，并规定最后约 0.9 秒必须到达且保持一个“续片锚点”。不要在一个时间区间里塞入多个复杂动作，也不要让后面的动作提前发生。模型仍可能产生约 0.5–1 秒偏差，但这种写法通常比单纯使用 `then / after that` 更稳定。

时间段描述是生成目标，不是逐帧硬约束。第一次测试应固定 seed 和所有采样参数；只有当动作持续偏早或偏晚时，才调整对应时间段的文字。

## 第 1 段：追上末班车（文生视频）

```text
integrated_multimodal_description:
A single continuous 10.1-second cinematic shot with no cuts. The timing below is mandatory; do not perform later actions early and do not invent additional actions. A realistic rainy night in a narrow neon-lit old city street. The same adult female courier has a short black bob haircut, a bright yellow raincoat, a red canvas crossbody satchel, and pushes a worn silver bicycle with a small warm-white lamp hanging from the handlebar. Blue and magenta signs reflect across wet pavement. Natural body motion, stable face, coherent hands, realistic fabric and rain, 35mm lens, shallow depth of field.

0.0-2.5 seconds: Establish a medium tracking shot from her left side. She walks quickly while pushing the bicycle. The last streetcar is visible ahead but she has not reacted yet.

2.5-6.5 seconds: One tram bell rings. She looks toward the streetcar and accelerates into a controlled run while continuing to guide the bicycle. The camera gently moves behind her. She must not reach the door yet.

6.5-9.2 seconds: The streetcar slows at the intersection. She reaches the doorway, stops the bicycle beside her, and raises only her right hand toward the exterior door handle.

9.2-10.1 seconds — mandatory final anchor: Her right hand touches the tram door handle, her left hand holds the bicycle, and the doorway fills the right side of the frame. Hold this composition steadily until the clip ends. Do not open the door in this segment.

overall_soundscape:
0.0-2.5 seconds: Continuous rainfall, distant wet-street traffic, soft bicycle-chain clicks and walking footsteps. 2.5-6.5 seconds: exactly one clear tram bell, then faster splashing footsteps and heavier breathing; at about 4.5 seconds she says softly in Mandarin, “等等我。” 6.5-10.1 seconds: footsteps slow and stop while rainfall and the idling tram remain continuous. Do not add a second bell or any other dialogue.

non_diegetic_music:
none
```

生成结束后，找到自动保存的 `t2v_last_frame_*.png`。把它上传到第 2 段图生视频工作流的 `Load Image` 节点。

## 第 2 段：车厢里的包裹（图生视频）

```text
integrated_multimodal_description:
A single continuous 10.1-second shot with no cuts. Continue exactly from the provided first frame. Preserve the exact same adult female courier: short black bob haircut, yellow raincoat, red canvas crossbody satchel, silver bicycle, matching face and clothing. Preserve the door, camera direction, rain, and blue-magenta neon continuity. The timing below is mandatory; do not perform later actions early.

0.0-2.0 seconds: Starting with her right hand already touching the handle, she pulls the tram door open. Her left hand keeps the bicycle stationary beside her. Do not enter before the door is fully open.

2.0-5.5 seconds: She guides the bicycle into the warm amber-lit vintage streetcar. The camera follows smoothly through the doorway. She takes only a few controlled steps and then stops beside the first row of wooden seats.

5.5-7.5 seconds: She steadies the bicycle with her left hand, turns her head, and notices a small paper parcel tied with red string on the nearest empty seat. She does not touch it yet.

7.5-9.2 seconds: The camera slowly moves into a close medium shot as she extends only her right hand toward the parcel.

9.2-10.1 seconds — mandatory final anchor: Her right hand hovers a few centimeters above the red-string parcel; the parcel remains on the seat, and her surprised face is clearly visible behind it. Hold this exact composition until the clip ends. Do not pick up the parcel in this segment.

overall_soundscape:
0.0-2.0 seconds: Continue the same rain outside and use one mechanical door-opening sound. 2.0-5.5 seconds: rain becomes naturally muffled as the camera enters; add damp footsteps, rolling bicycle tires and a gentle bicycle-bell rattle. 5.5-10.1 seconds: footsteps stop; maintain a soft tram electric hum and steady wheel rhythm. No door-closing sound and no dialogue in this segment.

non_diegetic_music:
none
```

生成结束后，选择自动保存的 `i2v_last_frame_*.png`，上传给第 3 段同一个图生视频工作流。

## 第 3 段：黎明的收件人（图生视频）

```text
integrated_multimodal_description:
A single continuous 10.1-second cinematic shot with no cuts. Continue exactly from the provided first frame. Preserve the exact same courier, yellow raincoat, short black bob haircut, red canvas satchel, silver bicycle, tram interior, red-string parcel, and lighting continuity. The timing below is mandatory; do not perform later actions early.

0.0-2.0 seconds: Her hovering right hand gently picks up the red-string parcel from the seat while the tram slows. She looks toward the closed doorway. The bicycle remains beside her.

2.0-4.0 seconds: The tram stops and the door opens once. Holding the parcel in her right hand and the bicycle in her left, she turns toward the doorway. She does not step down before the door is fully open.

4.0-6.5 seconds: She takes several controlled steps down onto the misty riverside platform. A small child in a navy hooded coat waits under the nearby canopy. She stops in front of the child and parks the bicycle beside her.

6.5-8.4 seconds: She slowly kneels while holding the parcel. Do not hand it over yet. The camera begins a very small semicircle, revealing the river and pale blue dawn.

8.4-9.4 seconds: She extends the parcel toward the child with both hands. The child accepts it and smiles.

9.4-10.1 seconds — mandatory final anchor: Hold on the courier kneeling beside the glowing bicycle lamp while the child holds the parcel. Keep both faces visible and stable until the final frame. The tram remains stopped in the soft background mist.

overall_soundscape:
0.0-2.0 seconds: Continue the steady tram hum and muffled rain. 2.0-4.0 seconds: the tram stops and the door produces one mechanical opening sound. 4.0-6.5 seconds: add steps descending onto the wet platform, riverside wind and very light rain. 6.5-8.4 seconds: footsteps stop and a few distant early birds enter. At about 8.5 seconds the child says quietly in Mandarin, “你真的送到了。” At about 9.1 seconds the courier answers, “答应你的。” 9.4-10.1 seconds: retain wind, light rain, birds and the idling tram. No additional dialogue.

non_diegetic_music:
none
```

## 验收要点

- 第 1 段最后一帧必须是右手触碰车门、左手扶自行车，车门尚未打开。
- 第 2 段开头应接续该姿势，结尾必须是手悬停在包裹上方，包裹仍在座位上。
- 第 3 段开头应从悬停的手开始拿起包裹，不能凭空出现在站台。
- 三段中人物发型、黄色雨衣、红色斜挎包和银色自行车不能改变。
- 如果某一段没有到达规定结束构图，应单独重做该段，不要依赖下一段自动修复。

## 拼接与连续音乐

推荐先用剪映专业版、Premiere 或 DaVinci Resolve：

1. 创建 1152×640、24 fps 时间线。
2. 依次放入三段视频。
3. 如果接缝出现短暂停顿，删除第 2、3 段开头重复的第一帧；24 fps 下约为 0.0417 秒。
4. 画面优先硬切；确有跳动时只使用 2–4 帧短叠化。
5. 对模型原声音频做 0.15–0.3 秒交叉淡化。
6. 添加一条覆盖完整时间线的背景音乐，不要给每段分别添加音乐。

以下 FFmpeg 方法适合三个片段编码参数完全一致、并且不需要手动删除重复帧的情况。

先把三段放在同一目录，例如：

```text
segment01.mp4
segment02.mp4
segment03.mp4
music.wav
```

在该目录创建 `list.txt`：

```text
file 'segment01.mp4'
file 'segment02.mp4'
file 'segment03.mp4'
```

先无损拼接三段画面和模型声音：

```cmd
ffmpeg -f concat -safe 0 -i list.txt -c copy merged_model_audio.mp4
```

再把一条连续音乐以较低音量混入整片。`-shortest` 会自动按视频长度结束：

```cmd
ffmpeg -i merged_model_audio.mp4 -stream_loop -1 -i music.wav -filter_complex "[0:a]volume=1.0[a0];[1:a]volume=0.16[bg];[a0][bg]amix=inputs=2:duration=first:dropout_transition=2[a]" -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k -shortest MiniMax_H3_30s_final.mp4
```

如果三个 MP4 的编码参数不一致，第一条 `-c copy` 可能失败。改用统一重编码：

```cmd
ffmpeg -f concat -safe 0 -i list.txt -c:v libx264 -preset medium -crf 18 -c:a aac -b:a 192k merged_model_audio.mp4
```

## 导出建议

```text
编码：H.264
帧率：24 fps
音频：AAC，48 kHz，192–320 kbps
中间成片：保持 1152×640
最终交付：全部剪辑完成后再统一放大到 1920×1080
```
