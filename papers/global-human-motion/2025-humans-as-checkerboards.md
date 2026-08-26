---
title: "Humans as Checkerboards: Calibrating Camera Motion Scale for World-Coordinate Human Mesh Recovery"
authors: "Fengyuan Yang, Kerui Gu, Ha Linh Nguyen, Tze Ho Elden Tse, Angela Yao"
venue: "ICCV 2025"
year: 2025
reading_date: 2026-08-26
status: skimmed
tags:
  - 3d-human-pose
  - camera-calibration
  - world-coordinate
  - slam
  - metric-scale
---

# Humans as Checkerboards: Calibrating Camera Motion Scale for World-Coordinate Human Mesh Recovery

## 基本信息

- **作者：** Fengyuan Yang, Kerui Gu, Ha Linh Nguyen, Tze Ho Elden Tse, Angela Yao
- **会议/期刊：** ICCV 2025
- **年份：** 2025
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `camera-calibration`, `world-coordinate`, `slam`, `metric-scale`
- **论文：** [arXiv:2407.00574](https://arxiv.org/abs/2407.00574)
- **代码：** [MartaYang/HumansAsCheckerboards](https://github.com/MartaYang/HumansAsCheckerboards)
- **数据集：** EMDB2、EgoBody
- **项目主页：** [HAC](https://martayang.github.io/HAC/)

## 一句话总结

HAC 将人体视为天然的 metric calibration object，利用 human–scene contact 提供尺度线索，校准 monocular SLAM 的 camera-motion scale，从而显著改善 world-coordinate human mesh recovery。

## 研究问题与动机

单目 SLAM 可以恢复相机旋转和相对平移，但绝对尺度不确定；HMR 又通常只输出 camera-relative 人体。若简单把二者串联，尺度误差会直接放大到 world-space human motion。此前方法常依赖耗时的全局优化，HAC 尝试用人体本身提供 metric cue。

## 核心方法

系统以 VIMO 作为 HMR，以 masked DROID-SLAM 作为 camera branch。利用人体与地面接触时关节的 metric depth，把人体当作“checkerboard”估计场景/相机尺度；当接触点不可直接使用时，再借助 ground-plane fitting 与 RANSAC 获得尺度约束。动态人体区域在 SLAM 中被屏蔽，以减少错误 correspondence。

## 数据集与评价指标

主要测试 EMDB2 的动态相机序列，并在 EgoBody 上评估头戴视角下的鲁棒性。指标包括 W-MPJPE、WA-MPJPE、PA-MPJPE、RTE，以及 camera ATE / scale-related error。还比较后处理时间与优化式方法。

## 主要结果

EMDB2 上 HAC 的 W-MPJPE 约 197.2 mm、WA-MPJPE 71.0 mm、RTE 1.2%；TRAM 对应约 222.4 / 76.4 / 1.4，WHAM 的 W-MPJPE 约 354.8 mm。尺度消融中，不做 calibration 时 W-MPJPE 约 949.75 mm、ATE-S 7.25 m，引入 contact-joint calibration 后降至约 197.24 mm 与 0.79 m。1000 帧级别的后处理约数秒，显著快于全局优化方法。

## 优点

- 把人体从“动态干扰物”变成 camera-scale estimator 的有用几何约束。
- 明确解决 monocular SLAM 的 metric scale 问题。
- 速度快，适合作为 online / lightweight world-HMR 的校准模块。

## 局限

- 仍依赖 SLAM 的基本轨迹质量；SLAM catastrophic failure 或长时 drift 无法由 HAC 单独解决。
- contact cue 对腾空、滑动、复杂地形或接触不明确的动作更弱。
- ground-plane assumption 对非平坦场景需要扩展。

## 个人评价

HAC 对当前研究非常关键，因为它说明人体信息不仅可以改善最终 pose，还可以反向约束 camera。它比“把 camera R,t 当 feature”更接近真正的 camera-human mutual geometry。

## 与我的研究关联

可作为 `GT camera / ViPE camera / HAC-scale camera / jointly refined camera` 四级 baseline。对于 360 自拍与滑雪场景，可以研究 contact cue 是否需要从 ground plane 扩展为 terrain / snow-surface constraint，并用多 perspective 3D keypoints 提供额外尺度和方向约束。

## 后续阅读

- JOSH。
- PhysDynPose。
- OnlineHMR。
- 研究人体尺度、接触和多透视 reprojection 如何共同校正 camera trajectory。
