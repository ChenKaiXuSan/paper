---
title: "PanoAir: A Panoramic Visual-Inertial SLAM with Cross-Time Real-World UAV Dataset"
authors: "Yiyang Wu, Xiaohu Zhang, Yanjin Du, Tongsu Zhang, Chujun Li, Siyang Chen, Guoyi Zhang, Xiangpeng Xu"
venue: "arXiv"
year: 2026
reading_date: 2026-09-04
status: skimmed
tags:
  - 360-vision
  - panoramic-slam
  - visual-inertial-slam
  - camera-trajectory
  - rtkgps
---

# PanoAir: A Panoramic Visual-Inertial SLAM with Cross-Time Real-World UAV Dataset

## 基本信息

- **作者：** Yiyang Wu, Xiaohu Zhang, Yanjin Du, Tongsu Zhang, Chujun Li, Siyang Chen, Guoyi Zhang, Xiangpeng Xu
- **会议/期刊：** arXiv
- **年份：** 2026
- **阅读日期：** 2026-09-04
- **阅读状态：** `skimmed`
- **标签：** `360-vision`, `panoramic-slam`, `visual-inertial-slam`, `camera-trajectory`, `rtkgps`
- **论文：** https://arxiv.org/abs/2604.00852
- **代码：** 暂无（官方仓库说明代码将在论文接收后开源）
- **数据集：** https://github.com/YYWumm/PanoAir
- **项目主页：** https://github.com/YYWumm/PanoAir

## 一句话总结

PanoAir 将 ERP 全景视觉、IMU、失真感知特征权重和 panoramic loop closure 统一到完整的 metric VI-SLAM 中，并提供 17 条真实 UAV 轨迹、15.8 km / 45 min、RTK ground truth 的全景视觉惯性数据，为高速 360° moving-camera 研究提供了比纯视觉 VO 更接近真实部署的 camera baseline。

## 研究问题与动机

现有 panoramic VO/SLAM 往往只能解决其中一部分问题：纯单目 ERP 方法缺少 metric scale，部分 360-VIO 方法虽融合 IMU，但缺少完整 loop closure，长序列仍会积累漂移；另一方面，真实 UAV 的快速 yaw、低纹理、低照度和高动态运动与手持或地面车辆差异明显，公开 ERP+IMU+真实轨迹 benchmark 仍不足。

PanoAir 因此同时做两件事：第一，建立真实 UAV panoramic visual-inertial dataset；第二，在优化式 VI-SLAM 中直接使用 ERP 投影、IMU residual 和 panoramic loop closure，目标是恢复带尺度且长期一致的相机轨迹。

## 核心方法

系统输入 ERP panoramic image 与 IMU。视觉前端同时使用 ORB 与 SuperPoint，并通过 FeatureBooster 增强描述子。由于 ERP 在高纬度区域有强拉伸，作者根据纬度构造 distortion-aware feature weight，减弱上下边界特征对优化的错误影响。

后端以 panoramic camera pose、速度、陀螺仪 bias、加速度计 bias 和 3D landmarks 为状态，将 IMU residual 与 ERP reprojection residual 放入统一优化。Loop closure 使用 coarse-to-fine 流程：BoW 检索候选，经过 feature matching、Sim(3)、连续性和重力方向检查后，再通过 pose graph optimization 与 global bundle adjustment 修正累计漂移。

## 数据集与评价指标

PanoAir 数据集包含 17 条真实 UAV sequence，总距离约 15.8 km、总时长约 45 min，覆盖 cloudy / sunny / night、不同飞行速度、快速 yaw、升降和飞行抖动。设备为 Insta360 X3 + DJI Matrice 300 RTK；panorama / fisheye / virtual pinhole 影像为 30 Hz，IMU 为 1000 Hz，RTK ground truth 为 5 Hz。最长 Seq17 约 1901 m / 234 s，Seq5 平均速度达到 10.0 m/s。

