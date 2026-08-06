---
title: "Calibrated Uncertainty for Trustworthy Clinical Gait Analysis Using Probabilistic Multiview Markerless Motion Capture"
authors: "Seth Donahue, Irina Djuraskovic, Kunal Shah, Fabian Sinz, Ross Chafetz, R. James Cotton"
venue: "IEEE Transactions on Biomedical Engineering; arXiv:2601.22412"
year: 2026
reading_date: 2026-08-06
status: read
tags:
  - medical-ai
  - clinical-gait-analysis
  - multiview
  - markerless-motion-capture
  - uncertainty-calibration
  - probabilistic-modeling
  - differentiable-biomechanics
---

# Calibrated Uncertainty for Trustworthy Clinical Gait Analysis Using Probabilistic Multiview Markerless Motion Capture

## Metadata / 基本信息

- **Authors / 作者：** Seth Donahue, Irina Djuraskovic, Kunal Shah, Fabian Sinz, Ross Chafetz, R. James Cotton
- **Venue / 期刊：** IEEE Transactions on Biomedical Engineering, online ahead of print
- **Publication status / 论文状态：** Peer-reviewed journal article; author preprint available / 已同行评审期刊论文；有作者预印本
- **Timeline / 时间：** arXiv v1 2026-01-29; arXiv v2 2026-05-29; IEEE online publication 2026-05-07
- **Reading date / 阅读日期：** 2026-08-06
- **Reading status / 阅读状态：** `read`
- **Tags / 标签：** medical-ai, clinical-gait-analysis, multiview, markerless-motion-capture, uncertainty-calibration, probabilistic-modeling, differentiable-biomechanics
- **DOI / 正式论文：** [10.1109/TBME.2026.3691128](https://doi.org/10.1109/TBME.2026.3691128) · [PubMed record](https://pubmed.ncbi.nlm.nih.gov/42096387/)
- **Preprint / 预印本：** [arXiv abstract](https://arxiv.org/abs/2601.22412) · [PDF](https://arxiv.org/pdf/2601.22412)
- **Code / 代码：** The probabilistic reconstruction code was not linked in the paper / 论文未提供概率重建代码；[MultiCameraTracking](https://github.com/IntelligentSensingAndRehabilitation/MultiCameraTracking) is cited for acquisition software, not as the full method implementation
- **Data / 数据：** No public participant-level dataset was linked / 未见受试者级数据公开链接
- **Funding / 资助：** NIDILRR 90REGE0030, NIH/NICHD/NCMRR R01HD114776, and ERC Horizon Europe grant 101171526
- **Conflict of interest / 利益冲突：** No dedicated conflict-of-interest statement was visible in arXiv v2; institutional and grant support is disclosed / arXiv v2 未见单独利益冲突声明，机构与项目资助已披露

## Takeaway (English)

Across 68 participants at two institutions, a variational, differentiable multiview biomechanics pipeline produced accurate gait measures and mostly calibrated per-instance uncertainty, enabling noisy outputs to be flagged, but joint-angle performance required participant-specific bias correction and pelvis rotation remained miscalibrated.

## 一句话总结（中文）

针对多视角无标记步态系统“平均误差可接受、却无法判断单次结果是否可信”的缺口，作者将隐式关节轨迹、可微生物力学与变分推断结合，在两中心 68 名受试者上获得大多低于 0.1 的校准误差并用不确定性识别高误差样本，但关节角结果依赖个体级偏置校正，骨盆旋转仍明显过度自信。

## Evidence Boundary / 证据边界

### What the authors actually did / 作者实际完成的工作

- Externally compared probabilistic multiview markerless outputs with an instrumented GaitRite walkway and a 12-camera Vicon marker-based system.
- Evaluated two different cohorts at two institutions: one primarily for step/stride length and one for lower-extremity kinematics.
- Used a differentiable MuJoCo biomechanics model and an implicit neural trajectory representation to output both the posterior mean and covariance of joint angles over time.
- Quantified probabilistic calibration with expected calibration error (ECE) and tested whether larger predicted uncertainty corresponded to larger observed error.
- Demonstrated an uncertainty-based filtering analysis, including the lowest 50% uncertainty and uncertainty extrema.

### What the results directly support / 结果直接支持的结论

The study supports the claim that the model's uncertainty is informative and mostly globally calibrated in these laboratory datasets. It does not prove that uncertainty remains calibrated in routine clinics, that filtering improves patient outcomes, or that raw joint angles are interchangeable with clinical marker-based measurements. The latter required participant-wise offset removal.

研究直接支持的是：在当前实验室数据中，模型的不确定性与真实误差相关，而且多数指标的全局校准较好。研究没有证明这种校准能无条件迁移到繁忙临床环境，也没有证明筛除高不确定性结果会改善患者结局；未经个体级偏置校正的关节角更不能直接与临床标记式动捕互换。

## Research Question and Motivation / 研究问题与动机

Most markerless motion-capture validation reports aggregate error across many trials. A low average error does not tell a clinician whether one particular patient, joint, frame, or trial is reliable. Failures caused by occlusion, poor camera geometry, assistive devices, therapist interference, or atypical anatomy can therefore be hidden by population means. The study asks whether a probabilistic reconstruction can satisfy both conventional accuracy and calibration: an X% confidence interval should contain the clinical reference approximately X% of the time.

多数无标记动捕研究只报告跨样本平均误差。即使总体准确，临床人员仍不知道某个患者、某个关节或某一帧是否可信。遮挡、相机几何、辅具、治疗师进入画面以及非典型解剖结构造成的失败，很容易被总体均值掩盖。作者因此把验证目标扩展为两部分：预测本身要准确，置信区间也要校准，即名义上的 X% 区间应当覆盖大约 X% 的临床参照值。

## Cohorts and Reference Systems / 队列与参照系统

### Shirley Ryan AbilityLab cohort / Shirley Ryan 队列

- **Participants:** 41; 416 trials.
- **Composition:** 9 able-bodied, 10 lower-limb prosthesis users, 18 people with neurologic gait impairments, and 4 pediatric participants.
- **Adults:** age 46.73 ± 19.0 years, height 170.0 ± 11.9 cm, weight 77.5 ± 24.8 kg.
- **Pediatric subgroup:** age 13.3 ± 3.6 years, height 155.9 ± 22.1 cm, weight 54.7 ± 18.0 kg.
- **Clinical complexity:** variable speeds, orthoses, canes, crutches, walkers, and occasional therapist assistance.
- **Reference:** GaitRite instrumented walkway for step and stride length.
- **Video:** 8–12 synchronized FLIR BlackFly S GigE cameras at 30 Hz.

### Shriners Children's cohort / Shriners 儿童医院队列

- **Participants:** 27; 133 walking trials.
- **Age and anthropometrics:** age 11.3 ± 3.3 years, height 137.7 ± 20.6 cm, weight 45.0 ± 26.4 kg.
- **Diagnoses:** diverse orthopedic and neurologic conditions; cerebral palsy was most common (n = 12).
- **Assistive devices:** forearm crutches and posterior walkers were represented; 24 participants used no assistance.
- **Reference:** 12-camera Vicon Vantage at 60 Hz with the Shriners Children's Gait Model, reflective markers, static calibration, manual labeling/gap filling, and 10-Hz low-pass filtering.
- **Video:** 8 synchronized and calibrated FLIR BlackFly S USB3 cameras at 60 Hz.

The total N is 68, but the two validation questions are distributed across cohorts rather than both references being available for every participant. This distinction matters when interpreting “two-center external validation.”

总样本量为 68，但两个队列承担的验证任务不同：一个主要验证步长/跨步长，另一个验证关节运动学，并非每名受试者都同时拥有两类金标准。这使研究具备跨中心证据，却不是同一任务在两个中心的完全重复验证。

## Pipeline / 方法流程

### 1. Multiview keypoint preprocessing / 多视角关键点预处理

- Images were processed with MetrABs-ACAE.
- The pipeline retained 87 keypoints from the MoVi keypoint set; dense torso coverage was intended to stabilize trunk estimation.
- EasyMocap was used for scene reconstruction and segmentation.
- A custom visualization tool was used to annotate the target participant.

### 2. Differentiable biomechanical model / 可微生物力学模型

- A GPU-accelerated MuJoCo model represented approximately 40 kinematic joint-angle variables.
- Anatomical priors included fixed or optimized segment properties, joint coupling, feasible joint limits, and site-offset regularization.
- The model reprojected virtual 3D markers into every camera and optimized the motion end-to-end against detected 2D keypoints.

### 3. Implicit probabilistic trajectory / 隐式概率轨迹

An MLP maps continuous time to a Gaussian joint-angle distribution rather than a single pose:

`f_phi(t) -> (mu_phi(t), u_phi(t)), q_phi(theta_t) = Normal(mu_phi(t), Sigma_phi(t))`

The output is a time-varying posterior mean and a low-rank covariance. The paper used hidden dimensions [128, 256, 512, 1024] and covariance rank 20. This representation models correlations among joint angles and provides a standard deviation for every joint and time point.

模型不是逐帧只输出一个确定角度，而是用 MLP 将连续时间映射到关节角高斯分布，得到每个时刻的均值、标准差及关节间相关结构。这个设计把轨迹平滑、解剖约束和不确定性传播统一在同一优化框架中。

### 4. Variational inference and objective / 变分推断与目标函数

Because the exact posterior is intractable and the 2D detector does not provide a complete likelihood, the authors approximate it with variational inference. The composite objective includes:

- negative log-likelihood of detected 2D keypoints;
- entropy of the pose posterior, forming the core evidence lower bound;
- site-offset regularization;
- joint-limit penalties and bounded outputs;
- temporal smoothness;
- robust multi-camera weighting and confidence masking;
- an ECE regularizer based on the highest-confidence detected keypoints as an internal pseudo-reference.

Five Shriners participants were used to tune the ECE weight from 0.0 to 1.0; 0.5 was selected and applied to the other 63 participants. The kinematic model annealed this loss after the first half of optimization. A five-trial session took approximately 50 minutes on a 48-GB GPU, so the current implementation is offline rather than real-time.

### 5. Calibration metric / 校准指标

For step and stride length, 1,000 samples from the joint-angle posterior were propagated through forward kinematics to obtain a predictive error distribution. Probability integral transform values were compared with a uniform distribution, and ECE measured their average deviation. For Gaussian joint-angle posteriors, errors were transformed with a half-normal CDF. Lower ECE is better; a perfect calibration curve follows the identity line.

## Main Results / 主要结果

### Spatial gait calibration / 空间步态参数校准

| Group | Step-length ECE | Stride-length ECE |
| --- | ---: | ---: |
| All participants | 0.05 | 0.04 |
| Able-bodied controls | 0.08 | 0.07 |
| Neurologic gait | 0.10 | 0.03 |
| Pediatric gait | 0.03 | 0.03 |
| Prosthetic gait | 0.08 | 0.04 |
| Slow / self-selected / fast | 0.02 / 0.02 / 0.07 | 0.02 / 0.02 / 0.04 |

The model remained at or below ECE 0.10 for all listed subgroups and speeds. This is stronger evidence than aggregate accuracy alone because it tests whether the predicted uncertainty distribution matches observed error frequency.

### Spatial accuracy and uncertainty filtering / 空间精度与不确定性筛选

| Selection | Step error, median (IQR), mm | Stride error, median (IQR), mm |
| --- | ---: | ---: |
| All data | 16.24 (25.60) | 11.83 (17.65) |
| Lowest 50% predicted uncertainty | 12.01 (17.59) | 9.13 (12.56) |
| Noisiest 10% | 38.73 (55.45) | 30.82 (47.94) |
| Most confident 10% | 9.85 (12.31) | 7.31 (8.47) |

The uncertainty ranking is clinically meaningful: bins with greater predicted uncertainty had greater reference error, and filtering the least confident half reduced the median error and spread. The neurologic group had the largest uncertainty and error, plausibly reflecting therapist presence, assistive devices, and more complex gait.

不确定性不是装饰性输出：模型认为“最不确定”的 10% 样本确实包含明显更大的误差，而只保留较低不确定性的 50% 后，误差中位数与离散程度均下降。不过这仍是回顾性筛选分析；论文没有评估因筛除数据而丢失的临床信息，也没有预先定义用于实际决策的固定阈值。

### Systematic kinematic bias / 运动学系统偏置

Before bias correction, substantial offsets existed between the MuJoCo model and the clinical marker-based model:

| Joint variable | Mean bias ± SD |
| --- | ---: |
| Pelvis tilt | 19.71° ± 4.91° |
| Pelvis obliquity | −0.22° ± 2.75° |
| Pelvis rotation | 0.76° ± 4.27° |
| Hip flexion | 23.25° ± 6.36° |
| Hip abduction | 4.56° ± 1.84° |
| Hip rotation | 2.19° ± 10.29° |
| Knee flexion | 3.19° ± 4.60° |
| Ankle flexion | 2.73° ± 6.46° |

The authors removed a constant participant-specific bias for each joint before reporting trajectory errors. Therefore, the small final kinematic errors quantify tracking fidelity after alignment of anatomical conventions, not raw interchangeability between systems.

偏置校正前，骨盆倾斜和髋屈曲的均值差分别达到约 19.7° 和 23.3°。作者按受试者、按关节估计固定偏置后再比较波形。因此，后续 1.5°–3.8° 的误差代表“去除个体零点差异后的轨迹跟踪能力”，不能表述为系统无需标定即可输出临床绝对角度。

### Kinematic calibration after bias correction / 偏置校正后的运动学校准

| Joint | ECE | Predicted uncertainty, median (IQR) | Absolute error, median (IQR) |
| --- | ---: | ---: | ---: |
| Pelvis tilt | 0.07 | 3.28° (0.76°) | 1.62° (0.81°) |
| Pelvis obliquity | 0.06 | 3.23° (0.67°) | 1.52° (1.27°) |
| Pelvis rotation | **0.18** | 1.99° (0.33°) | 2.15° (1.34°) |
| Hip flexion | 0.01 | 4.26° (0.99°) | 2.53° (1.13°) |
| Hip abduction | 0.05 | 3.74° (0.87°) | 1.88° (1.55°) |
| Hip rotation | 0.08 | 4.78° (1.20°) | 3.76° (1.55°) |
| Knee flexion | 0.06 | 3.56° (0.90°) | 2.84° (1.22°) |
| Ankle flexion | 0.07 | 4.73° (1.31°) | 3.52° (1.04°) |

Seven of eight kinematic variables had ECE below 0.10. Pelvis rotation was the clear failure case: ECE 0.18, with only about 70% of marker-based samples falling inside the model's nominal 95% interval. The authors attribute this partly to different Euler-angle application orders in MuJoCo and the Shriners clinical gait model.

## Strengths / 优点

- It validates uncertainty against external clinical instruments, extending earlier internal validation against high-confidence keypoints.
- The participants include neurologic, prosthetic, pediatric, and assistive-device users rather than only healthy adults.
- Both accuracy and calibration are reported, and the paper includes a concrete failure case instead of hiding miscalibration.
- Uncertainty is propagated through a biomechanical model to clinically interpretable variables, not limited to detector heatmaps or 2D keypoint confidence.
- The analysis tests whether uncertainty ranks errors, which is directly relevant to automated quality control.
- Hyperparameters, covariance rank, sampling count, optimization length, and processing time are disclosed in the appendix.

## Limitations / 局限

- **Laboratory-only validation:** large open spaces, designated 10-m walkways, synchronized cameras, and mostly unobstructed coverage do not represent busy therapy gyms or privacy-constrained clinics.
- **Participant-specific bias correction:** raw kinematic offsets can be large, and a clinical reference is needed to estimate the correction used in this paper.
- **Different validation tasks across sites:** GaitRite spatial metrics and Vicon kinematics were not both evaluated for all 68 participants.
- **Limited camera stress testing:** the kinematic cohort used eight optimized views; the paper does not provide systematic camera-count, placement, desynchronization, or occlusion ablations.
- **Global calibration metric:** ECE can hide conditional failures by gait phase, joint-angle magnitude, velocity, diagnosis, device use, or demographic subgroup.
- **Calibration tuning:** five participants were used post hoc to choose the ECE weight; broader nested or prospectively locked calibration would strengthen generalization evidence.
- **Pseudo-reference inside the loss:** internal ECE regularization treats high-confidence detector keypoints as a reference, which can inherit detector bias.
- **Offline computational cost:** approximately 50 minutes for a five-trial session on a 48-GB GPU limits real-time use.
- **Reproducibility:** the complete probabilistic code, trained settings, and participant-level data are not linked publicly.
- **Clinical utility not tested:** the paper does not show whether clinicians using confidence intervals make better decisions or whether excluding uncertain trials changes outcomes.

## Personal Assessment / 个人评价

The paper's key advance is not merely lower gait error; it changes the output contract of markerless motion capture from “a trajectory” to “a trajectory plus a calibrated quality signal.” This is exactly the kind of interface needed for clinical AI, where selective prediction and abstention can be safer than forcing a number for every frame. The most convincing result is that uncertainty bins track real error and identify high-error outliers without concurrent ground truth.

这篇工作的核心创新不是把平均误差再降低一点，而是让系统同时给出轨迹和可解释的可信度信号。对临床 AI 而言，允许模型在低质量输入上“拒答”通常比每帧都强制输出角度更安全。最有说服力的证据是：不确定性排序与真实误差同步变化，并能在没有当前金标准输入的推理阶段标记潜在异常结果。

However, the participant-wise bias correction is a major boundary. The framework appears strong for trajectory shape and data-quality control after aligning model conventions, but it is not yet a drop-in replacement for clinical marker-based absolute kinematics. Pelvis rotation also demonstrates that a globally calibrated system can still fail on one clinically relevant variable.

但个体级偏置校正是一条重要边界。当前证据更支持“对齐解剖定义后的轨迹跟踪与质量控制”，而不是“直接替代临床标记式绝对运动学”。骨盆旋转的失败也说明，即使总体 ECE 良好，特定变量仍可能严重过度自信。

## Relevance to My Research / 与我的研究关联

1. **Clinical video analysis:** Add calibrated uncertainty or selective prediction to gait and posture models, especially when deployment data contain occlusion, atypical anatomy, or assistive devices.
2. **Multiview 3D reconstruction:** The implicit continuous-time trajectory and low-rank covariance offer a useful alternative to framewise point estimates.
3. **Multimodal fusion:** RGB, keypoint confidence, camera geometry, contact, and clinical priors can be fused not only to improve accuracy but also to predict when each modality is unreliable.
4. **Explainability:** Per-joint, per-time uncertainty is more clinically actionable than one sequence-level confidence score; it can be visualized over gait phases.
5. **Evaluation design:** Report calibration curves, ECE, uncertainty–error correlation, risk–coverage curves, and performance after abstention, in addition to MPJPE or angle error.
6. **Spine and posture applications:** The same framework could propagate uncertainty from multiview keypoints into spinal alignment or posture parameters, provided anatomical-model bias is handled explicitly.
7. **Suggested reproduction:** Begin with a deterministic differentiable biomechanical pipeline, add mean/variance trajectory heads, validate posterior calibration against held-out marker-based data, and stress-test camera count, occlusion, synchronization, and out-of-distribution pathology.

## Practical Reading Guide / 建议精读位置

- **Sections 1.2–1.3:** definition of calibration and the distinction between aggregate accuracy and per-instance trust.
- **Sections 2.4–2.5:** multiview preprocessing, variational trajectory representation, loss, and hyperparameter selection.
- **Section 2.5.1:** PIT-based ECE calculation for spatial and kinematic outputs.
- **Tables 1–3 and Figure 3:** subgroup calibration and uncertainty-based filtering.
- **Tables 4–5:** raw systematic bias versus post-correction calibration.
- **Section 4.3:** laboratory constraints, camera geometry, bias correction, and future conditional calibration.
- **Appendix Table 6:** implementation and optimization settings.

## Follow-up Questions / 后续问题

- Can calibration survive a prospectively locked evaluation in unseen clinics and camera layouts?
- How does risk–coverage change as the system abstains on more trials, and which diagnoses are disproportionately excluded?
- Can anatomical definitions be harmonized so that participant-specific offset correction is unnecessary?
- Is calibration maintained conditionally across gait phase, joint velocity, assistive-device use, and severe pathology?
- Would ensemble, Bayesian, or conformal approaches give more reliable out-of-distribution uncertainty than the current variational Gaussian posterior?
- Can inference be accelerated enough for same-session clinical feedback?

## Note Provenance / 笔记来源与边界

This note was prepared with AI assistance and checked against arXiv v2 of the full manuscript, its appendix, the DOI record, and the PubMed record for the peer-reviewed IEEE publication. Numerical results are taken from the manuscript's Tables 1–6. Analytical comments on transferability and clinical use are clearly separated from claims directly demonstrated by the authors.

本笔记由 AI 辅助整理，并依据 arXiv v2 全文及附录、DOI 记录和 PubMed 的 IEEE 同行评审出版记录核对。数值结果来自原文表 1–6；关于研究迁移与临床使用的判断均与作者直接验证的结论分开表述。
