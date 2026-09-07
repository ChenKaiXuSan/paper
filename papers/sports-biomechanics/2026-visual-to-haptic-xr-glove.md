---
title: "Visual-to-Haptic Augmentation in XR: A Wearable Glove for Perceptual Grounding in Multimodal Interaction"
authors: "Mohd Faisal, Hamdi Elsaddik, Erhan Baturay Onural, Jihong Zhang, Fedwa Laamarti, Abdulmotaleb El Saddik"
venue: "ACM CHI 2026 Workshop on Shaping Future Human Connections: Social Augmentation through XR Technologies (SAXR 2026), CEUR Workshop Proceedings Vol. 4226"
year: 2026
reading_date: 2026-09-08
status: skimmed
tags:
  - haptics
  - wearable
  - multimodal
  - xr
  - visual-to-haptic
---

# Visual-to-Haptic Augmentation in XR: A Wearable Glove for Perceptual Grounding in Multimodal Interaction

## 基本信息

- **作者：** Mohd Faisal, Hamdi Elsaddik, Erhan Baturay Onural, Jihong Zhang, Fedwa Laamarti, Abdulmotaleb El Saddik
- **会议/期刊：** ACM CHI 2026 Workshop on Shaping Future Human Connections: Social Augmentation through XR Technologies (SAXR 2026), CEUR Workshop Proceedings Vol. 4226, pp. 252–262
- **年份：** 2026
- **Workshop 日期：** 2026-04-13
- **CEUR 发布日期：** 2026-07-17
- **arXiv 提交日期：** 2026-08-11
- **阅读日期：** 2026-09-08
- **阅读状态：** `skimmed`
- **标签：** `haptics`, `wearable`, `multimodal`, `xr`, `visual-to-haptic`
- **价值类型：** Method Module / Related Work
- **阅读优先级：** A（高）
- **论文：** https://ceur-ws.org/Vol-4226/paper27.pdf
- **arXiv：** https://arxiv.org/abs/2608.10368
- **DOI：** https://doi.org/10.48550/arXiv.2608.10368
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

该工作把图像/视频中的 motion、edge 与 brightness 特征映射为 29 个分布式 vibrotactile actuator 的时空强度，在 20 人 within-subject XR 实验中显著提高动态视频的主观 realism，但对静态 texture 没有显著增益；其真正可迁移价值是提供一个可解释的 `visual dynamics → spatial haptic cue` 模块，可进一步替换为人体 pose / biomechanics 驱动的智能触觉反馈。

## 研究问题与动机

XR 系统已经能够提供高质量视觉和听觉，但触觉反馈往往仍依赖预先编写的 haptic library 或固定事件映射，难以直接适应任意动态视觉内容。本文希望建立一个无需为每个内容单独设计 tactile pattern 的 visual-to-haptic layer，让普通图像与视频中的空间、时间视觉特征自动转成可穿戴手套上的分布式振动。

论文关注的是“perceptual grounding”而不是动作控制：视觉里出现的运动、结构和亮度变化被编码成对应的 tactile intensity，使用户看到的动态事件在时间和空间上得到触觉强化。

## 核心方法

系统采用四层架构：XR environment、media/content handling、visual-to-haptic processing 和 embedded haptic hardware。

视觉处理从图像/视频提取三类可解释特征：motion、edge/structure 与 brightness，然后进行归一化和加权线性融合，得到映射到手套 actuator grid 的 intensity map。为减少无意义振动，低于 15/255 的强度被抑制；视频使用 3-frame moving average 稳定时间变化。每次处理输出 29 维 actuator command。

硬件为作者自制的伸缩织物 glove，集成 **29 个 Linear Resonant Actuators (LRA)**，对应近似 5×7 空间布局；由 Teensy 4.1、DRV2605L drivers 与四个 I2C multiplexers 驱动。Actuator command 以 0–255 强度映射到驱动信号，并以 **150 ms interval** 更新。

## 数据集与评价指标

实验不是公开数据集 benchmark，而是受控 XR user study：

- 参与者：**20 人**，10 male / 10 female，年龄 20–35；
- 设备：Meta Quest 2 + 29-actuator glove；
- 环境：quiet indoor；
- 设计：within-subject；每位参与者都体验 visual-only 与 visual+haptic；
- Texture module：glass（smooth）与 sandpaper（rough）；
- Dynamic video module：flowing water 与 bouncing ping-pong ball；
- 每个 condition 约 2 min，顺序 counterbalanced；
- 评分：5-point Likert；主要比较 realism，并记录 immersion、timing/responsiveness、tactile–visual correspondence、engagement；
- 统计：paired-sample t-test，显著性阈值 α=0.05，使用 Cohen's dz 计算 repeated-measures effect size。

研究经 University of Ottawa Research Ethics Board 批准（H-09-23-9473）。

