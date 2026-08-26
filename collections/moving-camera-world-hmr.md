# Moving-camera World HMR

## 研究问题

目标是在相机自身运动的情况下，将 camera-relative 人体姿态恢复为稳定、具有尺度意义的 world-coordinate 人体运动，并理解 camera、human 与 scene 三者如何互相约束。

## 方法脉络

1. **Camera estimation / scale**：先获得相机轨迹、内参和 metric scale。
2. **Camera-relative → world**：将 HMR/HPE 输出变换到统一世界坐标。
3. **Global refinement**：利用时序、人体尺度、接触或 motion prior 修正漂移。
4. **Joint optimization**：让 human、camera、scene 不再单向串联，而是互相校正。

## 关键论文

- [AnyCam](../papers/multiview-geometry/2025-anycam.md) — 从 casual videos 恢复 camera pose 与 intrinsics。
- [VGGT](../papers/multiview-geometry/2025-vggt.md) — 通用视觉几何基础模型，可提供 camera / depth / point-map prior。
- [MoRe](../papers/multiview-geometry/2026-more-motion-aware-4d-reconstruction.md) — 显式区分动态物体与 camera motion。
- [Humans as Checkerboards](../papers/global-human-motion/2025-humans-as-checkerboards.md) — 用人体接触提供 metric scale。
- [PhysDynPose](../papers/global-human-motion/2025-physics-based-human-pose-moving-camera.md) — moving-camera world HMR + physics refinement。
- [OnlineHMR](../papers/global-human-motion/2026-onlinehmr.md) — 在线 world-grounded HMR。
- [DuoMo](../papers/global-human-motion/2026-duomo.md) — camera-space / world-space 双 motion prior。
- [UniSH](../papers/global-human-motion/2026-unish.md) — feed-forward scene-camera-human metric alignment。
- [JOSH](../papers/global-human-motion/2026-josh-joint-optimization.md) — human-scene-camera joint optimization。
- [TROPHIES](../papers/multiview-geometry/2026-trophies.md) — 多视角 human-scene-camera temporal reconstruction。

## 当前共识

单纯把 SLAM/camera estimator 与 HMR 串联容易把尺度、漂移和动态前景误差传递到 world-space human motion；人体接触、人体尺度、动态 mask、scene geometry 和 temporal prior 都可以成为额外约束。

## 研究空白

- **推断：**现有工作多数仍是 `camera → human` 的单向依赖，真正稳定的 camera-human mutual refinement 仍较少。
- 长距离、高速、低纹理和强旋转场景中的 camera drift 仍是核心问题。
- 多透视/360 观测如何用于反向修正物理相机 trajectory 尚未形成成熟 benchmark。

## 与我的研究关系

该 collection 可直接支撑 moving-camera 3D human reconstruction 的 Related Work 与 baseline 设计，特别适合组织 `GT camera / predicted camera / human-assisted camera / joint refinement` 的递进实验。

## 下一步阅读 / 实验

- 补充 ViPE、MASt3R-SLAM 等 camera trajectory 工作。
- 比较 ATE/RPE 与 W-MPJPE/RTE 是否同步改善。
- 测试人体 2D/3D reprojection、尺度和 contact 对 camera correction 的独立贡献。
