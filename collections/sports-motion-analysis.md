# Sports Motion Analysis

## 研究问题

研究如何从视频、3D pose、惯性/压力等传感器与生物力学约束中分析体育动作，并进一步生成可量化、可解释的技术评价或反馈。

## 方法脉络

1. 运动姿态与时序表征。
2. 高速动作、遮挡与移动相机下的 3D reconstruction。
3. physics / contact / biomechanics constraints。
4. 运动技术评价、coaching 与自然语言反馈。

## 关键论文

- [GRIP](../papers/sports-biomechanics/2026-grip.md) — sparse IMU + insole pressure + physics-based human motion capture。
- [BioCoach](../papers/sports-biomechanics/2026-biocoach.md) — 3D pose 到 biomechanics-grounded coaching。
- [Action Motifs](../papers/motion-understanding/2026-action-motifs.md) — 可变长度层级运动表征。
- [EventGait](../papers/motion-understanding/2026-eventgait.md) — event streams 下的高速/低照度运动识别思路。
- [PhysDynPose](../papers/global-human-motion/2025-physics-based-human-pose-moving-camera.md) — moving camera + physics refinement。
- [EmbodMocap](../papers/multiview-geometry/2026-embodmocap.md) — 双移动相机 4D human-scene reconstruction。

## 当前共识

体育动作分析需要同时关注局部姿态精度、全局运动、接触与动力学合理性；单纯 PA-MPJPE 很难反映技术动作是否真实、稳定和可解释。

## 研究空白

- 高速户外体育缺少带 camera trajectory、world human motion 和接触/地形 GT 的统一 benchmark。
- **推断：**360° 伴随式拍摄可以提高观测连续性，但需要专门处理同中心多透视、动态 camera 和场景尺度。
- biomechanics 规则与 learned representation 如何结合、并跨运动项目泛化仍不明确。

## 与我的研究关系

用于组织滑雪、体育视频 3D motion reconstruction、physics/contact evaluation 和动作技术解释相关文献。

## 下一步阅读 / 实验

- 增加 skiing / alpine sports 专门文献与数据集。
- 报告 joint angle、root trajectory、velocity/acceleration、foot/contact 与 camera metrics。
- 探索从 3D pose / biomechanics variables 自动生成技术反馈。
