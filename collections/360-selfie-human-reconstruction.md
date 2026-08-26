# 360° Selfie Human Reconstruction

## 研究问题

研究 360° 自拍或伴随式相机中，如何从 equirectangular / fisheye 观测生成多透视视图，并将人体局部 3D、相机运动与世界坐标轨迹统一起来。

## 方法脉络

1. 360° / fisheye 投影与 virtual camera 建模。
2. ERP → perspective views 的人体检测与 3D pose / mesh estimation。
3. 同中心多透视融合与双 360 真实 baseline 几何。
4. 物理相机 trajectory 估计与 world-coordinate human reconstruction。

## 关键论文

- [SAM 3D Body](../papers/3d-human-pose/2026-sam-3d-body.md) — 可作为每个 perspective view 的单目人体恢复前端。
- [VGGT](../papers/multiview-geometry/2025-vggt.md) — camera/depth/geometry prior。
- [AnyCam](../papers/multiview-geometry/2025-anycam.md) — casual video camera pose/intrinsics。
- [EmbodMocap](../papers/multiview-geometry/2026-embodmocap.md) — 双移动相机的 metric human-scene reconstruction。
- [Kineo](../papers/multiview-geometry/2025-kineo.md) — calibration-free sparse RGB multi-camera geometry。
- [OnlineHMR](../papers/global-human-motion/2026-onlinehmr.md) — moving-camera world-grounded human motion。

## 当前共识

从同一个 360° 相机切出的 perspective views 共享相同 camera center，主要提供方向和观测完整性的互补，而不是传统 stereo baseline；多台 360 相机则额外提供真实平移几何。

## 研究空白

- 公开领域仍缺少与“360° 自拍 + 全身 3D GT + camera trajectory GT”完全匹配的 benchmark。
- **推断：**多透视人体观测反向修正 360 物理相机 trajectory 的联合方法仍有明显空间。
- ERP 重采样、FoV、yaw spacing、人体有效像素与下游 3D pose 误差之间缺少系统评价。

## 与我的研究关系

用于组织 360 自拍滑雪、多透视 3D kpt fusion、camera-aware joint optimization 和 synthetic/public benchmark 设计。

## 下一步阅读 / 实验

- 补充 ViPE、360Loc 与 omnidirectional SLAM 文献。
- 对 2/4/6/8 perspective views、不同 FoV 和 crop resolution 做消融。
- 区分单 360 同中心融合与双 360 有 baseline 融合。
