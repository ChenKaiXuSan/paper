---
title: "DuoMo: Dual Motion Diffusion for World-Space Human Reconstruction"
authors: "Yufu Wang, Evonne Ng, Soyong Shin, Rawal Khirodkar, Yuan Dong, Zhaoen Su, Jinhyung Park, Kris Kitani, Alexander Richard, Fabian Prada, Michael Zollhöfer"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - 3d-human-pose
  - world-coordinate
  - motion-diffusion
  - moving-camera
---

# DuoMo: Dual Motion Diffusion for World-Space Human Reconstruction

## 基本信息

- **作者：** Yufu Wang, Evonne Ng, Soyong Shin, Rawal Khirodkar, Yuan Dong, Zhaoen Su, Jinhyung Park, Kris Kitani, Alexander Richard, Fabian Prada, Michael Zollhöfer
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `world-coordinate`, `motion-diffusion`, `moving-camera`
- **论文：** [arXiv:2603.03265](https://arxiv.org/abs/2603.03265)
- **代码：** [facebookresearch/DuoMo](https://github.com/facebookresearch/DuoMo)
- **数据集：** AMASS、BEDLAM、3DPW、Goliath；评测 EMDB、RICH、EgoBody
- **项目主页：** 暂无

## 一句话总结

DuoMo 将 camera-space motion recovery 与 world-space motion refinement 拆成两个 diffusion prior，先显式利用 camera pose 把人体提升到世界坐标，再用全局运动生成模型吸收 camera noise、遮挡和深度歧义。

## 研究问题与动机

moving-camera 视频中，局部 pose 往往比绝对 global trajectory 更容易估计。直接端到端同时学习局部姿态和世界坐标运动会受到 camera error 与尺度问题影响。DuoMo 因此把问题分解为局部恢复与全局修正两阶段。

## 核心方法

第一阶段从 RGB、dense 2D keypoints 与相机内参估计 camera-space motion；利用外部 camera pose 显式变换到 world frame。第二阶段 world-space diffusion model 对 root-centered mesh、root velocity 与时序缺失进行再生成，使用 temporal masking 提高遮挡鲁棒性。人体采用 595 个 sparse mesh vertices 表示。

## 数据集与评价指标

训练使用 AMASS、BEDLAM、3DPW、Goliath，评测覆盖 EMDB、RICH、EgoBody。关注 PA/MPJPE、W-MPJPE、RTE 等 world-space 指标，并测试 camera noise 对结果的影响。

## 主要结果

论文报告在 EMDB 上带 height condition 时 W-MPJPE 约 167.1 mm、RTE 1.1%；RICH 上约 80.4 mm、RTE 1.3%。作者强调相较既有方法 world-space error 有明显下降，并通过 camera-noise robustness 实验证明第二阶段可以吸收部分外部 camera 误差。

## 优点

- 明确把 camera-space 与 world-space reconstruction 分开建模。
- 直接研究 camera noise 对 human reconstruction 的影响。
- world-motion prior 对遮挡、缺失和 global trajectory refinement 都有作用。

## 局限

- camera pose 仍作为外部输入，第二阶段主要吸收误差，而不会显式输出被人体反向修正后的 camera trajectory。
- 没有显式 3D scene geometry，复杂人—场景接触仍可能不一致。
- binary visibility 会丢失连续检测置信度。

## 个人评价

DuoMo 很适合用来区分“camera estimator 必须非常准”与“world-motion prior 可以容忍 camera noise”两个问题。对于联合优化研究，它可以作为 human-only refinement 的强对照。

## 与我的研究关联

可比较 `ViPE camera + direct fusion`、`ViPE camera + DuoMo-style world refinement` 与 `camera-human mutual refinement`。如果只使用 skeleton 而非 mesh，也可以保留两阶段思想，将第二阶段改成 3D keypoint motion prior。

## 后续阅读

- OnlineHMR。
- JOSH。
- HAC。
- 研究 continuous uncertainty 与 camera correction 的联合建模。
