# Multi-view Pose Fusion

## 研究问题

研究多个视角下的人体 2D/3D 表示如何在已知或未知 camera geometry 条件下融合，以减少遮挡、深度歧义和单视角估计偏差。

## 方法脉络

1. 单视角 HPE/HMR 作为 per-view estimator。
2. Camera calibration / relative pose / geometry estimation，以及跨视频 synchronization。
3. Cross-view feature / pose fusion，以及显式 multi-view algebraic geometry constraints。
4. Confidence、uncertainty、temporal consistency 与 metric-world alignment。

## 关键论文

- [Mocap-2-to-3](../papers/3d-human-pose/2026-mocap-2-to-3.md) — multi-view lifting 与 2D pretraining。
- [Towards Balanced Multi-Modal Learning in 3D HPE](../papers/3d-human-pose/2026-balanced-multimodal-3d-hpe.md) — 多模态贡献平衡与融合优化。
- [EgoPoseFormer v2](../papers/3d-human-pose/2026-egoposeformer-v2.md) — calibrated multi-view conditioning 与时序建模。
- [Spatiotemporal Multi-Camera Calibration using Freely Moving People](../papers/multiview-geometry/2025-spatiotemporal-multicamera-calibration-people.md) — 把人体 3D motion 当作动态 calibration target，联合估计 camera `R/t`、time offset 与跨视角 person association。
- [Unconstrained Multi-view Human Pose Estimation with Algebraic Priors](../papers/multiview-geometry/2026-uncalibrated-multiview-algebraic-priors.md) — 不输入 camera intrinsics/extrinsics，以 learned triangulation、Gröbner basis 多视图代数约束和 temporal equivariance 同时恢复 3D pose 与隐式 camera geometry。
- [Kineo](../papers/multiview-geometry/2025-kineo.md) — 无标定 sparse RGB cameras。
- [Flex4DHuman](../papers/multiview-geometry/2026-flex4dhuman.md) — flexible multi-view video modeling。
- [Human4K](../papers/multiview-geometry/2026-human4k.md) — 高分辨率 multi-view mocap benchmark。
- [LAMP](../papers/multiview-geometry/2026-lamp.md) — metric 3D world 中的 multi-camera localization/tracking。
- [TROPHIES](../papers/multiview-geometry/2026-trophies.md) — human-camera-scene multi-view reconstruction。

## 当前共识

多视角的价值不仅来自“更多图像”，还来自不同遮挡模式、不同深度歧义以及明确的 camera geometry；融合模型需要区分可靠视角与异常视角。Calibration-free 并不意味着应完全抛弃 geometry：Gröbner basis 等显式 projective constraints 可以作为 learned fusion 的结构先验，减少模型仅记忆固定 camera rig 的风险。

同时，**calibration 与 synchronization 本身也可以利用人体运动反向求解**。Spatiotemporal Multi-Camera Calibration using Freely Moving People 表明，单目 3D pose 中的关节方向和时间变化可以作为动态 calibration pattern，同时优化 camera rotation / translation、time offset 和跨视角人员 association。因此更合理的 multi-view pipeline 不一定是“先独立标定，再做人”，而可以是 human observation 与 camera geometry 互相提供约束。

评价 calibration-free / self-calibrating fusion 时不能只看最终 MPJPE。若方法声称同时恢复或隐式估计 camera geometry，应尽可能单独报告 camera rotation、translation、scale、relative-pose error 和 temporal-offset error，并测试 camera layout / focal length / perturbation 的变化，否则难以区分真正的 geometry generalization 与对固定采集系统的统计适配。

## 研究空白

- calibration-free fusion 与显式 camera-aware fusion 的公平比较仍不充分。
- human-based spatiotemporal calibration 已能在**静止、已知 intrinsics** 的多相机网络中工作，但 moving camera、变化 intrinsics、fisheye/ERP 以及长序列在线 refinement 仍缺少系统验证。
- 现有强 calibration-free benchmarks 仍大量建立在 Human3.6M、CMU Panoptic 等固定、同步实验室 camera rigs 上；真正 unseen camera layout、moving camera、变化 intrinsics 与 outdoor high-speed 场景验证仍不足。
- 同中心多透视（360 crop）与多中心真实多相机的几何性质不同，不能简单当成同一设定；标准 epipolar / multilinear constraints 在 shared-center virtual views 上可能退化。
- **推断：**camera uncertainty、joint-wise pose confidence、temporal offset uncertainty 的联合建模可能比单独 attention 更稳定。
- global metric scale 在纯人体 spatiotemporal calibration 中仍可能不确定，需要 scene prior、已知人体尺度或其他 metric cue。
- 只报告 pose MPJPE 而缺少 camera `R/t/scale` 和 synchronization 定量误差，会掩盖 calibration-free 方法是否真的恢复了正确 camera geometry。

## 与我的研究关系

可用于组织双视角/多透视 3D pose fusion、camera perturbation robustness，以及 calibration-based 与 calibration-free 方法的对比。一个清楚的递进实验可以是：

`checkerboard/QR + audio sync → human-motion spatiotemporal calibration → camera-parameter-free pose fusion → learned triangulation → + algebraic geometry prior → + temporal equivariance → human-camera joint refinement`。

对于双 360 物理相机，可以测试初始 offline calibration 后，是否能利用滑雪者运动进一步 refinement `R/t/time offset`，并统一报告 camera error、sync error 和 3D pose/world-trajectory error。对于 360 场景，还需要把本文基于 pinhole epipolar geometry 的部分替换为 spherical/fisheye-consistent geometry。

对于单台 360 切出的同中心 perspective views，则仍应与双 360 真 baseline 分开评价。**推断：**后者更适合直接测试 epipolar / Gröbner-style multi-center constraints；前者需要 spherical 或 shared-center-specific geometry。

## 下一步阅读 / 实验

- 复现 human-motion-based `R/t/time offset` estimation，与 checkerboard + audio/QR baseline 做定量比较。
- 统一评估 MPJPE、PA-MPJPE、W-MPJPE 与 camera rotation / translation / scale / temporal-offset error。
- 增加 view masking、joint masking、camera noise、focal-length perturbation、人工时间偏移和 cross-layout zero-shot。
- 比较显式 `R,t`、learned camera latent、无 camera 输入，以及 `+` algebraic geometry constraints。
- 在 Human3.6M/CMU 之外增加 moving-camera 或自建双 360 数据，检查 fixed-rig calibration-free performance 能否迁移。
- 分别测试双 360 真 baseline 与单 360 shared-center perspective crops，明确多视图代数约束和 human-based calibration 的适用边界。
