---
title: "DriverGaze360: OmniDirectional Driver Attention with Object-Level Guidance"
authors: "Shreedhar Govil, Didier Stricker, Jason Rambach"
venue: "CVPR 2026, pp. 39786–39795"
year: 2026
reading_date: 2026-10-27
status: skimmed
tags:
  - motion-understanding
  - driver-attention
  - 360-vision
  - gaze
  - interpretable-ai
  - multimodal-learning
---

# DriverGaze360: OmniDirectional Driver Attention with Object-Level Guidance

## 基本信息

- **作者：** Shreedhar Govil, Didier Stricker, Jason Rambach
- **会议/期刊：** CVPR 2026，pp. 39786–39795（正式会议论文）
- **年份：** 2026；arXiv 首投 2025-12-16，v3 于 2026-03-05 更新，CVPR 2026 于 2026 年 6 月发表
- **阅读日期：** 2026-10-27
- **阅读状态：** `skimmed`
- **标签：** `driver-attention`、`360-vision`、`gaze`、`multimodal-learning`
- **论文：** https://openaccess.thecvf.com/content/CVPR2026/html/Govil_DriverGaze360_OmniDirectional_Driver_Attention_with_Object-Level_Guidance_CVPR_2026_paper.html
- **arXiv：** https://arxiv.org/abs/2512.14266
- **DOI：** https://doi.org/10.48550/arXiv.2512.14266
- **代码：** https://github.com/dfki-av/drivergaze360
- **数据集：** https://huggingface.co/datasets/dfki-av/drivergaze360 （公开仓库，但需接受使用条件；论文 19 人，发布数据剔除了 1 人）
- **项目主页：** https://dfki-av.github.io/drivergaze360/
- **主要价值类型：** Dataset / Baseline / Method Module / Related Work
- **阅读优先级：** A

## 一句话总结

提出覆盖司机周围 360° 的 gaze-annotated 模拟驾驶基准，并通过 attention map + attended-object semantic segmentation 联合学习，提高宽视场驾驶注意力预测及其对象层面的可解释性。

## 研究问题与动机

过去驾驶注意力数据多为窄前视角，难以记录变道、转弯或观察后视镜时的侧后向视线分配。本文通过可控 CARLA 场景和真人佩戴眼动仪驾驶，收集全周视域的注意力分布，再评估交通目标语义是否能帮助稀疏注视点预测。

## 核心方法

- **采集：** CARLA、SCENIC；三块前方大屏及两块后视镜显示；Pupil Core 眼动仪（120 Hz）、30 Hz 模拟环境视频。利用 AprilTag/homography 将真实注视点映射到模拟空间，30 帧时间窗内高斯核融合形成 saliency target。
- **输入：** 五路水平拼接 rectilinear RGB（**并非原生 ERP**），每段 16 帧，resize 后 `16 × 3 × 224 × 1120`。
- **网络：** Video Swin Transformer backbone + 共享时空特征 + gaze saliency decoder + attended-object segmentation decoder。只给被注视过的车辆、行人、骑行者、路牌和信号灯保留语义标签。
- **输出：** 全周 gaze attention map 与 attended-object 语义像素图；目标函数合并 KLD/CC、Dice/IoU 与 cross-entropy。
- **代码：** 官方仓库提供预处理、训练和推理脚本及公开 checkpoint 的入口；数据下载需先同意使用条款。

## 数据集与评价指标

- 论文统计为约 **9 小时、100 万 gaze-labeled frames、19 位参与者**（最初招募 21 人，2 人因晕动或操作困难排除）；五路 1280×720 RGB，30 FPS；自由导航、目标导航、风险事件三个场景类别。
- 训练/验证以 **CARLA Town** 划分，训练 Towns 2/3/4/7/10/11，验证 Towns 1/5/6；并额外使用真实 DADA-2000 的窄视场视频做迁移评估。
- **Saliency 指标：** KLD（低优）、SIM/CC/NSS（高优）；**对象分割：** Dice、IoU。
- **发布口径变化：** 官方 Hugging Face 数据卡写明发布前又剔除一名参与者，实际发布版本为 18 人（帧数仍约 100 万）。引用或复现必须区分论文 19 人和公开 18 人。

## 主要结果

- **DriverGaze360（Table 4）：** 模型 KLD **1.067**、SIM **0.515**、CC **0.667**、NSS **6.309**；FBLNet 为 **1.215 / 0.494 / 0.639 / 6.012**。
- **Attended-object 消融（Table 6）：** attention-only KLD **1.127** / CC **0.654**；加入全对象 segmentation 为 **1.092** / **0.659**；仅保留 attended object 后达到 **1.067** / **0.667**，NSS 从 **6.158** 升至 **6.309**。
- **真实窄视场 DADA-2000（Table 5）：** SIM **0.396**、CC **0.506**、NSS **3.478**，好于 FBLNet **0.369 / 0.480 / 3.305**；KLD **1.654** 略差于 DADANet 的 **1.646**。

## 优点

- 补足侧后方环境的 driver attention 标签，带 object segmentation、RGB、depth 与同步驾驶状态。
- 不只回归模糊的 attention heatmap，而是明确监督“看到了什么对象”；消融可分辨对象语义的收益。
- 公开训练脚本、模型和数据访问入口，便于复现实验。

## 局限

- 主要是 **模拟驾驶**，与真实道路的光学、行为和风险动态存在 sim-to-real gap。
- 所谓 360° 使用**五个矩形/透视投影拼接**而非单镜头 fisheye/ERP；不能直接推断球面畸变鲁棒性。
- DADA-2000 外部实验是窄 FoV，尚不等于真实世界 360° 多方向驾驶 gaze 验证。
- 数据公开版本比论文少一名参与者；训练/验证主要按 Town 划分，不能代替严格的独立 driver-disjoint 外部测试。
- 预测的环境物体注视不等于实际的驾驶员头部代偿或视野缺损状况，需另有相应 ground truth。

## 个人评价

**推断：**本文对驾驶员视觉行为研究更有价值的不是把“360°”当成模型卖点，而是 **语义对象引导 + 环境/眼动坐标配准 + 多方向注意力覆盖** 三种设计。未来应分开评价 gaze targeting、head scanning 与外部道路风险，使 attention map 的提升不被误解为驾驶安全性能提升。

## 与我的研究关联

**推断：**可用于驾驶员模拟视野缺损（VFL）研究的环境端对照：将现有三视角 FaceMesh/head motion（扫描幅度、速度、频次）与交通对象方位和驾驶任务事件对齐，比较不同 VFL、昼夜和交通负荷条件下的 attentional coverage。实验应明确区分“头转动”与“眼注视”，若没有 eye-tracker，不能把 FaceMesh 的 head pose 当成 gaze GT。可参考其 attended-object loss 构造可解释辅助任务，并以 driver-disjoint、town-disjoint、时间段隔离及真实驾驶视频做泛化检验。

## 后续阅读

- 优先阅读 **Sec. 3.3–3.5、Sec. 4、Sec. 5.2–5.3、Tables 4–6、Sec. 11–12**。
- 复现 Attention-only → +all-object segmentation → +attended-object segmentation，对比不同 gaze window 和场景条件。
- 官方数据卡注明需接受 license，且存在 CARLA replay phantom pedestrian、log-depth map 等已知问题；使用前记录数据版本及过滤规则。
