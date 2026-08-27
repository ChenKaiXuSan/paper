# Clinical Gait Analysis

## 研究问题

研究如何从视频、3D pose、运动学和多模态信号中提取具有临床意义的步态表征，并用于疾病识别、严重度评估、纵向追踪和可解释分析。

## 方法脉络

1. 视频 / markerless pose acquisition。
2. Pose refinement 与 biomechanically constrained 3D kinematics。
3. Kinetics / ground reaction force / joint moment / muscle-force-related variables。
4. 周期、时空和 biomechanics representation，用于疾病分类、严重度与 longitudinal modeling。
5. uncertainty、clinical knowledge 与解释性。

## 关键论文

- [Video pose estimation vs markerless motion capture in knee OA](../papers/medical-ai/2026-video-pose-estimation-knee-oa.md) — 临床视频姿态测量有效性。
- [Calibrated Uncertainty for Clinical Gait Analysis](../papers/medical-ai/2026-calibrated-uncertainty-clinical-gait.md) — probabilistic multiview markerless gait 与 uncertainty。
- [OpenCap Monocular](../papers/medical-ai/2026-opencap-monocular.md) — 单 smartphone video 经 pose refinement、OpenSim IK 与 physics/ML dynamics 得到临床可解释 kinematics 与 kinetics。
- [CARE-PD](../papers/medical-ai/2025-care-pd.md) — Parkinson's gait assessment 数据资源。
- [GaitEncoder](../papers/medical-ai/2026-gaitencoder.md) — gait kinematics foundation representation。
- [BioGait-VLM](../papers/medical-ai/2026-biogait-vlm.md) — vision-language-biomechanics 临床解释框架。
- [SIMSPINE](../papers/medical-ai/2026-simspine.md) — biomechanics-aware spine simulation / annotation。
- [MedVCR](../papers/medical-ai/2026-medvcr.md) — clinically-grounded counterfactual reasoning。
- [ResiHMR](../papers/medical-ai/2026-resihmr.md) — 非标准人体拓扑下的 HMR 偏差问题。

## 当前共识

临床 gait 模型的可信度不仅取决于分类精度，还取决于跨受试者划分、测量误差、外部验证、运动学可解释性以及模型对病理性形态/动作的偏差。OpenCap Monocular 进一步说明，上游 monocular pose error 应通过物理/生物力学约束被显式修正，并且临床评价不应停留在 MPJPE：GRF、joint moment、KAM 等 kinetics 及其 clinically meaningful error threshold 能更直接检验视频方法是否具有临床价值。

## 研究空白

- 多中心、跨设备和真实 clinical deployment 的外部验证仍不足。
- **推断：**将周期时序表示、3D pose uncertainty、biomechanical kinematics/kinetics 与 clinical knowledge 显式结合，可能比纯 RGB 分类更稳健且更可解释。
- 上游姿态模型通常在健康/通用人体上预训练，可能把病理动作拉回“正常”先验；OpenCap Monocular 本身也只在 10 名年轻健康成人上验证，真实病理人群仍需要独立评估。
- 单目 biomechanics 对 camera viewpoint、out-of-plane motion、已知 intrinsics / height 和 static-camera assumption 仍敏感。

## 与我的研究关系

该 collection 可直接支持临床步态、脊柱疾病视频分析、周期运动建模和可解释多模态融合的 Related Work 与实验设计。特别适合把 `RGB/keypoints → latent classification` 扩展为 `video → pose → constrained kinematics/kinetics → diagnosis`，并把 biomechanical variables 作为辅助监督或解释层。

## 下一步阅读 / 实验

- 比较 RGB、keypoints、optical flow、biomechanical variables 与 gait latent。
- 增加 pose-only、pose + constrained kinematics、pose + kinetics、multimodal fusion 的递进实验。
- 强制 subject-disjoint / site-disjoint protocol，并增加真实病理 cohort 与跨 camera/viewpoint 测试。
- 同时报告 discrimination、calibration、uncertainty、kinematic/kinetic error 和 clinical interpretability。
