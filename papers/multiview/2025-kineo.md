---
title: "Kineo: Calibration-Free Metric Motion Capture From Sparse RGB Cameras"
authors: "Charles Javerliat, Pierre Raimbaud, Guillaume Lavoué"
venue: "arXiv:2510.24464"
year: 2025
reading_date: 2026-08-26
status: skimmed
tags:
  - multiview
  - camera-calibration
  - metric-reconstruction
  - 3d-human-pose
---

# Kineo: Calibration-Free Metric Motion Capture From Sparse RGB Cameras

## 基本信息

- **作者：** Charles Javerliat, Pierre Raimbaud, Guillaume Lavoué
- **会议/期刊：** arXiv:2510.24464
- **年份：** 2025
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `multiview`, `camera-calibration`, `metric-reconstruction`, `3d-human-pose`
- **论文：** [arXiv:2510.24464](https://arxiv.org/abs/2510.24464)
- **代码：** [liris-xr/kineo](https://github.com/liris-xr/kineo)
- **数据集：** EgoHumans、Human3.6M
- **项目主页：** [Kineo](https://liris-xr.github.io/kineo/)

## 一句话总结

Kineo 从未同步、未标定的稀疏 RGB 相机出发，利用音频同步、人体 2D keypoints、图优化、bundle adjustment 与人体尺度先验恢复 metric camera calibration 和 3D motion capture。

## 研究问题与动机

真实多相机采集往往缺乏严格同步和标定。传统 mocap 需要提前 calibration，降低部署便利性。Kineo 希望让人体本身成为相机标定线索，实现消费级 RGB 多相机的 calibration-free metric reconstruction。

## 核心方法

先用音频估计跨相机时间偏移；再从高置信度 2D keypoints 建立 SfM correspondence，估计外参、焦距、主点和 Brown–Conrady distortion。相机图通过最小生成树初始化，再执行 bundle adjustment。最终三角化 3D keypoints，并通过 SMPL 或 monocular metric depth 恢复绝对尺度。

## 数据集与评价指标

实验使用 EgoHumans 和 Human3.6M，评价 camera translation/angular error、W-MPJPE、PA-MPJPE 等。Human3.6M 为受控四视角数据，EgoHumans 提供更动态、更复杂的多人/egocentric 场景。

## 主要结果

EgoHumans 上完全 calibration-free 时，camera TE/AE 约 0.34 m / 0.69°，HSfM 约 2.09 m / 9.35°；W-MPJPE 从 HSfM 的约 1.04 m 降至 0.17 m。Human3.6M 上 W-MPJPE 可从约 0.47 m 降至 0.04 m。畸变建模对 EgoHumans 很重要：不估 distortion 时 W-MPJPE 约 0.41 m，估计后约 0.17 m。

## 优点

- calibration、同步、畸变和 metric scale 都在同一 pipeline 中处理。
- 明确利用人体关键点估 camera geometry，而不是把人体仅视为动态干扰。
- 对稀疏消费级相机部署很有实用价值。

## 局限

- 假设跨相机人物 re-identification 已知；实际复杂场景中 re-ID/segmentation 可能成为主要成本。
- 音频同步依赖合适的声学条件。
- 更适合多物理相机 calibration，而不是单 moving-camera 的长时 trajectory estimation。

## 个人评价

Kineo 非常适合作为 camera-human geometry 的互补参考：它证明 2D human observations 可以直接用于 camera calibration，而不只是后端 pose fusion。

## 与我的研究关联

在 EgoHumans / FreeMan 或双 360 物理相机设置中，可以比较 `已知标定 / Kineo-style calibration-free / ViPE trajectory / joint refinement`。其 distortion ablation 也提示 360 perspective crop 的投影模型和 intrinsics 必须明确建模。

## 后续阅读

- HAC。
- LAMP。
- 对比同中心 virtual views 与多物理相机 baseline 的几何差异。
