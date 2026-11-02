---
title: "Camera Motion Agnostic Method for Estimating 3D Human Poses"
authors: "Seong Hyun Kim, Sunwon Jeong, Sungbum Park, Ju Yong Chang"
venue: "Sensors 22(20), 7975 (2022)"
year: 2022
reading_date: 2026-11-03
status: skimmed
tags:
  - global-human-motion
  - moving-camera
  - camera-motion-invariant
  - smpl
  - motion-prior
  - classic
---

# Camera Motion Agnostic Method for Estimating 3D Human Poses

## 基本信息

- **作者：** Seong Hyun Kim, Sunwon Jeong, Sungbum Park, Ju Yong Chang
- **会议/期刊：** Sensors 22(20), 7975；正式发表于 2022-10-19
- **年份：** 2022（arXiv 早期题名为 *Camera Motion Agnostic 3D Human Pose Estimation*，首次提交 2021-12-01）
- **阅读日期：** 2026-11-03
- **阅读状态：** `skimmed`
- **标签：** `global-human-motion`, `moving-camera`, `camera-motion-invariant`, `smpl`, `motion-prior`, `classic`
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高（重要经典补读）
- **论文：** https://doi.org/10.3390/s22207975
- **arXiv：** https://arxiv.org/abs/2112.00343
- **代码：** 暂无（原文所指 https://github.com/seonghyunkim1212/GMR 在 2026-11-03 核验返回 404，不作为有效公开代码链接）
- **数据集：** https://amass.is.tue.mpg.de/ （训练数据 AMASS；3DPW、Human3.6M 和自建合成数据用于评价）
- **项目主页：** 暂无
- **论文状态：** 同行评审正式期刊论文；不是新论文

## 一句话总结

用摄像机坐标无关的局部关节旋转序列回归相邻帧人体根部旋转与平移增量，再累积生成相对于初始帧的全局 SMPL 运动，构成不显式依赖 camera pose 的经典 motion-only baseline。

## 研究问题与动机

移动相机下，现成 HMR 的 global/root pose 往往在相机坐标系内表达，不能直接代表人的真实世界运动。作者将人体的局部关节相对旋转视为与观测坐标系选择相对独立的信号，尝试从局部动作模式恢复相机运动无关的全局运动增量。重点并不是求解相机位置，而是绕过 camera estimation 的一条替代路线。

## 核心方法

1. 现成视频姿态估计器 VIBE 输出 SMPL 局部关节旋转与形体；抛弃 VIBE 的 camera-relative root orientation。
2. GMR 将 23 个局部关节旋转组成的时序表示输入双向 GRU（最终采用 4 层、2048 hidden units），回归逐帧 root orientation / translation 的相对变化。
3. 以初始全局变换为单位矩阵，将局部 `ΔR, ΔT` 随时间积分得到 `G_t`，与局部 pose 和 SMPL 结合得到全球相对 mesh。因 `G_1=I`，并非独立测得地理世界坐标系的绝对初始位姿。
4. 训练包含 vertex offset、orientation（chordal rotation loss）、translation 和 smoothness；对 AMASS 时序进行反向播放数据增强。

## 数据集与评价指标

- **训练：** AMASS mocap 序列，重采样 10 fps；训练窗口 64 帧。
- **Human3.6M：** 3.6M 图像帧的数据集；使用 S9/S11 和真值局部姿态进行消融，不能混同为实际使用 3.6M 帧训练。
- **3DPW：** 原集 60 条户外序列；经背景 masking + COLMAP 获得 pseudo-GT 并人工过滤后，仅 **17 条**用于作者的全局运动评价。
- **自建合成：** AMASS 中 CMU 人体动作用于生成旋转/环绕运动相机视频；**16 名受试者、50 条**合成测试序列。
- **输入/输出：** 现成视频 HMR 的局部关节旋转序列 → 帧间 global root motion / 累积世界相对 SMPL mesh。
- **指标：** Orientation Motion Error (OME, °)、Translation Motion Error (TME, mm)、Vertex Motion Error (VME, mm)；均评价运动增量，不能当作 camera ATE 或绝对 W-MPJPE。

## 主要结果

- 3DPW（17 条，pseudo-GT）相对于 VIBE-CAM：OME **3.88° → 3.67°**，TME **49.83 → 38.55 mm**，VME **127.07 → 120.37 mm**。以 GT 局部 pose 输入时 VME **29.39 mm**，说明前端姿态错误是明显瓶颈。
- 合成 camera-motion-on：VIBE-CAM 的 TME/VME 为 **81.70 / 132.63 mm**，GMR 为 **39.27 / 108.08 mm**；camera-motion-off 时分别为 **58.01 / 117.28** 与 **36.37 / 105.11 mm**。
- Human3.6M 消融中 final GRU architecture + losses 将 VME 降至 **7.01 mm**；flip augmentation 后进一步至 **6.47 mm**（属于以 GT 局部姿态为输入的消融，不能与 3DPW 结果直接对比）。

## 优点

- 不要求获得 camera extrinsics，可作为检查“人体动作先验究竟可恢复多少 global motion”的独立 baseline。
- 明确把 local joint articulation 与 root motion 分离，报告增量方向/位移/mesh 误差。
- camera-motion-on/off 合成控制变量适合分析相机运动敏感性。

## 局限

- 3DPW 仅 17/60 条可构造作者所用 pseudo-GT，且 pseudo-GT 依赖 COLMAP + 人工过滤，外部验证规模有限。
- `G_1=I` 只定义相对于初始帧的世界系，不能证明恢复了绝对相机轨迹、实际地理坐标与独立 metric scale。
- 运动先验可能将不常见动作、滑雪器械约束或非典型步态拟合成常见动作；长期增量积分误差可能积累（推断）。
- 官方 arXiv 所列原始代码地址当前返回 404；需要确认可复现实现。
- 不能修正 camera R/t/intrinsics；不属于 joint camera-human refinement。

## 个人评价

**推断：**它应列入 `motion-only decoupling` 对照，而不是与 TRAM / MegaSaM 直接等同为 camera trajectory estimator。强结果并不代表现实双 360 相机精度会提高，因为它可能依赖训练集中动作模式，对旋转身体、滑雪周期动作及临床病理动作泛化不明。

## 与我的研究关联

**推断：**为 moving-camera world HMR 提供一个不依赖 SLAM 的对照分支：`SAM3D/VIBE pose → local-pose GMR → global root motion`。与 `camera-only 360DVO/ViPE + HMR`、`human-assisted camera+HMR` 三路线并行比较。针对滑雪建议用自有 Unity GT 检验 root velocity / displacement、drift、cycle consistency，同时分开报告 camera ATE/RPE 与 human W-MPJPE；针对 ASD gait，测试异样动作是否被通用 AMASS prior 过度平滑。

## 后续阅读

- 优先读原文 Sections 3.3–3.6（BiGRU、rotation/translation loss 与积分），Sections 4.2–4.5（17 条 3DPW pseudo-GT、相机运动合成消融和局限）。
- 与现有 [WHAM](2024-wham.md)、[TRAM](2024-tram-global-trajectory-and-motion.md)、[PACE](2024-pace.md)、[SynCHMR](2024-synchmr.md) 比较 camera independence 与 camera-human coupling 的定义。
- 再核验作者原始 GMR 代码仓库是否迁移；目前勿宣称代码仍公开可用。

## 一手资料

- 出版方论文：https://www.mdpi.com/1424-8220/22/20/7975
- arXiv 原文（方法、数据、表 1–6）：https://arxiv.org/abs/2112.00343
- AMASS 官方数据页：https://amass.is.tue.mpg.de/
