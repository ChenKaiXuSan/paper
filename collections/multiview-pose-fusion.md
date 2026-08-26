# Multi-view Pose Fusion

## 研究问题

研究多个视角下的人体 2D/3D 表示如何在已知或未知 camera geometry 条件下融合，以减少遮挡、深度歧义和单视角估计偏差。

## 方法脉络

1. 单视角 HPE/HMR 作为 per-view estimator。
2. Camera calibration / relative pose / geometry estimation。
3. Cross-view feature / pose fusion。
4. Confidence、uncertainty、temporal consistency 与 metric-world alignment。

## 关键论文

- [Mocap-2-to-3](../papers/3d-human-pose/2026-mocap-2-to-3.md) — multi-view lifting 与 2D pretraining。
- [Towards Balanced Multi-Modal Learning in 3D HPE](../papers/3d-human-pose/2026-balanced-multimodal-3d-hpe.md) — 多模态贡献平衡与融合优化。
- [EgoPoseFormer v2](../papers/3d-human-pose/2026-egoposeformer-v2.md) — calibrated multi-view conditioning 与时序建模。
- [Kineo](../papers/multiview-geometry/2025-kineo.md) — 无标定 sparse RGB cameras。
- [Flex4DHuman](../papers/multiview-geometry/2026-flex4dhuman.md) — flexible multi-view video modeling。
- [Human4K](../papers/multiview-geometry/2026-human4k.md) — 高分辨率 multi-view mocap benchmark。
- [LAMP](../papers/multiview-geometry/2026-lamp.md) — metric 3D world 中的 multi-camera localization/tracking。
- [TROPHIES](../papers/multiview-geometry/2026-trophies.md) — human-camera-scene multi-view reconstruction。

## 当前共识

多视角的价值不仅来自“更多图像”，还来自不同遮挡模式、不同深度歧义以及明确的 camera geometry；融合模型需要区分可靠视角与异常视角。

## 研究空白

- calibration-free fusion 与显式 camera-aware fusion 的公平比较仍不充分。
- 同中心多透视（360 crop）与多中心真实多相机的几何性质不同，不能简单当成同一设定。
- **推断：**camera uncertainty 与 joint-wise pose confidence 的联合建模可能比单独 attention 更稳定。

## 与我的研究关系

可用于组织双视角/多透视 3D pose fusion、camera perturbation robustness，以及 calibration-based 与 calibration-free 方法的对比。

## 下一步阅读 / 实验

- 统一评估 MPJPE、PA-MPJPE、W-MPJPE 与 camera error。
- 增加 view masking、joint masking、camera noise 和 cross-layout zero-shot。
- 比较显式 R/t、learned camera latent 与无 camera 输入。
