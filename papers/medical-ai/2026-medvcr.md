---
title: "Clinically-Grounded Counterfactual Reasoning for Medical Video Diagnosis"
authors: "Jianzhe Gao, Churan Wang, Weiyi Zhang, Jianghua Li, Li-An Li, Wenguan Wang, Yixin Zhu, Yizhou Wang"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-24
status: skimmed
tags:
  - medical-ai
  - medical-video
  - counterfactual-reasoning
  - clinical-knowledge
  - interpretable-ai
  - temporal-modeling
---

# Clinically-Grounded Counterfactual Reasoning for Medical Video Diagnosis

## 基本信息

- **作者：** Jianzhe Gao, Churan Wang, Weiyi Zhang, Jianghua Li, Li-An Li, Wenguan Wang, Yixin Zhu, Yizhou Wang
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-24
- **阅读状态：** `skimmed`
- **标签：** `medical-ai`, `medical-video`, `counterfactual-reasoning`, `clinical-knowledge`, `interpretable-ai`, `temporal-modeling`
- **论文：** [CVF Open Access](https://openaccess.thecvf.com/content/CVPR2026/html/Gao_Clinically-Grounded_Counterfactual_Reasoning_for_Medical_Video_Diagnosis_CVPR_2026_paper.html) / [arXiv:2605.26483](https://arxiv.org/abs/2605.26483)
- **代码：** 暂无；arXiv v1 标注 “The code will be released”
- **数据集：** Colposcopy 为作者院内数据（未公开）；公开实验数据包括 [Hyper-Kvasir](https://datasets.simula.no/hyper-kvasir/) 与 [LDPolypVideo](https://github.com/dashishi/LDPolypVideo-Benchmark)
- **项目主页：** [MedVCR](https://gaozzzz.github.io/MedVCR/)（截至阅读日仍为模板占位页）
- **DOI：** [10.48550/arXiv.2605.26483](https://doi.org/10.48550/arXiv.2605.26483)

## 一句话总结

MedVCR 不再只把临床视频端到端映射到诊断标签，而是显式生成“如果病理状态相反会怎样”的 counterfactual tissue evolution，并用临床规则约束表征，再将事实与反事实证据共同用于诊断。

## 研究问题与动机

许多 medical video 模型依赖外观和时序相关性，容易把 illumination、reagent color、camera motion 等非病理变化误认为诊断线索；同时模型通常不显式使用临床诊断原则，也缺少医生常用的假设比较过程。

论文把这一缺口定义为“缺少 clinically grounded、hypothesis-driven counterfactual reasoning”。其目标不是单纯提升 backbone，而是让模型学习：观察到的组织演变是否更符合某种病理假设，以及如果健康状态相反时，组织在下一阶段应当出现怎样的变化。

## 核心方法

### 1. Counterfactual Generator (CG)

CG 是 conditional diffusion model。给定当前临床阶段的图像 $x_t$ 与指定健康状态 $h\in\{benign, malignant\}$，生成下一阶段的 hypothetical frame。这样同一真实观察可以得到 benign 与 malignant 两种 counterfactual evolution。

### 2. Counterfactual Representation Learning (CRL)

Medical video learner 使用 I3D 提取局部 clip features，并用 temporal Transformer 建模跨阶段长程关系。作者再把三条 clinical rules 变成表征约束：

- **Temporal consistency：**同一病理状态在相邻检查阶段保持诊断一致性。
- **Pathological separability：**benign 与 malignant representation 应可分离。
- **Counterfactual alignment：**事实应靠近与真实病理一致的 counterfactual，并远离相反病理假设。

### 3. Dual Diagnostic Prediction (DDP)

最终诊断同时使用 video-level temporal context 与 frame-level factual/counterfactual contrast。预测形式把真实证据加到 global logits 上，同时减去相反病理 counterfactual 的 logits，从而模拟 differential diagnosis。

## 数据集与评价指标

论文验证了两种 medical video supervision：

- **完全监督 colposcopy biopsy-site localization：**院内数据共 **623 个 patient-specific examination records**，每例含 saline、acetic acid、alcohol、iodine 四阶段完整视频、关键帧和 pathology report；每例由 3 名 senior gynecologists 复核。采用 5-fold cross-validation，指标为 Recall、Precision、Accuracy、Recall@1。
- **弱监督 colonoscopy polyp-frame detection：**融合 Hyper-Kvasir 与 LDPolypVideo，超过 **1 million frames**。训练集为 **61 个无 polyp + 102 个有 polyp videos**，测试集为 **30 + 60 videos**；训练仅有 video-level label，测试使用 frame-level label，指标为 AP 与 AUC。
- Colonoscopy 输入时每段视频切为 32 个 snippet，每个 snippet 16 帧。

## 主要结果

Colposcopy 上 MedVCR 达到 **Recall 80.3%、Precision 74.4%、Accuracy 55.0%、Recall@1 93.0%**。最佳对比方法 SurgFormer 的 Recall@1 为 82.8%，因此提升 **10.2 个百分点**。

Colonoscopy 上达到 **AP 94.8%、AUC 99.6%**，高于此前最强 Fadmb 的 92.2/99.4，AP 提升 **2.6 个百分点**。

关键消融中，仅使用基本 medical video learner 时 Colposcopy Recall@1 / Colonoscopy AP 为 **77.9 / 82.8**；加入 clinical rules 后为 **89.4 / 91.6**；只加入 DDP 为 **80.2 / 85.5**；两者同时使用达到 **93.0 / 94.8**。结果说明显式 clinical rules 的贡献尤其明显。

## 优点

- 将临床知识从“附加文本说明”变成可优化的 representation constraints。
- counterfactual generator 为数据稀缺场景提供一种假设驱动的比较信号，而不是只依赖表面相关性。
- 同时验证 fully supervised 与 weakly supervised medical video setting，且包含 5-fold cross-validation。
- 消融能比较 clinical rules 与 dual prediction 的独立贡献，方法解释较清楚。
- 对“临床知识 + 时序视频 + 可解释推理”的结合提供了较完整的结构化范式。

## 局限

- 作者明确指出 CG 只学习相邻临床阶段的 short-range transition，尚未验证更长时间或跨多个阶段的 counterfactual evolution。
- 当前 clinical rules 只覆盖部分专家诊断原则，不能代表完整临床推理。
- 多中心与跨机构泛化仍不足；Colposcopy 主实验使用 623 例院内数据。
- 官方代码截至阅读日尚未发布，项目页仍为模板占位，复现条件有限。
- **推断：**counterfactual 图像是否“视觉合理”不等于其病理因果机制真实，因此临床使用时仍需要独立的医学有效性验证。

## 个人评价

这篇论文对临床动作分析最有价值的不是具体的宫颈或结肠任务，而是“临床知识应该如何进入时序网络”。相比把医生知识写成 attention mask 或 feature weighting，MedVCR 更进一步：先构造明确的替代病理假设，再要求模型比较 factual 与 counterfactual trajectory。

我认为它的关键实验启示是 clinical rules 的增益大于单独 DDP，说明临床知识若能变成稳定的训练约束，可能比仅在预测头加入可解释机制更有效。不过其生成式 counterfactual 是否满足真正 causal validity 仍需要谨慎区分。

## 与我的研究关联

对视频步态/姿态和脊柱疾病辅助诊断有明显可迁移性：

1. 可把 clinical rules 从“组织阶段变化”改成“病理状态下的运动规律”，例如 gait-cycle consistency、左右对称/不对称、躯干-骨盆耦合或周期相位稳定性。
2. 不一定要生成 RGB counterfactual；**推断：**可以在 3D keypoint、joint-angle、optical-flow 或 latent motion space 中生成“假设为正常/异常时”的 counterfactual motion，再与真实序列做对比。
3. 对小样本疾病分类，可以增加 pathological separability 与 counterfactual alignment loss，检验其是否提高跨患者、跨视角泛化。
4. 值得设计 `data-driven baseline → clinical-rule regularization → counterfactual branch → full model` 的消融，并加入 calibration / uncertainty，避免只看 Accuracy 或 AUROC。

## 后续阅读

- 临床知识引导的 gait / posture video classification。
- Counterfactual explanation 与 causal representation learning 在 medical imaging 中的工作。
- 时序 medical video foundation model 与 weakly supervised clinical video localization。
