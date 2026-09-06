---
title: "BodySLAM++: Fast and Tightly-Coupled Visual-Inertial Camera and Human Motion Tracking"
authors: "Dorian F. Henning, Christopher Choi, Simon Schaefer, Stefan Leutenegger"
venue: "IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS 2023)"
year: 2023
reading_date: 2026-09-07
status: skimmed
tags:
  - moving-camera
  - visual-inertial-slam
  - camera-human-mutual-refinement
  - smpl
  - factor-graph
  - real-time
---

# BodySLAM++: Fast and Tightly-Coupled Visual-Inertial Camera and Human Motion Tracking

## 基本信息

- **作者：** Dorian F. Henning, Christopher Choi, Simon Schaefer, Stefan Leutenegger
- **会议/期刊：** IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS 2023)
- **年份：** 2023
- **阅读日期：** 2026-09-07
- **阅读状态：** `skimmed`
- **标签：** `moving-camera`, `visual-inertial-slam`, `camera-human-mutual-refinement`, `smpl`, `factor-graph`, `real-time`
- **论文：** https://arxiv.org/abs/2309.01236
- **DOI：** https://doi.org/10.1109/IROS55552.2023.10342291
- **代码：** 暂无
- **数据集：** 暂无（论文描述并使用自建 BodySLAM 数据集，但本次未核验到独立官方数据下载页）
- **项目主页：** 暂无

## 一句话总结

BodySLAM++ 将人体状态与视觉惯性 SLAM 放进同一个紧耦合因子图中联合估计，是一个很早但非常直接的 `human → camera` 显式反馈先例：人体重投影与运动模型不仅改善人体 MPJPE，也能降低相机 ATE。

## 研究问题与动机

移动相机下的人体追踪通常把相机轨迹估计和人体姿态恢复拆开：先由 SLAM 得到 camera trajectory，再将单帧人体结果变换到世界坐标。问题是，人体会遮挡静态场景特征，而 camera error 又会传递到人体世界运动。BodySLAM++ 的目标是让两类状态在同一个优化问题里互相提供约束，而不是单向串联。

论文基于 OKVIS2 的 visual-inertial state estimation 框架，联合估计相机/传感器状态与人体 6D root pose、SMPL posture 和 shape。与只把人当动态干扰的 SLAM 不同，人体观测本身成为因子图中的有效测量。

## 核心方法

系统输入移动 stereo camera 的图像与 IMU，并使用 2D human joint observations。核心因子包括：

1. OKVIS2 原有的静态 landmark reprojection 与 IMU factors；
2. 人体关节重投影误差，将 SMPL 关节投影到相机图像并与观测对齐；
3. human motion model，对人体位置与姿态的时序变化施加约束；
4. body-shape / anthropometric priors，约束 SMPL shape 与人体结构；
5. 相机与人体状态在同一个非线性 least-squares / factor-graph 中联合优化。

因此，相机轨迹影响人体世界坐标，同时人体重投影与运动模型也反向参与相机轨迹优化。实现采用 Ceres Solver 和解析 Jacobian，在 Intel i7-12700K CPU 上达到 15+ FPS。

## 数据集与评价指标

论文自建 BodySLAM 数据。单人部分包含 `SP_01`–`SP_15` 共 15 个序列，使用移动 stereo visual-inertial camera，并由室内 optical motion-capture system 提供 6D camera pose 和 22 个人体关节的 3D ground truth。动作主要覆盖慢到中等速度的 walking、上下楼梯、坐下/站起、抓取与放置物体。

相机轨迹实验还使用存在三人的 multi-person 数据，论文将其描述为 `MP_01`–`MP_15`；结果表实际列出 MP_01–MP_13 与 MP_15。论文未明确给出独立受试者总人数，因此不自行推断。

人体指标为 MPJPE，相机指标为 ATE。主要 baseline 包括 `GT camera + SMPLify`、`ORB-SLAM3 + SMPLify`、`OKVIS2 + SMPLify`，以及 ORB-SLAM3 / OKVIS2 的相机轨迹。

## 主要结果

