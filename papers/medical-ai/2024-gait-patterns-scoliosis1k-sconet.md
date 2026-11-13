---
title: "Gait Patterns as Biomarkers: A Video-Based Approach for Classifying Scoliosis"
authors: "Zirui Zhou, Junhao Liang, Zizhao Peng, Chao Fan, Fengwei An, Shiqi Yu"
venue: "MICCAI 2024, LNCS 15005, pp. 284–294"
year: 2024
reading_date: 2026-11-13
status: skimmed
tags:
  - clinical-gait
  - scoliosis
  - spine
  - medical-video
  - silhouette
  - gait-recognition
  - multi-task-learning
---

# Gait Patterns as Biomarkers: A Video-Based Approach for Classifying Scoliosis

## 基本信息

- **作者：** Zirui Zhou, Junhao Liang, Zizhao Peng, Chao Fan, Fengwei An, Shiqi Yu
- **会议/期刊：** MICCAI 2024，LNCS 15005，pp. 284–294；arXiv 首次提交 2024-07-08
- **年份：** 2024
- **阅读日期：** 2026-11-13
- **阅读状态：** `skimmed`
- **标签：** `clinical-gait`, `scoliosis`, `spine`, `medical-video`, `silhouette`, `gait-recognition`
- **价值类型：** Dataset / Baseline / Method Module / Related Work
- **阅读优先级：** 高（2024 年经典临床脊柱步态工作补读）
- **论文：** https://papers.miccai.org/miccai-2024/348-Paper1763.html
- **DOI：** https://doi.org/10.1007/978-3-031-72086-4_27
- **代码：** https://github.com/ShiqiYu/OpenGait/blob/master/opengait/modeling/models/sconet.py
- **数据集：** https://zhouzi180.github.io/Scoliosis1K/
- **项目主页：** https://zhouzi180.github.io/Scoliosis1K/
- **arXiv：** https://arxiv.org/abs/2407.05726
- **论文状态：** 正式 MICCAI 2024 会议论文；Scoliosis1K 数据须签署学术使用协议申请解压权限
- **核验来源：** MICCAI 官方论文正文及 PDF、上述作者数据集主页、OpenGait 官方 ScoNet 文件与 config

## 一句话总结

该论文提出真实学校筛查场景中的 Scoliosis1K 青少年步态数据集，并以 silhouette-based ScoNet-MT 多任务学习对脊柱侧弯的筛查标签分类，展示步态潜在非侵入式线索，同时暴露样本分布和临床验证边界。

## 研究问题与动机

脊柱侧弯早筛依赖专业体检以及必要时的影像，而视频步态能以低成本收集；既有 gait recognition 研究偏重身份识别，缺乏与真实脊柱筛查标签相结合的数据集。作者从校园走廊非接触拍摄人群，建设经过医护人员现场评估标注的 Scoliosis1K，并研究 silhouette 是否携带可用于 scoliosis screening 的信息。

## 核心方法

1. **采集与标注：** 在单所学校走廊摄录青少年直线行走，现场体检将受试者标记为 scoliosis positive、neutral（边界）或 negative；其标签不能自动等同影像学 Cobb angle 确诊。
2. **人体提取：** 基于 ByteTrack/PP-HumanSeg 等预处理生成人体轮廓序列，减少脸部及背景可识别信息；模型输入约 30 帧、64×44 的 binary silhouettes。
3. **ScoNet：** 使用 ResNet 风格时空特征提取、temporal pooling、horizontal pooling、BNNeck 和三分类 cross-entropy 预测 positive/neutral/negative。
4. **ScoNet-MT：** 加入身份步态识别相关的 triplet loss（margin 0.2），以多任务监督提升潜在区分能力；并做类别分布及样本比例消融。
5. **数据更新说明：** 2024 原文以 silhouettes 为主；项目主页明确 2025-08 起另提供 2D pose 数据，不能将后续数据发布误写为原模型输入。

## 数据集与评价指标

