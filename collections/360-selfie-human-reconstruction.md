# 360° Selfie Human Reconstruction

## 研究问题

研究 360° 自拍或伴随式相机中，如何从 equirectangular / fisheye 观测生成多透视视图，并将人体局部 3D、相机运动与世界坐标轨迹统一起来。

## 方法脉络

1. 360° / fisheye 投影与 virtual camera 建模。
2. ERP → perspective views 的人体检测与 3D pose / mesh estimation。
3. 同中心多透视融合与双 360 真实 baseline 几何。
4. 物理相机 trajectory、intrinsics、metric scale 与 near-metric depth 估计。
5. Visual-only OVO、visual-inertial panoramic SLAM 与 long-sequence loop closure。
6. Camera-only geometry 与 human-aware constraints 联合的 world-coordinate human reconstruction。

## 关键论文

- [SAM 3D Body](../papers/3d-human-pose/2026-sam-3d-body.md) — 可作为每个 perspective view 的单目人体恢复前端。
- [VGGT](../papers/multiview-geometry/2025-vggt.md) — camera/depth/geometry prior。
- [AnyCam](../papers/multiview-geometry/2025-anycam.md) — casual video camera pose/intrinsics。
- [ViPE](../papers/multiview-geometry/2025-vipe.md) — 直接支持 pinhole / wide-angle / 360 panorama 的 camera intrinsics、trajectory 与 near-metric depth；Web360 提供约 2,000 个 ERP 视频的 camera pose / distance-map annotation。
- [360DVO](../papers/360-vision/2026-360dvo.md) — 针对 monocular ERP/360 视频的深度 visual odometry，用 spherical distortion-aware features 与 omnidirectional differentiable bundle adjustment 直接恢复 360 camera trajectory，并提供真实 360DVO benchmark。
- [PanoAir](../papers/360-vision/2026-panoair.md) — 将 ERP panorama、IMU、distortion-aware hybrid features 与 panoramic loop closure 统一进 metric VI-SLAM，并公开 17 条、15.8 km / 45 min、RTK GT 的真实 UAV panoramic visual-inertial benchmark。
- [EmbodMocap](../papers/multiview-geometry/2026-embodmocap.md) — 双移动相机的 metric human-scene reconstruction。
- [Kineo](../papers/multiview-geometry/2025-kineo.md) — calibration-free sparse RGB multi-camera geometry。
- [OnlineHMR](../papers/global-human-motion/2026-onlinehmr.md) — moving-camera world-grounded human motion。

## 当前共识

从同一个 360° 相机切出的 perspective views 共享相同 camera center，主要提供方向和观测完整性的互补，而不是传统 stereo baseline；多台 360 相机则额外提供真实平移几何。ViPE 说明物理 360° 相机的 trajectory 与 depth 可以直接在 panorama / ERP pipeline 中估计，因此 camera estimation 没有必要完全依赖 perspective crop 之后的独立求解。360DVO 进一步说明，直接在 spherical / ERP geometry 中学习 distortion-aware features 并执行 omnidirectional bundle adjustment，在强旋转和大 FoV 条件下可以成为更有针对性的 visual-only 360 camera baseline；但全景视野同时也会包含更多动态前景，因此在人群或强动态场景中可能增加错误匹配。

PanoAir 又补上了另一类关键 camera baseline：在 panorama 上融合 IMU 获得 metric scale，并通过 panoramic loop closure 显式纠正长序列 accumulated drift。其公开数据还同时提供 ERP、dual-fisheye、virtual pinhole、IMU 与 RTK ground truth，因此现在可以更系统地把 `visual-only OVO`、`visual-inertial panoramic SLAM` 和 `human-assisted camera refinement` 分开评价，而不是把所有 camera error 混在最终人体 MPJPE 中。

对于 human-centric reconstruction，更合理的分工是由 panorama-level geometry / VIO 估计物理 camera，再利用多个 perspective views 提供人体细节与跨视角人体约束；如果有 IMU，还应测试人体结构是否在已有 metric-scale inertial prior 的基础上继续降低 camera drift。

## 研究空白

- 公开领域仍缺少与“360° 自拍 + 全身 3D GT + camera trajectory GT”完全匹配的 benchmark；Web360 提供 camera pose / distance map，360DVO 提供 camera trajectory pseudo-GT，PanoAir 提供 RTK camera GT，但三者都不是 human-centric full-body 3D benchmark。
- **推断：**ViPE、360DVO 与 PanoAir 可以分别构成 learned geometry、visual-only spherical VO 和 visual-inertial metric SLAM 三类 camera-only baseline；将人体从“mask-out dynamic object”升级为显式 camera constraint，并比较 `visual-only / VIO / human-assisted VIO` 的 ATE/RPE，仍有清晰方法空间。
- ERP 重采样、FoV、yaw spacing、人体有效像素与下游 3D pose 误差之间缺少系统评价。
- Panorama camera ATE/RPE 的改善是否会同步转化为 world-human W-MPJPE/RTE 改善，需要在统一 benchmark 中验证。
- 360DVO 与 PanoAir 都指出 dynamic objects / large occlusion 仍可能破坏 camera estimation；雪地、天空和高速运动正是这一问题的高风险场景。
- PanoAir 的 stitched panorama 使用前后 fisheye optical centers 中点定义 virtual camera center；这种近似对精细 dual-fisheye geometry 和人体三角测量是否足够准确，需要单独验证。
- 现有 panorama VIO/SLAM 基本没有利用人体 velocity、contact、scale 或跨视角 skeleton consistency 作为 camera correction factor。

## 与我的研究关系

用于组织 360 自拍滑雪、多透视 3D kpt fusion、camera-aware joint optimization 和 synthetic/public benchmark 设计。ViPE、360DVO 与 PanoAir 现在可以构成三级 camera baselines：`learned panorama geometry / visual-only OVO / visual-inertial metric SLAM`，再接 `perspective HMR → human-assisted camera correction → joint world HMR`。这样既可以测试没有 IMU 时人体能否补 camera scale，也可以测试已有 IMU 时人体是否还能修正 rotation/translation drift。

PanoAir 数据的 17 条真实 UAV sequence、最高约 10 m/s 平均速度、快速 yaw、night 和最长约 1.9 km 轨迹，也很适合作为滑雪 camera module 的独立 stress test，先把 camera subsystem 的鲁棒性验证清楚，再进入人体重建实验。

## 下一步阅读 / 实验

- 在同一移动 360 视频上比较 ViPE、360DVO 与 PanoAir-style VI-SLAM 的 ATE/RPE、scale drift、tracking success、速度和 long-loop consistency。
- 用 PanoAir 的 ERP / dual-fisheye / virtual-pinhole 三种输入做 projection ablation，检查 stitched panorama virtual-center approximation 对 trajectory 的影响。
- 对 2/4/6/8 perspective views、不同 FoV 和 crop resolution 做人体消融。
- 区分单 360 同中心融合与双 360 有 baseline 融合。
- 比较 panorama-level camera estimation 与 perspective-crop camera estimation，特别关注快速 yaw/pitch/roll 与低纹理雪地。
- 测试 `dynamic mask → human-aware mask → human-as-constraint → human+IMU joint correction`，观察 camera ATE/RPE 与人体 W-MPJPE/RTE 是否同步改善。
