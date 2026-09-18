---
title: "Clinical-Prior Guided Multi-Modal Learning with Latent Attention Pooling for Gait-Based Scoliosis Screening"
authors: "Dong Chen, Zizhuang Wei, Jialei Xu, Xinyang Sun, Zonglin He, Meiru An, Huili Peng, Yong Hu, Kenneth MC Cheung"
venue: "arXiv:2602.06743"
year: 2026
reading_date: 2026-09-19
status: skimmed
tags:
  - medical-ai
  - clinical-gait
  - scoliosis
  - multimodal-learning
  - clinical-prior
  - explainable-ai
  - subject-disjoint
---

# Clinical-Prior Guided Multi-Modal Learning with Latent Attention Pooling for Gait-Based Scoliosis Screening

## 基本信息

- **作者：** Dong Chen, Zizhuang Wei, Jialei Xu, Xinyang Sun, Zonglin He, Meiru An, Huili Peng, Yong Hu, Kenneth MC Cheung
- **会议/期刊：** arXiv:2602.06743
- **年份：** 2026
- **提交日期：** 2026-02-06
- **阅读日期：** 2026-09-19
- **阅读状态：** `skimmed`
- **标签：** `medical-ai`, `clinical-gait`, `scoliosis`, `multimodal-learning`, `clinical-prior`, `explainable-ai`, `subject-disjoint`
- **DOI：** 10.48550/arXiv.2602.06743
- **论文：** https://arxiv.org/abs/2602.06743
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

该工作构建 subject-independent 的 ScoliGait benchmark，并把 238 维临床运动学 knowledge map、RGB gait video 与临床文本通过 latent attention pooling 融合，在 AIS gait screening 中同时提升分类性能与可解释性。

## 研究问题与动机

Adolescent Idiopathic Scoliosis（AIS）的 radiographic Cobb angle 是临床诊断金标准，但重复 X-ray 不适合作为大规模无创筛查。视频步态分析有潜力提供 scalable screening，不过已有工作常面临两个问题：同一受试者多个 clip 跨 train/test 造成 data leakage，以及纯 RGB / silhouette 特征难以说明模型到底依据哪些临床运动模式进行判断。

作者因此希望同时解决 dataset split reliability、临床可解释表征与 multimodal fusion 三个问题：使用 radiographic Cobb angle 作为标签，强制 test subject 完全未见，并把临床 gait kinematics 显式编码为可查询的 knowledge map，再与 video 和 text 共同学习。

## 核心方法

### ScoliGait benchmark

数据集共招募 **850 名参与者**，以 Cobb angle 10° 为二分类阈值，其中 488 positive、362 negative。550 名训练参与者的原始视频被切成互不重叠的 clips，每个 clip 为 **96 frames、30 Hz、1080p**，最终得到 **1,572 个 training clips**；测试集包含 **300 个 clips，对应 300 名训练中未出现的独立受试者**，其中 212 positive、88 negative。

### Clinical-prior kinematic knowledge map

knowledge map 共 **238 个 features**，由 motion space 140 项、self-skeleton space 32 项、signal cross-correlation 66 项组成，覆盖 joint distance、inter-segment angle 与 limb synchronization 等 gait dynamics。knowledge map 与原 video 在时间轴上对齐，attention 可以重新映射到具体临床变量与时段，从而比单纯 skeleton heatmap 更容易解释。

### Multimodal latent attention pooling

模型为三路 encoder：knowledge map 与 video 均使用 ViT，text 使用 `all-MiniLM-L6-v2`。单模态实验使用 8 个 Transformer layers，多模态设置使用 4 layers。作者以 latent attention pooling 替代 standard average pooling，用 learnable latent dictionary 通过 cross-attention 压缩 sequence features，再融合 knowledge map、video 与 clinical text。

## 数据集与评价指标

- **ScoliGait：**850 participants；training subjects 550，1,572 clips；held-out test 300 participants / 300 clips，无 participant overlap。
- **类别分布：**总计 488 positive / 362 negative；test 为 212 positive / 88 negative。
- **输入：**RGB gait video、temporal kinematic knowledge map、clinical text description。
- **标签：**radiographic Cobb angle，10° threshold；annotations 由 senior orthopaedic specialists 进一步核验。
- **主要指标：**Accuracy、overall F1，以及 positive / negative class Precision、Recall、F1。
- **对比：**Video-only、Knowledge Map-only、Knowledge Map+Video、三模态模型，以及 ScoNet-MT；同时比较 concat、standard attention 与 latent attention pooling，并消融 positional alignment。

## 主要结果

