---
title: "Forecasting Epileptic Seizures from Contactless Camera via Cross-Species Transfer Learning"
authors: "Mingkai Zhai, Wei Wang, Zongsheng Li, Quanying Liu"
venue: "arXiv:2603.12887"
year: 2026
reading_date: 2026-09-11
status: skimmed
tags:
  - medical-video
  - seizure-forecasting
  - video-mae
  - self-supervised-learning
  - cross-species-transfer
  - few-shot-learning
---

# Forecasting Epileptic Seizures from Contactless Camera via Cross-Species Transfer Learning

## 基本信息

- **作者：** Mingkai Zhai, Wei Wang, Zongsheng Li, Quanying Liu
- **会议/期刊：** arXiv:2603.12887
- **年份：** 2026
- **阅读日期：** 2026-09-11
- **阅读状态：** `skimmed`
- **标签：** `medical-video`, `seizure-forecasting`, `video-mae`, `self-supervised-learning`, `cross-species-transfer`, `few-shot-learning`
- **论文：** https://arxiv.org/abs/2603.12887
- **DOI：** https://doi.org/10.48550/arXiv.2603.12887
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

该工作把 seizure video analysis 从“发作后检测”推进到“发作前预测”，利用 rodent epilepsy videos 与少量 human videos 对 VideoMAE 做跨物种自监督预训练，在只有 2–4 shot 标注的 human forecasting 场景中探索 contactless video 的早期预警能力。

## 研究问题与动机

癫痫 seizure forecasting 通常依赖 EEG/iEEG，长期家庭监测需要专用硬件；现有 video-based epilepsy 研究又多集中于 seizure onset 后的检测或分类。本文定义严格 video-only 的 forecasting 任务：使用发作前 3–10 秒的视频片段，预测接下来 5 秒内是否发生 seizure。核心难点是 human pre-ictal video 稀缺，因此作者尝试用大规模 rodent epilepsy behavior 学习可跨物种迁移的时空运动表示。

## 核心方法

方法以 VideoMAE-base 为 backbone，分两阶段训练。

第一阶段进行 continual self-supervised pretraining：把 rodent epilepsy videos 与未标注的正常 human epilepsy-patient videos混合，采用 tube masking，让 encoder 从可见 spatiotemporal patches 中提取表示，decoder 重建被遮挡像素，以 MSE 作为训练目标。作者系统测试 0.1–0.9 的 mask ratio，最终完整设置选择 0.3。

第二阶段丢弃 decoder，保留 encoder；使用 CLS token 作为全局视频表征，再接 lightweight binary classifier 预测 seizure / non-seizure。下游采用 2-shot、3-shot、4-shot few-shot fine-tuning，并与 CSN、X3D、SlowFast、linear probing、human-only pretraining 等 baseline 比较。

## 数据集与评价指标

- **RodEpil：** 超过 13,000 个 10 秒视频、19 只 rodents；本文使用全部 2,952 个 epileptic samples，并随机抽取 3,000 个 normal samples 以平衡预训练数据。
- **Human continual-pretraining data：** 深圳市第二人民医院 6 名成年 epilepsy patients，共 1,870 个 5 秒 non-seizure clips。
- **Few-shot forecasting benchmark：** 40 个视频序列，其中 20 个 pre-ictal、20 个 interictal，来自两个 public epilepsy databases 与 proprietary clinical videos。每个 N-shot task 内 support/query 不重叠，但 2/3/4-shot 是从同一 40-video pool 独立采样的试验。
- **输入：** forecasting 定义使用 3–10 秒 pre-ictal clips；VideoMAE 实现中采样 16 frames、sample rate 2、224×224。
- **指标：** balanced accuracy（bacc）、ROC-AUC、PR-AUC。

## 主要结果

完整 cross-species setting 在 2/3/4-shot 平均得到 **bacc 0.7230、ROC-AUC 0.7558、PR-AUC 0.7091**；human-only baseline 为 **0.7149 / 0.7491 / 0.6943**，SlowFast 为 **0.6620 / 0.7065 / 0.6812**。

分 shot 看，完整方法 2-shot 的 bacc / ROC-AUC / PR-AUC 为 **0.7389 / 0.7682 / 0.7269**；3-shot 为 **0.7176 / 0.7374 / 0.6674**；4-shot 为 **0.7125 / 0.7617 / 0.7331**。需要注意，2-shot balanced accuracy 上 human-only 的 **0.7444** 反而略高于完整方法，说明跨物种预训练的收益并非所有 setting 都一致。

预训练数据消融中，`+R(Y/N)+H` 的平均结果最好；mask-ratio 实验显示完整设置在 **0.3** 时平均 bacc 达到最高 **0.7230**。

## 优点

- 把 clinical video 从 seizure detection 推向 forecasting，研究问题具有明显的主动预警价值。
- 在 human 标注极少的情况下尝试利用 cross-species behavioral dynamics，是一种有启发的数据扩展策略。
- 同时报告 human-only、rodent-subset 与 cross-species ablation，可以区分收益究竟来自什么预训练数据。
- 使用 balanced accuracy、ROC-AUC、PR-AUC，而不是只报告普通 accuracy。

## 局限

- 核心 forecasting benchmark 只有 **40 个视频**，规模非常小，无法支持可靠的临床外部有效性结论。
- support/query 虽在单次 task 内无重叠，但不同 shot 实验来自同一个 40-video pool；尚缺独立、纵向、prospective external cohort。
- 预测 horizon 固定为未来 5 秒，尚未验证更长 lead time、Seizure Prediction Horizon / Seizure Occurrence Period 与真实 false-alarm burden。
- 跨物种收益总体不大，例如平均 bacc 仅从 human-only 的 0.7149 提升到 0.7230，而且某些单独 setting/baseline 在个别指标上更好。
- 当前只使用视频；作者也明确提出未来应结合 audio、HRV 等非侵入式模态。
- 目前为预印本，且本次未核验到官方代码、该论文专用数据下载页或项目主页。

## 个人评价

这篇工作的主要价值不在当前 0.72 左右的 balanced accuracy，而在两个思路：一是将 medical video 任务从当前状态分类改成 **future event forecasting**；二是在极端数据稀缺时使用相关但不同 domain/species 的无标签视频做 representation pretraining。

**推断：**在真正临床部署前，必须补充 subject-independent / site-independent longitudinal validation，并使用 sensitivity、false alarms per hour、time-to-event、calibration 等更贴近 forecasting 的指标，否则 ROC-AUC 很难对应实际预警价值。

## 与我的研究关联

对于临床 gait / 医疗视频 AI，这篇可以启发从“疾病分类”进一步转向 **未来风险或状态变化预测**。例如使用连续 gait clips 预测未来功能恶化、跌倒风险或 severity transition，并通过大规模健康/非目标疾病运动视频做自监督预训练。

**推断：**更适合迁移的实验结构是 `task-specific supervised → human-only self-supervised → cross-domain/cross-cohort pretraining → multimodal fusion`，而不是直接迁移 rodent data。对于 RGB、光流、关键点、VLM 的多模态路线，可分别比较 appearance、motion、pose 与结构化临床特征对未来事件预测的贡献，并同时评价 calibration 和 abstention。

## 后续阅读

- VideoMAE 及医学视频中的 self-supervised temporal representation。
- seizure forecasting 的 EEG / multimodal clinical evaluation protocol，尤其 false-alarm rate 与 prediction horizon。
- 与 gait progression、fall-risk forecasting、driver future-motion prediction 等 temporal forecasting 任务做方法层比较。