## 主要结果

在 visual+haptic 条件下，Texture / Video 的平均评分分别为：

- Realism：**3.50±1.32 / 3.95±1.05**；
- Immersion：**3.65±1.23 / 4.10±0.97**；
- Timing / Responsiveness：**3.90±1.02 / 4.30±0.86**；
- Tactile–Visual Correspondence：**3.15±1.42 / 3.75±0.97**；
- Engagement：**3.85±1.14 / 4.30±0.98**。

最关键的 baseline comparison 显示：

- 静态 texture realism：visual-only **3.45±1.39**，visual+haptic **3.50±1.32**，`t(19)=0.11, p=0.914, dz=0.03`，**无显著改善**；
- 动态 video realism：visual-only **3.25±1.33**，visual+haptic **3.95±1.05**，`t(19)=2.15, p=0.044, dz=0.48`，达到统计显著，effect size 为中等。

因此本文的证据主要支持“动态视觉事件经过触觉增强可提升感知 realism”，而不是证明该硬件对所有 texture / XR content 都有效。

## 优点

- Visual-to-haptic mapping 由 motion、edge、brightness 等明确特征组成，解释性强，比完全 black-box cross-modal generation 更容易分析 cue 来自哪里。
- 系统同时包含视觉算法、embedded hardware 和真实 user study，而不是仅离线生成 haptic signal。
- within-subject + counterbalanced 设计降低个体差异与 order effect，对原型系统验证较合理。
- 论文分别报告了显著与不显著结果，没有把 texture 的微小数值提升包装成有效改善。

## 局限

- 仅 **N=20**，且年龄集中在 20–35 岁，样本量与人群覆盖有限。
- 实验为 quiet indoor、single-user、Meta Quest 2 条件，不能直接外推到户外高速运动、临床康复或真实 sports coaching。
- 核心效果指标主要是 subjective Likert ratings，没有测量 reaction time、动作纠正幅度、技术表现、学习保持或 objective motor outcome。
- 静态 texture 的 realism 没有显著改善，作者也指出当前 actuator spatial granularity 不足；29 个振子仍较粗。
- 150 ms 更新间隔对高速动作反馈可能偏慢；**推断：**滑雪、球类或快速 gait correction 中，视觉处理、无线传输、执行器启动与人体反应延迟需要单独建立 end-to-end latency budget。
- 当前是 offline-synchronized / single-user setup，作者把 automated event-driven synchronization、多用户和 task-based performance evaluation 作为未来工作。
- 暂无官方代码、公开实验数据或独立项目主页，复现仍需要自行重建硬件和映射流程。

## 个人评价

这篇不是计算机视觉算法精度型论文，其价值更接近“智能触觉输出层”的工程与 HCI proof-of-concept。对当前研究最值得保留的结论是：动态 motion cues 比静态 texture 更容易通过低分辨率 vibrotactile array 获得稳定感知增益；这与体育/康复中强调动作时序变化的任务天然更匹配。

**推断：**如果直接把普通 optical motion 映射成振动，反馈仍然缺乏“技术意义”。真正值得发展的方向是把视觉层替换成结构化人体信息，例如 pose error、joint-angle deviation、CoM shift、contact timing、左右不对称或动作 phase，再通过空间/时间编码转成 haptic pattern。这样触觉不只是“增强视频感觉”，而是能表达可操作的运动纠正信息。

## 与我的研究关联

该工作与“智能触觉”方向直接相关，也可以连接体育视频与临床 gait：

- **推断：**滑雪技术反馈可用 `3D pose / world trajectory / CoM / edge-angle or turn phase → haptic cue`，把需要修正的身体部位、方向和时机编码到不同 actuator 区域；
- **推断：**临床 gait / rehab 可用左右步态不对称、trunk lean、joint ROM 或 phase event 生成可穿戴触觉提示；
- 可做 `generic RGB visual-to-haptic → pose-aware haptic → biomechanics-aware haptic` 消融，比较主观 realism 之外的动作误差、reaction time、correction success 与 retention；
- 与 VLM / coaching 系统结合时，可将自然语言解释与低延迟 haptic cue 分离：语言负责解释原因，触觉负责在动作发生的正确时刻给即时提示。

## 后续阅读

- 继续阅读 haptic sports coaching、rehabilitation biofeedback、vibrotactile gait cueing 与 spatial wearable haptics 文献。
- 设计以 pose / CoM / contact event 为输入的 actuator encoding，并明确端到端 latency、cue confusion matrix 和用户学习成本。
- 实验评价从主观 realism 扩展到 objective motor correction：joint-angle error、phase timing、balance/CoM deviation、task performance 与 retention。
- 若用于滑雪，优先测试低温、手套/厚衣物、运动冲击、无线延迟和 safety constraints 对可穿戴触觉可感知性的影响。
