---
title: "WATCH: World-aware Allied Trajectory and pose reconstruction for Camera and Human"
authors: "Qijun Ying, Zhongyuan Hu, Rui Zhang, Ronghui Li, Yu Lu, Zijiao Zeng"
venue: "arXiv:2509.04600"
year: 2025
reading_date: 2026-09-12
status: skimmed
tags:
  - world-coordinate
  - moving-camera
  - camera-human
  - camera-trajectory
  - global-motion
  - smpl-x
  - temporal
---

# WATCH: World-aware Allied Trajectory and pose reconstruction for Camera and Human

## 基本信息

- **作者：** Qijun Ying, Zhongyuan Hu, Rui Zhang, Ronghui Li, Yu Lu, Zijiao Zeng
- **会议/期刊：** arXiv:2509.04600
- **年份：** 2025
- **提交日期：** 2025-09-04
- **阅读日期：** 2026-09-12
- **阅读状态：** `skimmed`
- **标签：** `world-coordinate`, `moving-camera`, `camera-human`, `camera-trajectory`, `global-motion`, `smpl-x`, `temporal`
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** A+（最高）
- **论文：** https://arxiv.org/abs/2509.04600
- **DOI：** https://doi.org/10.48550/arXiv.2509.04600
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

WATCH 将 camera orientation decomposition 与 camera translation trajectory 作为显式时序信息融入 world-space human reconstruction，在 RICH 与 EMDB 上改善人体全局轨迹、抖动与 foot sliding；它最适合作为“camera motion 如何帮助 human world reconstruction”的强 baseline，但并未形成可独立量化的 human residual 反向修正 camera trajectory 的闭环。

## 研究问题与动机

移动单目视频中的 global human motion reconstruction 同时受到人体运动与相机运动耦合的影响。现有 human-motion-centric 方法通常能够保持人体姿态和时序合理性，但对相机 orientation 的利用不充分；另一方面，camera translation 往往被简单 hard-decoding 或在后处理阶段用于坐标变换，难以稳定进入 learned temporal representation。

WATCH 的目标是把 camera-human motion relationship 更直接地放入时序模型中。作者重点处理两个问题：一是将相机旋转分解为更适合人体 world motion 的 heading / non-heading components；二是设计 camera trajectory integration，使 camera local translation velocity 作为可学习的条件信息影响人体全局轨迹，而不是只在最终几何变换时使用。

## 核心方法

WATCH 使用视频图像、人体检测框、2D keypoints 与相机运动信息构建统一时序表示。图像特征来自冻结的 HMR2.0 encoder，2D keypoints 由 ViTPose 提供，时序 backbone 使用带 RoPE 的 Transformer。

核心设计包括：

1. **Analytical heading angle decomposition**：将 camera orientation 拆分为与 world heading 相关和无关的部分，减少相机旋转与人体朝向耦合带来的歧义。
2. **Camera trajectory integration**：将相机局部平移速度作为时序特征“软”融入网络，让模型学习 camera translation 与 human global motion 的关系，而不是直接把 SLAM translation 作为固定世界轨迹。
3. **Joint camera-human temporal modeling**：模型同时处理 camera-space human state、world-space human trajectory 与 camera motion cues，使人体局部姿态与全局位移在同一时序框架中优化。
4. **Multi-task supervision**：除了常规 HMR loss，还加入 human trajectory consistency 与 camera motion constraint，约束人体和相机时序关系。

## 数据集与评价指标

训练数据组合包括 AMASS、BEDLAM、Human3.6M 与 3DPW。对于 AMASS，作者进一步合成相机运动以补充 moving-camera supervision。训练 sequence length 为 120，batch size 为 128。

主要 world-space 测试集包括：

- **RICH：** 191 videos，约 59.1 min；
- **EMDB-2：** 25 sequences，约 24.0 min；
- camera-space evaluation 还使用 **EMDB-1：17 sequences / 13.5 min** 与 **3DPW：37 sequences / 22.3 min**。

全局指标包括 WA-MPJPE100、W-MPJPE100、RTE、Jitter 与 Foot-Sliding；camera-space 指标包括 PA-MPJPE、MPJPE、PVE 与 Acceleration Error。论文中的 RTE 按 GT displacement 的百分比定义，因此不应自行换算成米或毫米。

## 主要结果

在 RICH 上，WATCH 达到 **74.3 WA-MPJPE100、119.0 W-MPJPE100、2.4 RTE、10.6 Jitter、2.6 Foot-Sliding**；对应 GVHMR 为 **78.8 / 126.3 / 2.4 / 12.8 / 3.0**。在 EMDB-2 上，WATCH 为 **106.4 / 269.3 / 1.7 / 14.4 / 3.3**，GVHMR 为 **111.0 / 276.5 / 2.0 / 16.7 / 3.5**。

