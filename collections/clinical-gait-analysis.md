# Clinical Gait Analysis

## 研究问题

研究如何从视频、3D pose、运动学和多模态信号中提取具有临床意义的步态表征，并用于疾病识别、严重度评估、纵向追踪、广泛健康表型预测和可解释分析。

## 方法脉络

1. 视频 / markerless pose acquisition。
2. Pose refinement 与 biomechanically constrained 3D kinematics。
3. Kinetics / ground reaction force / joint moment / muscle-force-related variables。
4. 周期、时空和 biomechanics representation，以及 self-supervised / foundation gait embedding，用于疾病分类、严重度、longitudinal modeling 与 broad health phenotyping。
5. uncertainty、clinical knowledge 与解释性。
6. **Tool-grounded clinical reasoning：**由 VLM/agent 提出步态假设，调用可复核的 2D/3D kinematics 与 gait-event 工具验证，保留支持、反证和不确定性，再输出解释报告。

## 关键论文

- [Video pose estimation vs markerless motion capture in knee OA](../papers/medical-ai/2026-video-pose-estimation-knee-oa.md) — 临床视频姿态测量有效性。
- [Calibrated Uncertainty for Clinical Gait Analysis](../papers/medical-ai/2026-calibrated-uncertainty-clinical-gait.md) — probabilistic multiview markerless gait 与 uncertainty。
- [OpenCap Monocular](../papers/medical-ai/2026-opencap-monocular.md) — 单 smartphone video 经 pose refinement、OpenSim IK 与 physics/ML dynamics 得到临床可解释 kinematics 与 kinetics。
- [A Gait Foundation Model Predicts Multi-System Health Phenotypes from 3D Skeletal Motion](../papers/medical-ai/2026-gait-foundation-health-phenotypes.md) — 用 3,414 人、351 h 3D skeletal motion 自监督预训练 GaitMAE，并将 gait embedding 扩展到多系统健康 phenotype prediction。
- [CARE-PD](../papers/medical-ai/2025-care-pd.md) — Parkinson's gait assessment 数据资源。
- [GaitEncoder](../papers/medical-ai/2026-gaitencoder.md) — gait kinematics foundation representation。
- [BioGait-VLM](../papers/medical-ai/2026-biogait-vlm.md) — vision-language-biomechanics 临床解释框架。
- [DrGait](../papers/medical-ai/2026-drgait-biomechanically-grounded-clinical-gait.md) — training-free TVS agent：VLM 仅规划可核验的临床证据，RTMW/WHAM 与 biomechanical tools 负责定量测量；GAVD/ProGait 上考察异常亚型和可审计报告。
- [SIMSPINE](../papers/medical-ai/2026-simspine.md) — biomechanics-aware spine simulation / annotation。
- [MedVCR](../papers/medical-ai/2026-medvcr.md) — clinically-grounded counterfactual reasoning。
- [ResiHMR](../papers/medical-ai/2026-resihmr.md) — 非标准人体拓扑下的 HMR 偏差问题。

## 当前共识

临床 gait 模型的可信度不仅取决于分类精度，还取决于跨受试者划分、测量误差、外部验证、运动学可解释性以及模型对病理性形态/动作的偏差。OpenCap Monocular 说明，上游 monocular pose error 应通过物理/生物力学约束被显式修正，并且临床评价不应停留在 MPJPE：GRF、joint moment、KAM 等 kinetics 及其 clinically meaningful error threshold 能更直接检验视频方法是否具有临床价值。

Gait foundation model 的 phenome-wide 结果进一步提示，gait embedding 不应只被视为单疾病分类特征：大规模 self-supervised skeletal representation 可以携带年龄、身体组成以及多个生理/行为系统相关的信息，并显著超越传统 engineered gait features。但这种能力目前仍高度依赖特定 3D capture system 与 cohort，必须通过跨设备、跨人群与真实疾病队列验证后才能作为临床数字生物标志物使用。

## 研究空白

- 多中心、跨设备和真实 clinical deployment 的外部验证仍不足。
- DrGait 补充了 **agentic differential diagnosis + deterministic measurement** 路线，但可核验工具仍受上游 2D/3D pose 偏差制约；原文 GAVD 异常亚型整体准确率在 Table 2（53.4%）与正文/Table 7（51.4%）之间不一致，且尚缺公开官方代码及前瞻性临床验证。
- **推断：**将周期时序表示、self-supervised gait foundation embedding、3D pose uncertainty、biomechanical kinematics/kinetics 与 clinical knowledge 显式结合，可能比纯 RGB 分类或单一 engineered features 更稳健且更可解释。
- 上游姿态模型通常在健康/通用人体上预训练，可能把病理动作拉回“正常”先验；OpenCap Monocular 本身也只在 10 名年轻健康成人上验证，真实病理人群仍需要独立评估。
- 大规模 gait foundation representation 当前主要来自 Azure Kinect 3D skeleton；迁移到 monocular RGB、2D keypoints、不同 pose estimators 与不同 skeleton topology 的 domain gap 尚未解决。
- phenome-wide gait association 目前主要是 cross-sectional，不能直接解释为因果或临床 screening validity；仍需 longitudinal / prospective validation。
- 单目 biomechanics 对 camera viewpoint、out-of-plane motion、已知 intrinsics / height 和 static-camera assumption 仍敏感。

## 与我的研究关系

该 collection 可直接支持临床步态、脊柱疾病视频分析、周期运动建模和可解释多模态融合的 Related Work 与实验设计。适合把 `RGB/keypoints → latent classification` 扩展为两条互补路线：一条是 `video → pose → constrained kinematics/kinetics → diagnosis`，另一条是 `large-scale pose sequence → self-supervised foundation embedding → disease / severity / phenotype prediction`，最终再与 biomechanics variables、RGB/flow 和 clinical knowledge 融合。

## 下一步阅读 / 实验

- 比较 engineered gait features、task-specific pose encoder、masked self-supervised gait embedding、以及 embedding + biomechanics variables 的性能。
- 在 masked reconstruction 中增加 velocity / phase / periodicity objective，测试其对病理 gait representation 的贡献。
- 增加 pose-only、pose + constrained kinematics、pose + kinetics、foundation embedding 与 multimodal fusion 的递进实验。
- 强制 subject-disjoint / site-disjoint protocol，并增加真实病理 cohort、跨 camera/viewpoint、跨 pose estimator 与跨 skeleton topology 测试。
- 研究 Azure Kinect 3D skeleton representation 向 monocular RGB / 2D pose 的 distillation 或 domain adaptation。
- 同时报告 discrimination、calibration、uncertainty、kinematic/kinetic error、external validity 和 clinical interpretability。
- **推断：**在 ASD/临床步态视频上额外比较 class-only、attention-map explanation、clinical-feature report 与 DrGait-style tool-verified VLM report；对关键点噪声、摄像机视角与测量冲突做压力测试，统计错误定量解释率、医生盲评及拒答/不确定性质量。