评价主要使用 ATE、RPE、tracking success rate 和 runtime，并与 ORB-SLAM3、VINS-Mono、DROID-SLAM、OpenVSLAM、360DVO、360-VIO、OpenVINS 比较。每个 sequence 运行 5 次并报告平均结果。

## 主要结果

在作者的 17-sequence UAV benchmark 上，PanoAir 达到 100% tracking success，平均 ATE 为 0.739 m；360DVO 同样 100% 成功，但平均 ATE 为 1.398 m，OpenVSLAM 成功率 94%，ORB-SLAM3 为 83%。在公开 360-VIO benchmark 上，PanoAir 的平均 ATE / RPE 为 0.145 / 0.073 m，360DVO 为 0.307 / 0.068 m，说明后者局部相对运动仍有竞争力，但 PanoAir 的全局轨迹更准确。

Loop closure 的作用非常明显：消融中去掉 panoramic loop closure 后平均 ATE 从 0.578 m 恶化到 1.223 m。在 5.5 km handheld sequence 上，未 loop closure 的终点误差约为 (-9.92, -49.37, -28.43) m，开启后收敛到 (-1.39, -0.08, 0.36) m。PC 端平均 tracking 约 24.9 ms（40 FPS），Jetson Orin NX 约 114.2 ms（9 FPS）。

## 优点

- 同时提供 ERP、dual-fisheye、pinhole、IMU 与 RTK GT，可直接比较 panorama 与 perspective/fisheye camera pipeline。
- 不只是 VO，而是包含 metric VI optimization 与 panoramic loop closure，适合研究长序列 drift。
- 17 条真实 UAV sequence 同时覆盖高速、快速旋转、夜间和低纹理等移动相机困难条件。
- 官方数据已经公开，且相机/IMU calibration results 一并提供。

## 局限

- 作者明确指出当前没有显式建模 dynamic objects 与大面积 occlusion；强动态场景仍可能造成错误 pose / map。
- stitched panorama 的 camera–IMU extrinsic 并非真实单一光心，而是以前后 fisheye optical centers 中点定义 virtual camera center，这一点对精细的双鱼眼几何需要特别注意。
- 当前公开仓库尚未释放完整源代码。
- **推断：**UAV scene 中通常没有长期占据大画幅的人体目标，因此其 100% tracking success 不能直接外推到滑雪者始终位于画面中的 human-centric moving-camera 视频。

## 个人评价

这篇比 360DVO 更适合充当“带 IMU、带 metric scale、带 loop closure”的 360 camera-only baseline。360DVO 更像纯视觉 learned OVO，而 PanoAir 更适合回答长序列是否会漂、IMU 能否稳定尺度、回环是否真正降低 global drift。两者一起使用，可以把 camera baseline 拆成 visual-only 与 visual-inertial 两类，而不是只比较不同 HMR 后端。

## 与我的研究关联

**推断：**对双 360 滑雪系统，可以建立 `360DVO (visual-only) → PanoAir-style VI-SLAM (camera+IMU) → human mask → human-as-constraint → camera-human joint refinement` 的递进链。最重要的是同时报告 camera ATE/RPE/scale drift 与人体 W-MPJPE/RTE，检查人体约束是否在已有 IMU 的情况下仍能进一步修正 camera。

数据集本身也值得作为 camera module 的独立 stress test：其中最高平均速度约 10 m/s、包含快速 yaw、night 和长达 1.9 km sequence，与滑雪移动相机的强旋转、高速和长序列问题有较高结构相似性。尤其应比较 ERP 直接估计与 dual-fisheye / virtual pinhole 输入的差异。

## 后续阅读

- 360DVO：纯视觉 spherical / ERP camera-only baseline。
- 360-VIO：filter-based panoramic visual-inertial baseline。
- Sphere-VIO：heterogeneous multi-camera 的统一 spherical VIO representation。
- 实验上优先比较 `360DVO vs PanoAir-style VI-SLAM vs human-assisted VI-SLAM` 的 ATE/RPE、scale drift、tracking success 与 W-MPJPE。