- **Video-only：**Accuracy / overall F1 为 **0.563 / 0.488**。
- **Knowledge Map-only：**提升到 **0.580 / 0.520**，说明显式 kinematic representation 本身比 raw video 更具判别性。
- **Knowledge Map + Video：**达到 **0.640 / 0.562**。
- **Knowledge Map + Video + Text：**最佳结果为 **0.700 Accuracy、0.619 overall F1**；positive class Precision / Recall / F1 为 **0.486 / 0.409 / 0.444**，negative class 为 0.769 / 0.820 / 0.794。
- **ScoNet-MT：**不同配置 Accuracy 为 0.644–0.664，但 positive Recall 仅 0.021–0.087，说明总体 Accuracy 会掩盖对真正 scoliosis cases 的漏检问题。
- **Fusion ablation：**aligned positional embeddings 下，Concat / Cat+Att / Cat+Latent 的 Accuracy 分别为 **0.550 / 0.610 / 0.640**，F1 为 **0.495 / 0.531 / 0.562**；将 Cat+Latent 改成 non-aligned embeddings 后降至 0.623 / 0.531。

## 优点

- 使用严格 subject-disjoint test，直接针对 gait-video 研究中常见的 repeated-subject leakage。
- 标签来自 radiographic Cobb angle，而不是只依赖主观 physical screening。
- 238 维 kinematic map 将 motion feature 与临床概念显式对应，使 attention 可以落到具体 gait variable 与时间段。
- 三模态结果不仅提高 overall performance，也明显改善 positive-class detection，相比只看 Accuracy 更符合 screening 需求。
- latent attention pooling 与 temporal positional alignment 都有明确消融支持。

## 局限

- 最佳模型虽然 Accuracy 达到 0.700，但 **positive Recall 仍只有 0.409**，对于真正筛查任务意味着漏诊率仍然较高，尚不能直接作为临床 screening tool。
- 论文展示的是同一 benchmark 内的 subject-independent held-out test，并未提供独立多中心、跨设备或 prospective cohort 验证。
- 本次未核验到公开代码、数据下载或项目主页，复现性目前受限。
- 数据类别与性别分布存在明显差异：positive 为 384 female / 104 male，negative 为 72 female / 290 male。**推断：**如果模型没有显式控制该混杂因素，RGB appearance 或 sex-correlated gait characteristics 可能被利用为 shortcut，需要做 sex-stratified / matched evaluation。
- 任务为 binary AIS screening，并没有直接验证 Cobb-angle regression、curve type、severity progression 或 longitudinal prediction。

## 个人评价

这篇对 clinical gait 的价值不只是“又一个多模态网络”，而是给出了一个比较具体的 clinical-prior representation：把可解释的运动学变量做成时序 knowledge map，再与 RGB / text 对齐融合。相比只在最终层加入 clinical metadata，这种设计更容易检查模型在什么时间、依据什么运动学变量做判断。

但当前 positive Recall 仍偏低，而且 demographic confounding 与跨中心泛化尚未解决。因此现阶段更适合作为 clinical-prior / multimodal method baseline，而不是临床性能上已经成熟的 screening system。

## 与我的研究关联

**推断：**对脊柱疾病 gait video，可直接借鉴 `RGB / flow / keypoints → kinematic knowledge map → clinical text / prior → latent attention fusion`。knowledge map 不必照搬 AIS 的 238 项，可以围绕 trunk lean、pelvis rotation、左右 asymmetry、hip/knee ROM、phase、step width/length、arm swing 与 periodic coordination 重新设计，并与 learned motion representation 并行比较。

一个很清晰的实验链是：`RGB-only → keypoint temporal model → clinical-knowledge map only → RGB+knowledge → RGB+knowledge+text → + 3D pose / biomechanics`，同时报告 overall AUROC/F1、positive sensitivity、calibration 与 subgroup performance。尤其应坚持 subject-disjoint，并增加 sex/age matching 或分层报告，避免模型利用 cohort-specific shortcut。

## 后续阅读

- Pose as Clinical Prior: Learning Dual Representations for Scoliosis Screening：该工作在本文 Related Work 中作为 clinical-prior scoliosis baseline。
- Scoliosis1K / ScoNet-MT：理解 silhouette gait screening 与 repeated-subject benchmark 的历史背景。
- BioGait-VLM：对比更通用的 vision-language-biomechanics clinical interpretation 路线。
- **计划实验：**将 clinical knowledge features 与周期 motion feature、3D keypoints、RGB / optical flow 做逐级融合，并进行 sex/age-stratified 与 cross-camera validation。
