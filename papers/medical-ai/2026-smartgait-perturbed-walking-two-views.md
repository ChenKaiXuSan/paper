---
title: "Validation of AI-based markerless gait event detection during perturbed walking using smartphone videos from two camera perspectives"
authors: "Valerie Graf, Vanessa Haug, Daniel Seebacher, Manuel Stein, Michael Denkinger, Markus Gruber, Neltje Piro, Michael Munz, Tim Fleiner, Michael Schwenk"
venue: "Frontiers in Sports and Active Living, volume 8 (2026)"
year: 2026
reading_date: 2026-11-18
status: skimmed
tags:
  - medical-ai
  - clinical-gait
  - gait-event-detection
  - multi-view
  - smartphone-video
  - robustness
---

# Validation of AI-based markerless gait event detection during perturbed walking using smartphone videos from two camera perspectives

## 基本信息

- **作者：** Valerie Graf, Vanessa Haug, Daniel Seebacher, Manuel Stein, Michael Denkinger, Markus Gruber, Neltje Piro, Michael Munz, Tim Fleiner, Michael Schwenk
- **会议/期刊：** Frontiers in Sports and Active Living, vol. 8；2026-08-04 在线发表；正式期刊论文
- **年份：** 2026
- **阅读日期：** 2026-11-18
- **阅读状态：** `skimmed`
- **标签：** `medical-ai`, `clinical-gait`, `gait-event-detection`, `multi-view`, `smartphone-video`, `robustness`
- **论文：** https://www.frontiersin.org/journals/sports-and-active-living/articles/10.3389/fspor.2026.1902010/full （DOI: https://doi.org/10.3389/fspor.2026.1902010）
- **代码：** 暂无（SMARTGAIT 软件的论文实验实现未核验到官方开源代码）
- **数据集：** 暂无（研究原始数据需基于法律及隐私条件向作者请求，不是已核验的公开数据集）
- **项目主页：** 暂无

## 一句话总结

利用五名老年人的前方/斜向双手机步态视频验证 SMARTGAIT 的初始/最终接触事件检测，发现视角对扰动条件下的鲁棒性影响显著，并探讨少量定向再训练的收益。

## 研究问题与动机

常规临床步态事件检测假设相对稳定、周期性的行走；突发跑台扰动会引入补偿步、节奏中断和异常着地，破坏正常步态下学习的时序规律。传统 IMU/mocap 测量不便于规模化临床采集。本文研究基于单目手机视频的 AI 系统在反应性平衡条件下能否正确识别脚部 **initial contact (IC)** 和 **final contact (FC)**，并观察摄影机视角与专门训练是否影响结果。

## 核心方法

1. **双视角同步视频：** 两台 iPhone SE（第三代）分别位于受试者前方与斜向，固定拍摄跑台行走及突发扰动，约 33 FPS。
2. **SMARTGAIT 特征：** 由 3D Pose Estimation Model (PEM) 提取 24 个关节的 3D 坐标以及六个角度，共 **78 维**结构化运动学特征。
3. **时序事件网络：** Gait Contact Event Model (GCEM) 用双向 LSTM 产生左右脚 IC/FC 时序概率，通过峰值识别确定事件帧。
4. **评估与适配：** 与人工逐帧事件标注比较，容忍窗口为 ±33 ms；比较正常/扰动条件、正面/斜向视角，并用留出划分的针对性再训练检查是否能改善困难条件。
5. **既有训练数据：** GCEM 先前在约 300 名健康及病理参与者的行走资料上训练，但其中 **不包含本研究所测试的跑台扰动**。

## 数据集与评价指标

- **参与者：** 五名老年人，年龄均值 **83 ± 4.47 岁**，三男两女；受控跑台步行，每位参与者设置八次意外扰动。
- **标注规模：** 共 **2,477 个**人工步态事件，其中正常行走 **2,106** 个、扰动行走 **371** 个；视频时长约每人 2–3 分钟的分析片段，非大规模独立测试集。
- **输入：** 33 FPS 手机 RGB 视频，分别从前方与斜向单独检测；论文并未把两个视角做融合网络。
- **输出：** 每只脚 IC/FC 的时间帧；不是脊柱疾病诊断类别、真实 GRF 或独立 3D mocap 关节真值。
- **指标：** 事件级 Precision、Recall、F1（±33 ms），以及不同条件/视角的比较和再训练实验。

## 主要结果

- 正常步态至扰动步态的事件检测 **median F1**：**正面视角 0.68 → 0.49**；**斜向视角 0.96 → 0.88**，说明前方视角对非典型补偿步更脆弱。
- 对正面视角的扰动片段进行针对性适配后，F1 **0.49 → 0.76**，但研究报告 **p=0.06**，按常用 0.05 阈值不显著；Recall **0.34 → 0.64**，同样 p=0.06。
- 斜向视角适配前后约 **0.88 → 0.88，p=0.18**，并无实质改进。
- 因为 n=5 且扰动事件仅 371 个，不应将趋势解释为已验证的临床泛化收益。

## 优点

- 定义了比平稳步行更困难的 **perturbed walking** 测试条件，有利于揭示一般步态模型面对补偿动作的退化。
- 同一批参与者使用两个摄像机视角并有人工事件标签，能直接分析视角敏感性。
- 不只报告正常条件平均 F1，还分开报告扰动、召回及训练适配效果，为小样本研究提供负结果。

## 局限

- 样本仅五人且均为高龄参与者；固定跑台和双固定手机不代表真实居家/移动相机场景。
- 未提供可公开直接下载的原始视频或完整开源 SMARTGAIT 权重/代码，独立复现受限。
- 人工事件标注和受控场景没有同步力板作为所有事件的独立参考，3D 运动学误差也未被直接验证。
- 研究报告两位作者与 SMARTGAIT 开发公司 Subsequent GmbH 存在雇佣关系，需要结合潜在利益关联审视外推范围。
- **推断：** 若仅在五名参与者内进行适配和评价，无法替代大样本 subject-disjoint、site-disjoint 的外部验证。

## 个人评价

最重要的信息是 **视角 × 步态状态交互**：同一事件模型在斜向视频上对扰动更稳定，而正面视频更依赖额外训练数据。其意义不是展示一个达到临床部署要求的模型，而是提供了 **normal→perturbed domain shift** 的实证案例。小样本下的 p=0.06 结果应视为探索性趋势而非显著改善，且不应混淆双视角对比与真正多视角融合。

## 与我的研究关联

**推断：** 对 ClinicalGated / 成人脊柱畸形侧视步态分类，可加入 gait event 辅助分支以及周期不规则性分层评估，防止模型仅识别平稳步频而忽略异常代偿。比较 `RGB-only → pose/IC-FC → clinical ROI gating → view/reliability-aware fusion`，以患者为单位拆分训练/验证/测试。对双视角滑雪，则可以借鉴“按视角和运动状态单独校准可靠性”，而不是把低 F1 的观察视角固定等权融合。

## 后续阅读

- 原文 Methods 中的 SMARTGAIT / PEM / GCEM、评估容忍窗口、视角设置与训练适配；对照 Results 的条件×视角统计及表格。
- 仓库相关笔记：VisionMD-Gait、GAVD、Scoliosis1K/ScoNet-MT、GaVA-CLIP、ClinicalGated 相关研究路线。
- **推断／拟复现：** 用现有侧视视频人工标注少量 IC/FC，设计 `steady vs irregular`、`same-view vs unseen-view`、`subject-disjoint vs naive split` 实验；同时报告类别 AUROC 与 gait-event F1 / Recall。