- **规模：** Scoliosis1K **1,050 名青少年**、**1,493 条行走序列**、约 **447,900 帧**；校内摄像为约 720p/15 FPS。
- **标签人数：** positive **176**，neutral **82**，negative **792**，明显类别不均衡；系学校筛查样本，非多中心诊断队列。
- **划分：** 745 条训练、748 条测试序列。论文未明确保证跨序列的 subject-disjoint，需要人工核查官方 protocol 再作患者级独立评价。
- **输入/输出：** 30 帧 64×44 silhouettes → 3 个筛查标签；不输入 3D 关键点、临床病历、Cobb 角和医疗影像。
- **指标：** 分类 Accuracy、Sensitivity、Specificity 与不同 positive/negative 比例消融；不是肢体 3D 重建精度。

## 主要结果

- **原文主结果：** ScoNet-MT Accuracy **82.0%**，作者报告 Sensitivity **99.0%**、Specificity **76.5%**；ScoNet（无多任务）Accuracy **51.3%**、Specificity **33.2%**，简单 CNN baseline Accuracy **45.5%**。
- **分布敏感性：** 论文示例改变训练类比后，ScoNet-MT Accuracy 从 **95.2%**（1:1:2）降为 **49.5%**（1:1:16），表明模型依赖采样和类先验。
- **需要复核的指标歧义：** 原文混淆矩阵正类行按可见计数为 357 / (59+357+3) 约 **85.2%**，与主文报告的 Sensitivity 99.0% 并不显然一致；可能涉及正负类定义、归一化方式或图中设置不同，不能直接视作真实临床 99% 患者召回率。该项属对原表与图的审慎比对，尚需作者解释。

## 优点

- 将视频步态与脊柱问题直接对接，数据规模明显大于许多先前的医学步态小样本研究。
- 提供真实的校园筛查、silhouette 隐私保护表示及官方代码入口；后续另有 2D pose 发布。
- 多任务和类比实验揭示提升可能来自表示学习和类别分布，不只是网络更深。

## 局限

- **单中心/学校：** 仅一所中国学校的青少年筛查，不能推广到成年 ASD、其他地区、不同病理或医院人群。
- **标签边界：** 主要依赖现场体格筛查评估，不能默认为全部样本经过独立脊柱 X 光 Cobb angle 真值确诊；screening 不等同 diagnosis。
- **数据泄漏风险（推断）：** 划分按视频序列展示，未见明确的严格 subject-disjoint 保证；同人多视频会高估泛化效果，需审查官方 split。
- **潜在混淆：** 类别极不平衡且摄像视角受控；衣着、体型、身高等非疾病因素可能构成 shortcut（推断）。
- **结果报告：** 主文 sensitivity 与混淆矩阵正类召回存在定义/计数疑问，应独立复算，而非直接将 99% 解释为临床灵敏度。
- **访问限制：** 原始 RGB 视频不作为完全开放资料提供；silhouette/2D pose 的使用需依照许可申请。

## 个人评价

最有价值的是首次为脊柱异常 video-based gait classification 提供可基准化的数据和对照。现阶段它更适合说明“从步态存在可研究的脊柱筛查信号”，不能据此证明单摄像头系统达到确诊要求。必须优先解决 subject-level split、类先验敏感性和独立临床真值三项问题。

## 与我的研究关联

**推断：**对 ClinicalGated 的成年 ASD 侧视视频，适合复现 silhouette-only baseline，比较 `silhouette / 2D pose / RGB+flow / +clinical ROI / +temporal phase`；借鉴身份 auxiliary/triplet 任务时谨慎避免直接把身份差异与疾病标签绑定。优先做 patient-disjoint、不同病理/相机 viewpoint 分组、性别年龄相关混淆控制，并补充 Sensitivity/Specificity、AUROC、置信区间、校准误差。该数据集可作外部异年龄、异场景 stress test，但青少年 scoliosis 和成人脊柱畸形 ASD 不是同一诊断终点。

## 后续阅读

- **优先阅读：** Sections 2.2、3.2、4.1–4.3、Tables 1–4、混淆矩阵。
- 使用官方 OpenGait 的 ScoNet 实现与 `configs/sconet/sconet_scoliosis1k.yaml`，确认论文对应版本、实验 split 是否受试者互斥。
- 对照自己 ClinicalGated、GAVD、VisionMD-Gait 和姿态强基线，区分 silhouette、pose 和 clinical ROI 的因果解释能力。
- 申请官方学术数据访问，核对 2025 年新增 pose 数据的骨架拓扑、样本数与许可范围。
- 复核原文 Sensitivity 与混淆矩阵定义，并报告每类 confusion matrix 和 patient-level recall。
