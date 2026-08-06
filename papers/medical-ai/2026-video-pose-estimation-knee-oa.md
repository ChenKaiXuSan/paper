---
title: "Gait assessment using a 2D video-based pose estimation app in comparison to a markerless motion capture system in subjects with osteoarthritis of the knee – a pilot study"
authors: "Mathis Wegner, Madita Molt, Clint Hansen, Florian Gellhaus, Maciej J. K. Simon, Andreas Seekamp, Babak Moradi, Peter Behrendt"
venue: "Frontiers in Bioengineering and Biotechnology, Volume 14"
year: 2026
reading_date: 2026-08-06
status: read
tags:
  - medical-ai
  - clinical-gait-analysis
  - knee-osteoarthritis
  - monocular-pose-estimation
  - markerless-motion-capture
  - method-comparison
---

# Gait assessment using a 2D video-based pose estimation app in comparison to a markerless motion capture system in subjects with osteoarthritis of the knee – a pilot study

## Metadata / 基本信息

- **Authors / 作者：** Mathis Wegner, Madita Molt, Clint Hansen, Florian Gellhaus, Maciej J. K. Simon, Andreas Seekamp, Babak Moradi, Peter Behrendt
- **Venue / 期刊：** Frontiers in Bioengineering and Biotechnology, Section Biomechanics, Volume 14
- **Publication status / 论文状态：** Peer-reviewed original research article / 已同行评审的原创研究论文
- **Timeline / 时间：** Received 2026-04-30; revised 2026-06-15; accepted 2026-06-26; published 2026-07-27
- **Reading date / 阅读日期：** 2026-08-06
- **Reading status / 阅读状态：** `read`
- **Tags / 标签：** medical-ai, clinical-gait-analysis, knee-osteoarthritis, monocular-pose-estimation, markerless-motion-capture, method-comparison
- **DOI / 论文：** [10.3389/fbioe.2026.1869878](https://doi.org/10.3389/fbioe.2026.1869878) · [Full text](https://www.frontiersin.org/journals/bioengineering-and-biotechnology/articles/10.3389/fbioe.2026.1869878/full) · [PDF](https://www.frontiersin.org/journals/bioengineering-and-biotechnology/articles/10.3389/fbioe.2026.1869878/pdf)
- **Code / 代码：** Not publicly released / 未见公开代码；2D-PE 为商业闭源应用
- **Data / 数据：** Raw data available from the authors upon reasonable request / 原始数据可向作者合理申请
- **Systems / 系统：** Orthelligent® VISION v2.0.2; Qualisys Miqus Video + Theia3D Axiom v2025.2.2
- **Funding / 资助：** The authors reported no financial support for the study or publication / 作者声明无研究或发表资助
- **Conflict of interest / 利益冲突：** The authors declared no commercial or financial conflicts / 作者声明无商业或财务利益冲突

## Takeaway (English)

In 30 people with knee osteoarthritis walking on a treadmill, a CE-registered monocular pose-estimation app agreed well with a 12-camera markerless system for speed, cadence, step length, and relative knee-waveform shape, but not for precise absolute joint angles or several temporal, hip, and ankle measures.

## 一句话总结（中文）

针对单目临床步态工具缺少病理人群验证的问题，作者在 30 名膝骨关节炎患者中同步比较 Orthelligent VISION 与 12 相机 Theia3D，发现前者可较可靠地估计速度、步频、步长及去除固定偏置后的膝关节波形，但不足以精确量化绝对关节角或稳定测量全部时间参数。

## Evidence Boundary / 证据边界

### What the authors actually did / 作者实际完成的工作

- Conducted a prospective, cross-sectional, single-session method-comparison study at one medical center.
- Simultaneously recorded treadmill gait with a monocular 2D pose-estimation app and a 12-camera markerless 3D system.
- Compared discrete parameters with ICC(2,1) and Bland–Altman analysis, and compared full gait-cycle waveforms with SPM1D.
- Separated absolute-angle agreement from waveform-shape agreement by additionally subtracting each curve's angle at initial contact.

- 在单中心开展前瞻性、横断面、单次测量的方法学比较研究。
- 使用单目 2D 姿态估计应用与 12 相机无标记 3D 系统同步记录跑台步态。
- 通过 ICC(2,1) 与 Bland–Altman 分析比较离散参数，并用 SPM1D 比较整个步态周期的连续波形。
- 额外将每条轨迹在初始接触时归零，从而把“绝对角度一致性”和“波形形状一致性”分开讨论。

### What the results directly support / 结果直接支持的结论

The evidence supports use of this specific monocular system for selected spatiotemporal outcomes and relative knee-motion monitoring under a controlled treadmill protocol. It does not establish equivalence to marker-based motion capture, overground validity, test–retest reliability, diagnostic accuracy, or clinical outcome benefit.

证据支持的是：在受控跑台协议下，这一特定单目系统可用于部分时空参数和膝关节相对运动变化的监测。研究没有证明其等同于传统标记式动捕，也没有验证地面步行、跨日重复性、疾病诊断准确率或对临床结局的改善。

## Research Question and Motivation / 研究问题与动机

Laboratory gait analysis is accurate but expensive, technically demanding, and difficult to deploy in routine clinics. Smartphone- or tablet-based monocular systems reduce hardware and workflow burden, but depth ambiguity, parallax, occlusion, smoothing, and mismatched anatomical definitions can distort kinematics. The paper therefore asks whether a low-threshold commercial 2D system provides acceptable concurrent validity against a research-grade markerless 3D system in a clinically relevant knee-osteoarthritis population.

实验室步态分析精度较高，但成本、空间、标定和专业人员要求限制了临床普及。手机或平板上的单目系统能显著降低门槛，却容易受到深度歧义、透视效应、自遮挡、时间平滑和关节中心定义差异影响。论文因此关注一个直接的临床工程问题：面向膝骨关节炎患者，低门槛商业 2D 系统与研究级 3D 无标记系统之间究竟在哪些指标上足够一致、又在哪些指标上不能互换。

## Study Design and Cohort / 研究设计与人群

- **Design / 设计：** prospective observational cross-sectional method comparison / 前瞻性观察性横断面方法比较
- **Site / 中心：** University Medical Center Kiel, Germany / 德国基尔大学医学中心
- **Collection period / 采集时间：** March–October 2025
- **Participants / 受试者：** 30; 13 men and 17 women; age 64.5 ± 7.1 years
- **Clinical condition / 临床条件：** clinically or radiologically confirmed anteromedial knee osteoarthritis, Kellgren–Lawrence grade ≥2; no neurological gait disorder
- **Recruitment context / 招募场景：** preoperative assessment, postoperative follow-up after unicompartmental knee arthroplasty, or initial conservative-treatment evaluation
- **Control group / 对照组：** none; the aim was simultaneous system comparison rather than disease-control discrimination
- **Sample-size planning / 样本量：** a priori calculation suggested 26; 30 were recruited to tolerate dropout and stabilize error estimates
- **Ethics / 伦理：** University Medical Center ethics approval D 536/21; written informed consent

该样本具有真实临床相关性，但疾病阶段和治疗状态可能混合，且没有健康对照。左右肢体在统计中分别分析，而单目相机固定在跑台左侧，因此视角条件本身并不对称。

## Acquisition Protocol / 采集协议

### Reference system / 参照系统

- 12 Miqus Video cameras with Qualisys capture and Theia3D Axiom v2025.2.2.
- Session calibration plus a static calibration trial.
- Kinematic acquisition at 100 Hz, followed by fourth-order zero-lag Butterworth low-pass filtering.

### Monocular system / 单目系统

- Orthelligent VISION v2.0.2, a CE-registered musculoskeletal-therapy application using a proprietary cloud algorithm.
- An 11th-generation 10.9-inch iPad was mounted on a tripod, orthogonal to the treadmill and on its left side.
- No special clothing was required; participants removed their shoes.
- Videos were visually inspected, and technically faulty trials were repeated.

### Walking protocol / 步行协议

- Treadmill speed started at 0.5 m/s and was increased to each participant's comfortable self-selected speed.
- Participants received 2–3 minutes of familiarization.
- The 3D kinematic recording lasted 20 s; the app automatically ended at 30 s.
- Each limb contributed a median of 28 valid steady-state gait cycles (range 18–35).
- Start/stop cycles, acceleration periods, and visible tracking artifacts were excluded.

## Outputs and Statistical Analysis / 输出与统计分析

### Outputs / 输出

- **Spatiotemporal:** cadence, walking speed, left/right step length, double-stance phase, step time, and stance time.
- **Kinematic:** sagittal hip, knee, and ankle angles for both limbs; initial-contact angle, toe-off angle, peak extension, peak flexion, and range of motion.
- **Waveforms:** each gait cycle was normalized to 101 points from 0% to 100%, then ensemble-averaged for each participant and limb.

### Analysis / 分析

- **ICC(2,1):** two-way random-effects, absolute-agreement, single-measure ICC with 95% confidence intervals.
- **Bland–Altman:** mean bias and 95% limits of agreement, with predefined clinical bands.
- **SPM1D:** paired two-tailed tests over the entire waveform at α = 0.05 using random-field-theory thresholds.
- **Offset handling:** raw absolute angles were first evaluated directly; a second representation subtracted the initial-contact value from the full curve. This normalization changes the question from absolute-angle validity to relative waveform-shape similarity.

这里最值得借鉴的是“离散参数 + 全周期波形 + 绝对一致性”的多层验证。只报告相关系数可能掩盖系统性偏置，只报告平均角度又可能遗漏局部步态相位差异。需要注意，论文称 0° 归一化主要用于波形图形比较，因此归一化后的结果不能被解读为系统具备准确的绝对角度测量能力。

## Main Results / 主要结果

### Spatiotemporal parameters / 时空参数

| Parameter | 2D-PE | 3D-MoCap | ICC(2,1) [95% CI] | Bias | 95% LoA |
| --- | ---: | ---: | ---: | ---: | ---: |
| Cadence (steps/min) | 85.74 ± 16.12 | 91.00 ± 12.00 | 0.736 [0.553, 0.852] | −4 | −27 to 20 |
| Speed (km/h) | 2.10 ± 0.90 | 1.95 ± 0.80 | 0.949 [0.896, 0.974] | 0.15 | −0.4 to 0.7 |
| Step length, left (cm) | 39.10 ± 12.70 | 35.30 ± 12.10 | 0.912 [0.772, 0.960] | 3.8 | −5.4 to 12.9 |
| Step length, right (cm) | 27.20 ± 13.70 | 24.50 ± 12.60 | 0.937 [0.881, 0.967] | 2.7 | −5.8 to 11.2 |
| Double stance, left (% gait cycle) | 27.80 ± 9.00 | 25.40 ± 8.30 | 0.617 [0.380, 0.779] | 2.4 | −16.3 to 21.0 |
| Double stance, right (% gait cycle) | 29.00 ± 11.70 | 23.30 ± 7.90 | 0.407 [0.097, 0.643] | 5.7 | −14.1 to 25.5 |
| Step time, left/right (s) | 0.70 / 0.76 | 0.60 / 0.60 | approximately 0 | 0.1 / 0.2 | −0.2–0.3 / −0.5–0.8 |
| Stance time, left/right (% gait cycle) | 65.49 / 65.42 | 68.60 / 70.60 | approximately 0 | −4.9 / −6.2 | −16.9–7.2 / −19.8–7.5 |

Speed and step length showed excellent ICCs, while cadence was moderate-to-good. Step and stance time had near-zero ICCs. The authors attribute part of this to restricted between-subject variability, but the wide limits of agreement still argue against treating these temporal outputs as interchangeable.

速度和步长的一致性最高，步频次之；步时间和站立时间的 ICC 接近 0。虽然受试者间取值范围窄会压低 ICC，但较宽的一致性界限意味着这些时间参数仍不宜被当作可以互换的测量结果。

### Joint kinematics after initial-contact normalization / 初始接触归一化后的关节运动学

- **Knee:** most reliable joint. Left initial-contact ICC was 0.738; left range-of-motion ICC was 0.669. Right initial-contact, toe-off, and range-of-motion ICCs were 0.654, 0.625, and 0.567.
- **Hip:** range-of-motion reliability was good (left 0.699, right 0.716), but initial-contact and toe-off angles had negative ICCs.
- **Ankle:** mostly low or unstable reliability. The best result was right peak extension, ICC 0.579; several coefficients were negative or close to zero.
- **SPM1D:** hip waveforms showed no significant clusters. Differences were localized to right-ankle early stance (~0%–20%), left-ankle swing (~60%–95%), right-knee mid-stance (~20%–35%), and left-knee terminal swing (~65%–85%).

The knee waveforms captured the broad extension-in-stance and flexion-in-swing pattern. However, peak swing flexion differed by side and timing: the right knee was about 32° for 3D-MoCap versus 28° for 2D-PE, while the left relationship reversed at roughly 35° versus 30°; 3D-MoCap peaked around 80% of the gait cycle versus about 75% for 2D-PE.

膝关节是表现最稳定的关节，髋关节只有活动范围相对可靠，踝关节结果整体较弱。SPM1D 显示差异集中在少数步态阶段，而不是贯穿整个周期。不过“差异持续时间短”不等于“可以忽略”，因为早期站立和末端摆动恰好可能是临床关注的相位。

### Absolute-angle bias / 绝对角度偏置

Before normalization, the 2D system systematically overestimated knee flexion. At initial contact it produced approximately 15°–16° versus 12°–13° for 3D-MoCap; during mid-stance it remained around 10°–13° while the 3D reference fell to about 5°–7°. Without recognizing this offset, a clinician could incorrectly infer a mild flexion contracture or persistent crouch gait.

归一化前，2D 系统存在持续的膝屈曲高估。这个结果是论文最重要的安全边界之一：波形趋势可能正确，但绝对零点并不可靠。如果直接把原始角度用于诊断，可能把系统偏置误判为患者的屈曲挛缩或蹲伏步态。

## Why the Errors Occur / 误差来源解释

- **Parallax and out-of-plane motion:** gait is not strictly sagittal; rotation, abduction, and adduction distort a single 2D projection.
- **Depth ambiguity:** a monocular model infers depth statistically, while a multi-camera system triangulates it geometrically.
- **Self-occlusion:** the contralateral limb can occlude the measured limb, especially during swing.
- **Temporal smoothing:** jitter suppression can attenuate small, rapid motions and shift peak timing.
- **Joint-center and skeletal-model definitions:** systematic differences in the estimated anatomical axis create constant angle offsets.
- **View asymmetry:** the left-side camera gave the nearer limb a more favorable view than the contralateral limb.

## Strengths / 优点

- Simultaneous acquisition minimizes changes in gait between the two systems.
- The cohort is clinically relevant rather than limited to healthy young adults.
- A priori sample-size planning was performed.
- The analysis combines ICC, Bland–Altman, and SPM1D instead of relying on one agreement statistic.
- The paper explicitly separates relative waveform shape from absolute-angle measurement.
- It reports system-specific failures, including near-zero and negative ICCs, rather than presenting only successful outcomes.

## Limitations / 局限

- **Small, single-center pilot:** 30 participants are adequate for the planned comparison but insufficient for broad subgroup or deployment claims.
- **Indirect reference:** Theia3D is a validated markerless system, not the historical marker-based gold standard; the study measures agreement between two markerless technologies rather than absolute biomechanical truth.
- **Controlled treadmill setting:** no overground, stairs, turning, sit-to-stand, or home/clinic free-living tasks were tested.
- **No repeated-session reliability:** only concurrent validity in one session was assessed.
- **Fixed unilateral camera:** the left-side setup creates side-dependent visibility and occlusion.
- **Proprietary model:** the 2D algorithm, training distribution, failure-detection logic, and calibration procedure cannot be independently inspected.
- **Preprocessing exclusions:** visibly faulty trials could be repeated or excluded, so real-world failure rates may be underestimated.
- **No external site or device replication:** transportability across hospitals, tablet cameras, lighting, clothing, and operators remains unknown.
- **Reference mismatch in recording duration:** the systems recorded 20 s and 30 s respectively; the paper does not fully detail frame-level temporal synchronization and cycle matching.

## Personal Assessment / 个人评价

The paper is most convincing as a measurement-validity study, not as evidence of clinical effectiveness. Its strongest contribution is the precise identification of which outputs survive the transition from a laboratory-grade multi-camera setup to a monocular application. The combination of good step-length/speed agreement, reasonable relative knee-waveform similarity, and poor absolute-angle reliability gives a practical deployment rule: use the app for screening and within-person trend monitoring, but keep precision biomechanics and diagnostic thresholds tied to a calibrated 3D reference.

这项工作的价值不在于提出新网络，而在于建立了清楚的“可用范围”。对于低成本临床视频系统，最合理的输出不是一个看似精确的绝对关节角，而是带有质量控制的时空参数与相对波形变化。如果后续研究以临床步态分析为目标，建议把系统性偏置、视角、遮挡和不同关节的可靠性作为模型输出的一部分，而不是在汇总指标中平均掉。

## Relevance to My Research / 与我的研究关联

1. **Clinical gait validation:** The paper provides a reusable validation template for video-based clinical motion analysis: simultaneous acquisition, pathological cohort, discrete agreement, full-waveform statistics, and explicit clinical tolerances.
2. **Representation target:** Relative waveform shape may be a more robust learning target than absolute angles when camera geometry and skeletal definitions vary.
3. **Multimodal fusion:** A future model could combine RGB, optical flow, 2D keypoints, treadmill pressure events, and clinical priors to improve event timing and detect unreliable phases.
4. **Explainability:** Phase-localized SPM1D differences are more actionable than one global error score and could be paired with attention or uncertainty maps.
5. **3D reconstruction:** The monocular failure modes motivate multiview or camera-aware 3D reconstruction, especially for ankle motion, depth, and out-of-plane rotations.
6. **Suggested reproduction:** First reproduce ICC(2,1), Bland–Altman, and SPM1D on a small synchronized dataset; then test whether viewpoint normalization, patient-specific calibration, or uncertainty filtering reduces absolute-angle bias.

## Practical Reading Guide / 建议精读位置

- **Methods 2.3–2.6:** acquisition geometry, cycle processing, ICC/Bland–Altman/SPM1D design.
- **Table 1:** parameter-level agreement and limits of agreement.
- **Table 2:** joint- and side-specific reliability after normalization.
- **Figures 3–4:** waveform shape and phase-localized differences.
- **Discussion:** absolute knee-flexion offset, monocular failure mechanisms, and clinical interpretation.

## Follow-up Questions / 后续问题

- Would a front-oblique or dual-view camera setup reduce the left/right asymmetry without losing clinical simplicity?
- Can a model estimate and report per-frame uncertainty, especially around loading response and terminal swing?
- How stable are patient-specific offsets across days, clothing, therapists, and camera placement?
- Does training on pathological gait reduce the depth and timing errors seen in out-of-distribution patients?
- Would pressure-derived gait events improve step-time and stance-time agreement?

## Note Provenance / 笔记来源与边界

This note was prepared with AI assistance and checked against the full peer-reviewed article, including its methods, Tables 1–2, results, discussion, limitations, data statement, funding statement, and conflict-of-interest statement. Interpretations labeled as assessment or research relevance are analytical comments, not claims made or clinically validated by the authors.

本笔记由 AI 辅助整理，并依据同行评审全文的方法、表 1–2、结果、讨论、局限、数据声明、资助声明和利益冲突声明进行核对。文中“个人评价”和“与我的研究关联”属于分析性判断，并非作者已经完成或临床验证的结论。
