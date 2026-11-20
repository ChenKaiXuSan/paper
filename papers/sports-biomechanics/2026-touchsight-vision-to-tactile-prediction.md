---
title: "TouchSight: Bare-Handed Tactile Prediction from Egocentric Video via Generative Visual Augmentation"
authors: "Danyan Zhou, Jinxuan Lu, Jiawei Lin, Tianxing Chen, Chuqiao Lyu, Wenbo Ding"
venue: "arXiv:2609.20414"
year: 2026
reading_date: 2026-11-21
status: skimmed
tags:
  - sports-biomechanics
  - intelligent-haptics
  - vision-to-tactile
  - contact-force
  - egocentric-video
  - multimodal-learning
  - physical-interaction
---

# TouchSight: Bare-Handed Tactile Prediction from Egocentric Video via Generative Visual Augmentation

## 基本信息

- **作者：** Danyan Zhou, Jinxuan Lu, Jiawei Lin, Tianxing Chen, Chuqiao Lyu, Wenbo Ding
- **会议/期刊：** arXiv:2609.20414（尚未核验正式发表 venue）
- **年份：** 2026
- **提交日期：** 2026-09-17
- **阅读日期：** 2026-11-21
- **阅读状态：** `skimmed`
- **标签：** `vision-to-tactile`, `physical-interaction`, `egocentric-video`, `intelligent-haptics`
- **论文：** https://arxiv.org/abs/2609.20414
- **DOI：** https://doi.org/10.48550/arXiv.2609.20414
- **代码：** 暂无（未核验到作者发布的 TouchSight 官方代码）
- **数据集：** https://oakink.net/v2/ （论文使用的 OakInk2 官方基准）；TwinTouch-20H 的公开下载入口暂无
- **项目主页：** 暂无
- **论文状态：** 可信 arXiv 预印本
- **价值类型：** Method Module / Dataset / Related Work
- **阅读优先级：** 中高

## 一句话总结

TouchSight 用 500 小时手套实测力信号、10 小时生成式去手套视频和自然 HOI 接触标签共同训练，从裸手 egocentric RGB 视频直接预测双手密集触觉/接触，而无需推断时佩戴压力手套。

## 研究问题与动机

- 视频可观察人体动作，却难以直接测量接触起止、抓握载荷与局部受力分布；高质量触觉 GT 需要昂贵/侵入式传感设备。
- 压力手套数据虽含真实 taxel 信号，但画面中的手套和固定背景造成训练/部署 domain gap。
- 核心问题：在训练期保留可测力标签、部署期只用 RGB，如何学习真实 bare-hand 的接触力推断？

## 核心方法

1. **数据桥接：** 基于 HumanTouch 500 h 记录，选取 10 h gloved 片段，用生成式视频把手套外观和背景变成裸手自然场景，保留原始力标签；原始+生成构成 TwinTouch-20H（20 h paired material）。
2. **质量控制：** 以透明度 40% 叠加原始/生成视频人工检查时空对齐，排除生成手形、交互、动作发生漂移的片段。
3. **视觉特征：** 冻结 DINOv3 语义编码和 VGGT 几何编码，采用双向 cross-attention、per-token gating 和 temporal module 融合。
4. **Region-query decoder：** 对指区/掌区输出 taxel-level forces；额外共享 MANO 接触预测头，混合 force supervision 与自然 HOI 的 binary contact supervision。
5. **输入/输出：** T=8 帧、隔帧采样的 RGB clip → 每帧双手 275 个可信 taxels/手的预测受力，或双手 778 MANO vertices/手的二值接触；推断时不需要手部显式 pose 或手套。

## 数据集与评价指标

- **HumanTouch：** 500 h synchronized gloved RGB、手部信息与 taxel 实测力，是力监督主来源。
- **TwinTouch-20H：** 10 h 原始+10 h 生成配对；生成部分提供 bare-hand appearance 和继承的测力标签；生成视频经过人工对齐过滤。
- **HOI：** 五个公开视频来源，总 22.01 h，自然裸手但仅几何接触伪标签（不含实测接触力）。
- **OakInk2：** 论文使用的独立 HOI contact benchmark；以 hand–object mesh proximity 生成 binary contact labels；具体此文使用的测试帧数/受试者数在核验段落未统一报告，不虚构。
- **指标：** 全 taxel MAE、活跃 taxel MAE (GT force >5 raw glove units)、Vol.IoU、Temporal Pearson；OakInk2 上 AUROC、IoU、Precision、Recall、F1。
- **Baselines：** OakInk2 接触预测对比 BSTRO、DECO、HACO；TwinTouch/原始 HumanTouch 的主消融为 w/o AIGC、w/o VGGT、w/o temporal module。force MAE 为原始 glove units，不能直接标作牛顿。

