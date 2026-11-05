---
title: "Computer Vision for Clinical Gait Analysis: A Gait Abnormality Video Dataset"
authors: "Rahm Ranjan, David Ahmedt-Aristizabal, Mohammad Ali Armin, Juno Kim"
venue: "IEEE Access, 13:45321–45339 (2025)"
year: 2025
reading_date: 2026-11-06
status: skimmed
tags:
  - medical-ai
  - clinical-gait
  - gait-abnormality
  - dataset
  - rgb-video
  - view-bias
  - clinical-annotation
---

# Computer Vision for Clinical Gait Analysis: A Gait Abnormality Video Dataset

## 基本信息

- **作者：** Rahm Ranjan, David Ahmedt-Aristizabal, Mohammad Ali Armin, Juno Kim
- **会议/期刊：** IEEE Access 13:45321–45339 (2025)，正式期刊论文
- **arXiv 首次提交：** 2024-07-05
- **年份：** 2025
- **阅读日期：** 2026-11-06
- **阅读状态：** skimmed
- **标签：** medical-ai, clinical-gait, gait-abnormality, dataset, rgb-video, clinical-annotation
- **论文：** https://arxiv.org/abs/2407.04190
- **DOI：** https://doi.org/10.1109/ACCESS.2025.3545787
- **代码：** 暂无（官方公开的是数据标注/URL 仓库，不等于可复现模型训练代码）
- **数据集：** https://github.com/Rahmyyy/GAVD
- **项目主页：** 暂无（独立官方项目页未核验；以作者 GitHub 数据仓库为准）
- **主要价值类型：** Dataset / Baseline / Related Work
- **阅读优先级：** 高（经典重要数据集补读；用于跨场景、跨视角临床视频测试）

## 一句话总结

GAVD 从开放网络视频汇集临床标注的正常/异常步态片段，既为真实世界视频病理步态研究提供可复核的标签和视角元数据，也通过 SlowFast/TSN 跨数据集实验暴露了模型对视角与类别分布的显著敏感性。

## 研究问题与动机

临床步态模型常缺少现实世界 RGB 视频与医学可解释标签，公开数据偏向健康行走或受控室内、且无细粒度病理异常标注。作者综述超过 150 个相关 gait datasets，提出建立由临床专家筛查的 Gait Abnormality in Video Dataset (GAVD)，使 abnormality recognition、临床动作类型标注及跨场景 generalization 可获得更现实的评价基础。

## 核心方法

1. **收集：** 从公开网络视频手工筛选正常与异常行走录像，临床专家观看并标注 gait class/subclass、视角及边界框。
2. **时序分段：** 将一段视频拆分为至少两个 gait cycles、基本单方向、遮挡少的 walking segments；每帧提供 bbox 与元信息。
3. **数据组织：** 官方仓库提供 CSV/PKL、frame index、cam_view、gait_pat、gait_event（部分有标注）、video ID 和 URL；**不直接分发原视频文件**。
4. **评价：** 以预训练 TSN 与 SlowFast 视频分类器评估正常/异常识别，另对健康 GPJATK 与受控模拟异常 CASD 作跨集测试。

## 数据集与评价指标

- **规模：** 452 个来源 URL（正常 41、异常 411）；1,874 条 gait sequences（正常 291、异常 1,583）；458,116 帧（正常 41,340、异常 416,776）。参与者超过 400 名；数据包含多种室内/室外、医疗场景和 camera views（Section 3.2、Table 3）。
- **标注：** normal/abnormal 及 11 种 abnormal subclasses、bbox、view、部分 gait events；大多数异常子类为 idiopathic，即外观异常未必对应已确诊病因。
- **测试构成：** GAVD 测试 300 人/1,132 条异常；GPJATK 测试 32 人/618 条正常；CASD 测试 20 人/738 条侧面序列（94 normal、644 abnormal）；均应注意类别分布。
- **任务/指标：** 视频级 binary abnormality classification accuracy；对不同 camera views 分组统计（Tables 6–8）。原文重点不是关节角回归或外部临床诊断准确率。

## 主要结果

- **GAVD abnormal test：** SlowFast 0.94、TSN 0.92。
- **GPJATK normal test：** SlowFast 0.76、TSN 0.59；侧面 view 的正常识别明显更差。
- **CASD simulated test：** SlowFast 0.87、TSN 0.73；样本严重不平衡且只含侧视。
- **视角差异：** GPJATK 的 SlowFast 在以左右侧视为主的 C3 为 0.56，而以正/背视为主的 C4 为 0.97；TSN 分别为 0.25 与 0.95（Table 8）。
- **关键提醒：** 0.94/0.92 来自只包含异常的 GAVD 测试子集，不能等价理解为平衡、跨患者真实诊断 accuracy/AUROC。

## 优点

- 对普通在线视频进行专家筛查，提供可供跨场景验证的病理步态标注和视角元数据。
- 公布 452 URL、1874 clips 与 458116 frames 的对应定义，便于研究者检查样本分布和 subset。
- 单独测正常数据与模拟异常数据，清楚揭示模型依赖视角、训练域和类别比例的风险。

## 局限

- 官方 GitHub 仅分享 annotation 与 URL，不提供原始视频；链接会失效且重下载须遵守平台条款、版权和机构伦理审批。
- 来源异质但非前瞻性确诊的患者队列；部分“abnormal”来自模拟或外观判断，无法当作疾病病因的金标准。
- 正常/异常及视角分布严重不均衡；GAVD 测试全为异常视频，单独 accuracy 容易误导。
- 仍需 patient-disjoint、source-video-disjoint、site-disjoint 的严格新协议，才能减少同源剪辑与视角捷径（这一点属于**推断**）。
- CASD 20 名受试者的模拟条件不能代表真实临床病理队列；无完整 3D biomechanics GT。

## 个人评价

**推断：**此数据集对临床视频外部泛化很有价值，但适合做 in-the-wild gait abnormality stress test，不能直接用作 ASD 病理诊断真值。最值得重视的是“高 abnormal accuracy 与较差 normal / side-view accuracy 并存”；这比在单个同源数据集上追求总体 accuracy 更具有临床方法学启发。

## 与我的研究关联

- **ASD/ClinicalGated：** 自有侧视 ASD 视频应特别注意视角捷径。建议加入 view-specific 分组、subject-disjoint、独立的 normal/abnormal sensitivity/specificity、calibration 与每名患者聚合的报告。
- **RGB+flow+pose：** **推断：**利用 GAVD 的 URL metadata/clinical subclasses 做异常步态 representation 的弱监督预训练，再在 ASD 81 人队列进行完全独立微调/外部验证；不要把异常标签直接映射为 ASD 标签。
- **可解释性：** 将 clinical ROI、gait phase、joint kinematics 与异常类型报告关联，避免只报告抽象视觉 attention。
- **数据治理：** 需评估 URL 是否仍有效、样本拆分是否具有同源重复，以及视频二次使用和隐私限制。

## 后续阅读

- 优先复核 Section 3.2、Tables 3/4、Section 4.2–4.3、Tables 6–8；审计 URL 可用性和视角标签分布。
- 对接已归档的 [DrGait](2026-drgait-biomechanically-grounded-clinical-gait.md)，注意该工作用 GAVD 的筛选子集，不能假设所有 benchmark 结果可直接比较。
- 尝试按 source video ID、subject 和 camera view 进行组级数据划分；另外报告正负类敏感度与特异度。