在 15 个单人序列上，平均 MPJPE 为：`GT + SMPLify 0.2401 m`、`ORB-SLAM3 + SMPLify 0.2380 m`、`OKVIS2 + SMPLify 0.1979 m`、`BodySLAM++ 0.1447 m`。相对最强 OKVIS2+SMPLify baseline，人体误差约下降 26%。

在 multi-person camera-trajectory 实验中，平均 ATE 为 `OKVIS2 0.0362 m`、`BodySLAM++ 0.0317 m`，约改善 12%；ORB-SLAM3 的成功序列平均值为 0.0476 m，但有序列跟踪失败。论文认为，在多人遮挡静态视觉特征时，human reprojection errors 与 human motion model 能为 SLAM 增加有效结构约束。

## 优点

- 不是简单的 `camera → human` pipeline，而是显式紧耦合地让 human factors 进入 camera optimization。
- 同时用人体 MPJPE 与 camera ATE 证明双向收益，评价目标和 camera-human mutual refinement 非常一致。
- 采用 stereo + IMU + factor graph，在 CPU 上达到 15+ FPS，具有在线系统意义。
- 数据同时提供 camera 与 human ground truth，能够把两类误差分开评价。

## 局限

- 数据主要是室内 optical mocap 环境，动作以慢到中等速度为主，与长距离高速户外运动差异明显。
- 输入是已标定的 stereo visual-inertial setup，不覆盖 monocular、未知 intrinsics、fisheye/ERP 或 360° camera geometry。
- 人体前端依赖 OpenPose 与 SMPL，属于较早一代人体表示与检测 pipeline，不能代表今天 foundation HMR 的能力。
- 人体 MPJPE 在比较前对完整 root trajectories 做了 6D alignment，因此不能等同于严格的未对齐 world-coordinate trajectory error。
- 论文没有现代 world-HMR 常用的 W-MPJPE、RTE、scale drift、长期 loop-closure robustness 等评价。

## 个人评价

这篇虽然是 2023 年论文，但对当前研究的价值非常高，因为它直接纠正了一个容易形成的错误印象：显式 `human → camera` feedback 并不是完全空白。BodySLAM++ 已经用经典 factor-graph / VI-SLAM 形式证明人体观测可以降低 camera ATE。

真正值得继续做的，不是重新声明“首次让人体帮助相机”，而是把这一思想推进到现代 HMR、foundation geometry、长序列户外运动和 360° camera：比较经典紧耦合 factor graph、Human3R 式 shared recurrent state、WHAC/SHOW/JOSH，以及显式 learned residual correction 的差异。

## 与我的研究关联

**推断：**对于双 360° 跟拍滑雪，可以把 BodySLAM++ 作为 camera-human mutual refinement 的经典强 baseline / Related Work。实验可组织为：

`camera-only SLAM/VIO → independent camera+HMR → BodySLAM++-style tightly-coupled human factors → Human3R shared state → learned human-assisted camera residual → joint refinement`。

关键不是只看 W-MPJPE，而应同时报告 camera ATE/RPE、rotation/translation/scale drift、W-MPJPE/RTE，以及在雪地弱纹理、人体大面积遮挡、快速旋转时 camera 指标是否因人体信息而改善。

**推断：**BodySLAM++ 中的 joint reprojection、anthropometric prior 和 temporal human motion factor 可以直接转化成现代约束；同时可增加跨 perspective / 双 360 一致性、foot contact、velocity/acceleration 等新因子，从而形成“经典显式因子图反馈 → 现代 learned / geometry-aware feedback”的清晰增量。

## 后续阅读

- 对比 BodySLAM++、WHAC、Human3R、SHOW、JOSH 中 `human → camera` 的作用位置：factor graph、scale、shared feature/state、dense geometry 与 joint optimization。
- 检查 BodySLAM 数据是否仍有作者维护的公开下载入口，并确认 multi-person 数据的完整序列数。
- 在自有数据中人为注入 camera rotation / translation / scale drift，测试 human reprojection、骨长、contact 与 temporal motion factors 能否定量降低 ATE/RPE。
- 将 pinhole/stereo factor 形式推广到 fisheye / spherical projection，验证双 360° camera 的显式 human-assisted camera correction。