## 主要结果

- **Table IV, OakInk2：** TouchSight AUROC **0.820**、IoU **0.230**、Precision **0.467**、Recall **0.311**、F1 **0.373**；HACO 为 AUROC 0.741、F1 0.329，但 HACO 的 Recall **0.623** 更高，不能声称所有指标最优。
- **Table III, TwinTouch bare-hand：** 用生成式数据训练后，MAE **1.851→1.671**、active-MAE **12.077→10.026**、Vol.IoU **0.239→0.300**、Temporal Pearson 0.255→0.272。
- **Table II, HumanTouch：** Full model MAE **1.639**、active-MAE **8.893**、Vol.IoU **0.360**、Temporal Pearson **0.567**；去掉 VGGT 后 Pearson 降至 0.451。
- **Table V：** 使用 0% vs 100% 的手套测力监督时，active-taxel Pearson **0.358→0.762**、active-MAE **7.71→3.12**（论文所设 seen tasks 条件）。跨数据集的 Ego-Exo4D/Ego4D/EgoDex 只报告定性演示，没有实测外部 force GT。

## 优点

- 清晰地区分实测 force、生成视频继承标签与仅几何推断的 contact 标签。
- 证明 geometry-aware tokens、时间模块和生成外观桥接可分别提升重要触觉指标；报告 active-MAE 避免大量零力区域主导 MAE。
- 对智能触觉、视觉-物理交互与跨模态训练提供可迁移的 region-query decoding 和混合监督策略。

## 局限

- 研究场景为手物接触，不能直接推断足底压力、雪板接触载荷或人体 GRF。
- 合成裸手画面可能存在接触/动作不完全一致；虽然人工剔除明显失败样本，但 GT 仍从原手套片段拷贝。
- OakInk2 ground truth 来自几何距离二值标签，而非实测接触力；自然数据的定量 force 泛化尚无独立验证。
- 模型对不同材质、遮挡、视角和身份的泛化，以及力单位的物理校准仍是应用边界；官方可运行代码和 TwinTouch-20H 公开下载暂未核验。

## 个人评价

**推断：**这篇的重要价值是把视觉运动表征连接到独立物理观测标签，而不是仅从姿态估计“物理上可能接触”。然而输出的 raw glove units、合成视频标签与几何接触伪标签不能混为同一种 GT。部署前应通过真实压力传感器进行盲测。

## 与我的研究关联

- **体育/滑雪生物力学：** **推断：**把 region-query 的掌指分区改为滑雪靴/足底/雪板受力分区；用鞋垫压力/IMU 采集 GT，训练 RGB+3D pose → pressure/contact 的弱监督模块，并独立评价实测压力而非只测 MPJPE。
- **CanonFuse3D 与物理残差：** **推断：**让视频几何 token 与姿态时间表征对齐，再引入 contact-head 约束动态姿态融合；使用 contact event F1、active-contact MAE、跨滑雪者 LOSO 评价实际收益。
- **ClinicalGated：** **推断：**临床步态模型可研究身体 ROI + 时序区域 queries 预测足底接触阶段，但无需把未经验证的估计力解释为患者真实 GRF。
- **建议复现：** 阅读 Section III 数据桥接与质量控制、Section IV 双向跨模态融合、Section V Tables II–V；优先复现 active-MAE、temporal Pearson 和区分真实/合成/几何标签的消融。

## 后续阅读

- OakInk2 官方数据与基准：https://oakink.net/v2/
- [Beyond Motion Patterns 原有笔记](../motion-understanding/2026-beyond-motion-patterns-physical-forces.md)：对比显式动力学信号如何提升 motion representation。
- [BadmintonGRF 原有笔记](../sports-biomechanics/2026-badmintongrf.md)：对比真实场景中可独立验证的 force/GRF 指标。
- 核验官方代码、TwinTouch-20H 数据发布状态；若未来评估滑雪，首先制定外部压力传感器/接触标签采集及受试者独立划分。

## 事实来源

- arXiv 官方全文：https://arxiv.org/html/2609.20414v1 （Sections III–V、Tables I–V）
- arXiv 摘要与提交日期：https://arxiv.org/abs/2609.20414
- OakInk2 官方项目：https://oakink.net/v2/
