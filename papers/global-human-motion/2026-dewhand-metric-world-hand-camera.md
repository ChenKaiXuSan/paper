---
title: "DewHand: Direct Metric World-Space Hand Motion Reconstruction from Egocentric Videos"
authors: "Naru Suzuki, Satoshi Ikehata, Takehiko Ohkawa, Yoichi Sato, Shuhei Kurita"
venue: "arXiv"
year: 2026
reading_date: 2026-11-24
status: skimmed
tags:
  - world-coordinate
  - camera-human
  - egocentric
  - hand-pose
  - flow-matching
  - metric-scale
---

# DewHand: Direct Metric World-Space Hand Motion Reconstruction from Egocentric Videos

## 基本信息

- **作者：** Naru Suzuki, Satoshi Ikehata, Takehiko Ohkawa, Yoichi Sato, Shuhei Kurita
- **会议/期刊：** arXiv 预印本；正式发表暂无核验
- **年份：** 2026
- **首次提交：** 2026-10-21
- **阅读日期：** 2026-11-24
- **阅读状态：** `skimmed`
- **标签：** world-coordinate, camera-human, egocentric, hand-pose, flow-matching, metric-scale
- **论文：** https://arxiv.org/abs/2610.29936 （arXiv:2610.29936）
- **代码：** 暂无
- **数据集：** [HOT3D](https://facebookresearch.github.io/hot3d/)、[ARCTIC](https://arctic.is.tuebingen.mpg.de/)（既有数据）
- **项目主页：** 暂无（论文链接指向 https://narusuzuki.github.io/projects/26-dewhand/ ，本次无法访问，未确认页面内容）
- **价值类型：** Method Module / Related Work / Baseline（手部—相机子任务）
- **阅读优先级：** 高

## 一句话总结

以共享公制坐标直接联合生成双手和相机运动，将场景几何与手部特征作为条件，避免独立轨迹组合带来的尺度及漂移误差。

## 研究问题与动机

传统 egocentric world-space hand recovery 常先预测局部手部再用独立 SLAM 轨迹变换到世界坐标。尺度不一致、遮挡与观测丢失会放大误差。论文关注直接生成共享世界表示，并保留真实尺度。

## 核心方法

- 冻结 VGGT-Ω 的场景几何特征，结合 HaMeR 手部特征；为缺失观测使用左右手专用 mask token，训练时随机遮挡帧、连续片段或整条手轨迹。
- 全局流同时建模相机及左右手腕轨迹，局部流建模 MANO articulation/shape。用同一 flow time 联合去噪，预测干净的状态；训练加入世界关节与相机监督。
- 使用两个 Euler 步、默认五个样本推理；无需另行组合 SLAM 轨迹。输入单目第一视角 RGB，输出共享公制世界中的双手及相机运动。

## 数据集与评价指标

HOT3D 官方数据覆盖 19 人、833 分钟、33 种物体；本文具体评价窗口：HaWoR split 606 个（1,205 hand-clips），StableHand split 244 个（485 hand-clips），ARCTIC 155 个。另有 HoloAssist 410、H2O 180、TACO 133 个零样本窗口。窗口为 150 帧、30 fps。HOT3D 训练后在 ARCTIC 微调。指标包括 PA-MPJPE、W-MPJPE、WA-MPJPE（SE(3)/Sim(3) 对齐）、腕部 WA、AccEr、MRRPE-A，以及相机 raw/对齐 ATE、旋转、RPE 和尺度准确率。SE(3) 不消除尺度误差。

## 主要结果

- Table 1：HOT3D HaWoR split，预测形状时 WA-MPJPE(SE(3)) 35.94 mm，HaWoR 84.80 mm；AccEr 1.89 对 14.77 m/s²。
- StableHand split 同为 GT shape 时 32.90 → 28.40 mm；ARCTIC 同为 GT shape 时 76.22 → 36.18 mm（均为 SE(3) WA-MPJPE）。
- Table 2：HOT3D 相机 raw ATE 为 31.5 mm，VGGT-Ω 为 77.0 mm，ViPE 为 96.4 mm；Sim(3) ATE 13.1 mm 对 VGGT-Ω 27.7 mm；尺度比在 1.10 内的窗口占 66.2%，ViPE 24.6%。
- Table 4：去掉相机监督时手 WA-MPJPE 从 35.94 变为 42.36 mm；L2 regression 为 45.13 mm，flow matching 为 35.94 mm，但 raw camera ATE 分别 31.4/31.5 mm，不能声称生成目标本身改善相机。
- 三个零样本数据集的刚体对齐 WA-MPJPE 均领先，但其他指标并非全面领先。

## 优点

同时报告尺度、原始与对齐相机误差及人体局部/全局误差。消融区分几何特征、相机监督和生成目标。使用缺失观测掩码明确建模遮挡。

## 局限

作者指出未来应扩展固定离线窗口及物体接触。StableHand split 排除重叠测试窗口，但同一序列的其他段仍在训练，不能称为严格 subject-disjoint。**推断：**手部模型不等于全身模型；camera supervision 改善手定位不证明手部信息反向改善相机。去掉 hand-specific feature 后相机误差仍接近，必须进一步做人体因子消融。代码未核验公开。

## 个人评价

**推断：**真正有价值的是将尺度与相机轨迹设为共同预测目标，且报告未经尺度对齐的相机误差；可作为现有独立 VO+HMR、MINT/ Human3R 路线的补充，不宜直接宣称已实现 full-body mutual refinement。

## 与我的研究关联

**推断：**双 360 滑雪可以将全身 pelvis trajectory、local joints 和双物理相机 SE(3) 放到共享生成状态，增加跨视角一致性及 rig 固定相对位姿约束；比较 camera-only、camera+body joint regression、joint flow、去掉人体特征和去掉相机监督。分别报告 camera raw/SE(3)/Sim(3) ATE、RPE、W-MPJPE、骨长、速度/加速度。需要独立 camera GT，而非从人体重投影误差推断相机精度。

## 后续阅读

- Sections 3.2–3.4、4.1–4.4、Tables 1–4、Appendix B/C；重点复现 mask token、共享 flow time、相机尺度评价。
- 对照仓库 MINT、PACE、Human3R、Fast-Camera 笔记；测试长序列 stitching 与双 360 spherical geometry。
- 事实来源：论文原文及官方 HOT3D/ARCTIC 页面；项目页面访问失败以原文替代。
