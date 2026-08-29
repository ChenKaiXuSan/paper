# 360° Selfie Human Reconstruction

## 研究问题

研究 360° 自拍或伴随式相机中，如何从 equirectangular / fisheye 观测生成多透视视图，并将人体局部 3D、相机运动与世界坐标轨迹统一起来。

## 方法脉络

1. 360° / fisheye 投影与 virtual camera 建模。
2. ERP → perspective views 的人体检测与 3D pose / mesh estimation。
3. 同中心多透视融合与双 360 真实 baseline 几何。
4. 物理相机 trajectory、intrinsics 与 near-metric depth 估计。
5. Camera-only geometry 与 human-aware constraints 联合的 world-coordinate human reconstruction。

## 关键论文

- [SAM 3D Body](../papers/3d-human-pose/2026-sam-3d-body.md) — 可作为每个 perspective view 的单目人体恢复前端。
- [VGGT](../papers/multiview-geometry/2025-vggt.md) — camera/depth/geometry prior。
- [AnyCam](../papers/multiview-geometry/2025-anycam.md) — casual video camera pose/intrinsics。
- [ViPE](../papers/multiview-geometry/2025-vipe.md) — 直接支持 pinhole / wide-angle / 360 panorama 的 camera intrinsics、trajectory 与 near-metric depth；Web360 提供约 2,000 个 ERP 视频的 camera pose / distance-map annotation。
- [360DVO](../papers/360-vision/2026-360dvo.md) — 针对 monocular ERP/360 视频的深度 visual odometry，用 spherical distortion-aware features 与 omnidirectional differentiable bundle adjustment 直接恢复 360 camera trajectory，并提供真实 360DVO benchmark。
- [EmbodMocap](../papers/multiview-geometry/2026-embodmocap.md) — 双移动相机的 metric human-scene reconstruction。
- [Kineo](../papers/multiview-geometry/2025-kineo.md) — calibration-free sparse RGB multi-camera geometry。
- [OnlineHMR](../papers/global-human-motion/2026-onlinehmr.md) — moving-camera world-grounded human motion。

## 当前共识

从同一个 360° 相机切出的 perspective views 共享相同 camera center，主要提供方向和观测完整性的互补，而不是传统 stereo baseline；多台 360 相机则额外提供真实平移几何。ViPE 说明物理 360° 相机的 trajectory 与 depth 可以直接在 panorama / ERP pipeline 中估计，因此 camera estimation 没有必要完全依赖 perspective crop 之后的独立求解。360DVO 进一步说明，直接在 spherical / ERP geometry 中学习 distortion-aware features 并执行 omnidirectional bundle adjustment，在强旋转和大 FoV 条件下可以成为更有针对性的 360 camera-only baseline；但全景视野同时也会包含更多动态前景，因此在人群或强动态场景中可能增加错误匹配。

对于 human-centric reconstruction，更合理的分工是由 panorama-level geometry 估计物理 camera，再利用多个 perspective views 提供人体细节与跨视角人体约束。

## 研究空白

- 公开领域仍缺少与“360° 自拍 + 全身 3D GT + camera trajectory GT”完全匹配的 benchmark；Web360 提供 camera pose / distance map，360DVO 提供 camera trajectory pseudo-GT，但都不是 human-centric full-body 3D benchmark。
- **推断：**ViPE 与 360DVO 都可以视为不利用人体结构的 camera-only baseline；将人体从“mask-out dynamic object”升级为显式 camera constraint，并比较 `ViPE / 360DVO / human-assisted camera` 的 ATE/RPE，仍有清晰方法空间。
- ERP 重采样、FoV、yaw spacing、人体有效像素与下游 3D pose 误差之间缺少系统评价。
- Panorama camera ATE/RPE 的改善是否会同步转化为 world-human W-MPJPE/RTE 改善，需要在统一 benchmark 中验证。
- 360DVO 表明动态目标和重复/弱纹理区域仍会造成错误匹配；雪地、天空和高速运动正是这一问题的高风险场景。

## 与我的研究关系

用于组织 360 自拍滑雪、多透视 3D kpt fusion、camera-aware joint optimization 和 synthetic/public benchmark 设计。ViPE 与 360DVO 可以共同构成“完全不利用人体”的 camera-only baselines，从而形成 `ViPE / 360DVO panorama camera → perspective HMR → human-assisted camera correction → joint world HMR` 的递进实验，并清楚量化人体信息对 camera trajectory 的真实贡献。

## 下一步阅读 / 实验

- 在同一移动 360 视频上直接比较 360DVO 与 ViPE panorama camera 的 ATE/RPE、稳定性、速度和尺度漂移，再补充 360Loc / 360-VIO / omnidirectional SLAM 文献。
- 对 2/4/6/8 perspective views、不同 FoV 和 crop resolution 做消融。
- 区分单 360 同中心融合与双 360 有 baseline 融合。
- 比较 panorama-level camera estimation 与 perspective-crop camera estimation，特别关注快速 yaw/pitch/roll 与低纹理雪地。
- 测试 `dynamic mask → human-aware mask → human-as-constraint` 三种策略，观察 camera ATE/RPE 与人体 W-MPJPE/RTE 是否同步改善。
