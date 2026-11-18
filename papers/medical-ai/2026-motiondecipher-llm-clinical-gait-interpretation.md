---
title: "MotionDecipher: LLM-Assisted Clinical Interpretation of Human Motion Signals for Efficient Gait Analysis"
authors: "Mingrui Ma, Tian Tan, Jinsi Guo, Jessica Rose, Yiwen Dong"
venue: "arXiv:2611.08795 (preprint; manuscript lists future IMWUT issue)"
year: 2026
reading_date: 2026-11-19
status: skimmed
tags:
  - medical-ai
  - clinical-gait
  - llm
  - gait-kinematics
  - report-generation
---

# MotionDecipher: LLM-Assisted Clinical Interpretation of Human Motion Signals for Efficient Gait Analysis

## 基本信息

- **作者：** Mingrui Ma, Tian Tan, Jinsi Guo, Jessica Rose, Yiwen Dong
- **会议/期刊：** arXiv 预印本；作者稿件元数据标注 IMWUT 10(4), Article 245、DOI 10.1145/3857982 及 2026 年 12 月期次，**本日尚未独立核实正式刊出**，故不宣称已经正式出版
- **提交日期：** 2026-11-05
- **年份：** 2026
- **阅读日期：** 2026-11-19
- **阅读状态：** skimmed
- **标签：** medical-ai, clinical-gait, llm, gait-kinematics, report-generation
- **论文：** https://arxiv.org/abs/2611.08795
- **代码：** 暂无
- **数据集：** 暂无（未找到患者数据官方公开下载入口）
- **项目主页：** 暂无
- **论文状态：** 可信预印本，近期研究

## 一句话总结

先将实验室步态时序波形转成可解释的临床 tokens 和按部位/步态相位组织的结构化证据，再检索专家示例提示 LLM，生成与临床专家描述更接近的定量步态报告。

## 研究问题与动机

临床 3D mocap 和测力台虽提供连续的 joint-angle、GRF、moment 和 power 波形，人工筛选关键异常并撰写报告十分耗时；直接向通用 LLM 输入波形又易出现错误数值和模糊表述。本文关注报告的运动学/动力学描述部分，不进行病因诊断、长期预后预测或个体治疗推荐。

## 核心方法

1. 从已检测 gait events、分割并归一化为 0–100% 的 gait cycle kinematic/kinetic signals 出发，Human Motion Tokenizer 使用 Tendency、Comparative、Statistical、Frequency 四类编码。
2. Meta Feature Selector 筛选关键异常，以 body-part × gait-phase 形成 Structured Representations，避免直接将冗余波形交给 LLM。
3. Hybrid Context Constructor 将文本和数值相似度结合，检索 Gait Representation–Text Demonstration Corpus（GRTDC）的专家描述，用 in-context learning 生成临床报告。
4. 额外设计 GaitTripleF1 评价解剖部位与运动状态的匹配，而不仅依赖普通文本 BLEU、ROUGE、BERTScore。

## 数据集与评价指标

- 2024-01 至 2025-07 单一认证步态实验室共 **122 名患者、46 名健康对照**。主要报告生成实验使用两名患者作为专家示范，其余 **120 名患者**作为测试；不能将 168 人误写成主报告测试例数。
- 10 台光学摄像机以 **100 Hz** 采集 marker-based 3D motion，测力台采样 **50 Hz**；每人 7 米通道 10–20 趟步行，提取 gait events 后逐步态周期标准化。
- 输入已预处理的 joint-angle、GRF、moment、power 时序；输出按身体部位与步态相位组织的临床文字描述。Baseline 包括 GPT-5、Kimi-K2、Gemini 3、Qwen3-Max 和模块消融。指标为 BLEU、ROUGE-L、METEOR、BERTScore、MEDCON-Gait、GaitTripleF1。

## 主要结果

- Table 1：MotionDecipher 的 **BERTScore 0.91±0.01**，GPT-5 为 **0.78±0.03**；**GaitTripleF1 0.91±0.07**，GPT-5 为 **0.63±0.10**；**MEDCON-Gait 0.76±0.05**，GPT-5 为 **0.60±0.06**。
- Table 2：去掉 Human Motion Tokenizer 后 BERTScore 为 **0.81**、GaitTripleF1 **0.68**；去掉 Meta Feature Selector 后 GaitTripleF1 **0.71**，规则式方法 **0.74**。
- 作者称报告生成约 **2 分钟/患者**，人工数据分析及报告起草约 **4–6 小时/患者**；这些是报告工时，不包括全部采集流程。
- 另有 60 条专家审阅描述，其中 4 条被判为漏述或措辞不精确（6.7%）；与 120 人主测试集不是同一评估计数。论文摘要及表格对百分比提升的口径不完全一致，建议只比较原始数值。

## 优点

- 先确定量化证据，再让 LLM 语言化，有助于临床可审计性。
- 多项文字与医学关系指标、模块消融及医生审阅共同评价，且清楚限制在描述性报告任务。

## 局限

- 依赖昂贵的 marker mocap 和 force plate，不是视频 RGB 输入，也没有 ASD 疾病外部队列/跨机构验证。
- 120 名主要测试患者中儿童与青少年合计 93 人，迁移到成人脊柱疾病不确定。
- 与专家报告相似不能证明病因判断或治疗建议正确；生成仍有信息遗漏和措辞不准。
- 本次未核验到官方可运行代码、患者公开数据或正式期刊最终刊出页面。

## 个人评价

最值得学习的是把数值测量、可复查语义 token 和生成语言分层。它不直接提升视频姿态估计或疾病分类，但可作为 ClinicalGated 结果解释的后处理模块。依赖精准上游信号的限制不能忽视。

## 与我的研究关联

**推断：**可把 ClinicalGated 的头/躯干/骨盆/足部 ROI、步态周期特征、倾斜角及不对称指标映射成 body-part × phase 结构化证据，并选择临床示例生成医生可核查的说明。建议比较 video-only classification → +verified kinematics → +structured token report → +retrieval ICL，同时做 patient-disjoint、跨视角和 pose noise 的误差传播测试，报告错误数字/事实幻觉率与医生盲评。没有 GRF 测量时不能把模型估计动力学当成真实测力数据。

## 后续阅读

- 精读 Sections 2.1–2.4、3.1–3.7 与 Tables 1–2，特别复现 Human Motion Tokenizer、Hybrid Index 和 GaitTripleF1。
- 与仓库 DrGait、GaVA-CLIP、BioGait-VLM 对比，核查未来公开代码/数据和正式刊出状态。