Camera trajectory ablation 更能说明其贡献：在 EMDB-2 + DPVO 条件下，去掉 camera trajectory integration 时为 **110.4 WA-MPJPE100 / 275.9 W-MPJPE100 / 2.0 RTE / 15.4 Jitter / 3.5 Foot-Sliding**；加入后改善为 **107.6 / 272.2 / 1.9 / 14.8 / 3.3**。使用 GT gyro 后达到 **106.4 / 269.3 / 1.7 / 14.4 / 3.3**。

Camera-space 的 EMDB-1 上，WATCH 达到 **42.7 mm PA-MPJPE、70.4 mm MPJPE、82.1 mm PVE**；GVHMR 为 42.7 / 72.6 / 84.2 mm。

需要注意，WATCH 并非所有 global metric 都最好。例如 EMDB-2 上 camera-trajectory-centric PromptHMR-vid 的 WA/W-MPJPE100 与 RTE 为 **71.0 / 216.5 / 1.3**，说明强化 human temporal plausibility 与追求绝对 global trajectory accuracy 之间仍存在方法取舍。

## 优点

- 把 camera translation 从简单后处理提升为 learned temporal conditioning，对 moving-camera world HMR 很直接。
- camera orientation 使用解析分解，结构清楚，便于与不同 SLAM / VIO 输入组合。
- 同时报告 global position、relative trajectory、jitter 与 foot sliding，避免只看静态位置误差。
- Camera trajectory ablation 明确显示 translation cue 对人体 world-space reconstruction 有可量化收益。

## 局限

- WATCH 仍需要外部 camera trajectory / motion source，例如 DPVO 或 GT gyro；当上游 camera trajectory 有严重 drift 时，模型本身不能保证修复。
- **推断：**其主要信息流仍是 `camera motion → human reconstruction`，虽然采用 joint modeling 和 camera-motion constraints，但论文没有建立一个可单独量化的 `human residual → camera rotation / translation / scale correction` 闭环，因此不应等同于完整 camera-human mutual refinement。
- 作者在 RICH 等相机更静态的场景中观察到相对收益缩小，说明其设计优势依赖明显的 camera motion。
- 论文没有针对长距离 camera ATE/RPE、fisheye/ERP、360° geometry 或变化 intrinsics 做专门评价。
- 当前为 arXiv 预印本，本次没有核验到已公开的官方代码、独立数据页或项目主页；arXiv 页面仅说明代码将公开。

## 个人评价

这篇对当前研究的价值主要在于补齐“**camera translation 如何进入 learned human world trajectory**”这一层。WHAC 强调 human-derived metric scale，BodySLAM++ 强调 factor-graph 中的人体反向约束 camera，SynCHMR 强调 human-aware metric SLAM，而 WATCH 更像把 camera orientation/translation 作为人体时序重建的结构化输入。

因此它适合作为 camera-human mutual refinement 的前一层 baseline，而不是最终目标。若新的方法能够在 WATCH-style camera conditioning 之外，让 human reprojection、骨长、contact、跨视角 consistency 或 high-order motion residual 反向降低 camera ATE/RPE，就能更清楚证明真正的双向 refinement。

## 与我的研究关联

对双 360° / skiing moving-camera reconstruction，可以构造：

1. camera-only trajectory + HMR；
2. WATCH-style camera orientation / translation conditioning；
3. + dual-view / dual-360 human fusion；
4. + human-assisted camera scale；
5. + explicit human residual → camera pose correction；
6. + global / loop refinement。

**推断：**在 360° 系统中，建议分别输入 physical-camera trajectory confidence、每个 perspective crop 的 human confidence 与 cross-view consistency，并把 camera ATE/RPE、scale drift、W-MPJPE/RTE、Jitter 和 Foot-Sliding分开报告。这样可以判断 camera cue 是只改善人体输出，还是人体 evidence 也真正稳定了 camera trajectory。

## 后续阅读

- 与 WHAC、BodySLAM++、SynCHMR、Human3R、HTD-Refine 对比 camera-human information flow。
- 重点复现 camera trajectory integration ablation，并用不同质量的 DPVO / SLAM trajectory 注入受控 rotation、translation、scale drift。
- 扩展到 fisheye / ERP geometry，测试 camera local velocity representation 在 360° moving-camera 中是否仍稳定。
- 增加 `camera confidence → human` 与 `human confidence → camera` 双向实验，区分 shared modeling 和真正 mutual refinement。
