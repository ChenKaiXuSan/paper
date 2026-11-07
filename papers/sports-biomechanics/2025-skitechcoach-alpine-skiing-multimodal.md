---
title: "SkiTechCoach: A Multimodal Alpine Skiing Dataset with 3D Body Pose, Sole Pressure, and Expert Coaching"
authors: "Toshihiro Hirano, Yuki Tabei, Yichen Peng, Chen-Chieh Liao, Erwin Wu, Hideki Koike"
venue: "MMSports '25（8th ACM Workshop on Multimedia Content Analysis in Sports）"
year: 2025
reading_date: 2026-11-08
status: skimmed
tags:
  - sports-biomechanics
  - alpine-skiing
  - multimodal-dataset
  - 3d-pose
  - pressure
  - coaching
  - vlm
---

# SkiTechCoach: A Multimodal Alpine Skiing Dataset with 3D Body Pose, Sole Pressure, and Expert Coaching

## 基本信息

- **作者：** Toshihiro Hirano, Yuki Tabei, Yichen Peng, Chen-Chieh Liao, Erwin Wu, Hideki Koike
- **会议/期刊：** MMSports '25（8th International ACM Workshop on Multimedia Content Analysis in Sports）, pp. 39–46；2025 年 10 月
- **年份：** 2025
- **阅读日期：** 2026-11-08
- **阅读状态：** `skimmed`
- **论文状态：** ACM 正式发表；近期重要专题论文补读
- **标签：** `sports-biomechanics`, `alpine-skiing`, `multimodal-dataset`, `pressure`, `coaching`
- **论文：** https://doi.org/10.1145/3728423.3759413
- **DOI：** https://doi.org/10.1145/3728423.3759413
- **代码：** 暂无（作者 GitHub 当前仅 README/图片资源，未见可核验可运行实现）
- **数据集：** 暂无（作者项目称公开数据集，但未发现可直接核验的下载或访问申请入口）
- **项目主页：** https://skitechcoach.pages.dev/
- **官方相关仓库：** https://github.com/koikelab-team/SkiTechCoach （介绍占位仓库，不应误称已提供数据或训练代码）
- **主要价值类型：** Dataset / Baseline / Related Work
- **阅读优先级：** 高（滑雪 motion+压力+技术评价的直接参考）
- **主要核验来源：** 作者项目主页、作者个人主页、实验室出版列表、官方相关仓库 README

## 一句话总结

SkiTechCoach 是在室内滑雪模拟器上同步采集 3D 姿态、足底压力、FIS points 与专业教练文本/手绘标注的高山滑雪多模态数据集，并以技术分数预测和自动教练建议作为应用基线。

## 研究问题与动机

仅依据人体关键点的动作识别，不一定能评价滑雪动作的受力、技术水平或给出可执行的训练建议；现有通用 pose 数据集也很少把压力与专家反馈同步保存。作者将独立的运动学、足底压力、运动员技术等级指标与人工反馈组织到统一样本，探索 pose/pressure 在 skill assessment 与语言反馈中的价值。

## 核心方法

- **数据采集：** 在室内滑雪模拟器上记录不同速度的 slalom turns，形成视频、3D motion、鞋内足压与运动员 FIS points 对应的记录；教练额外给出口头/文字及部分手绘反馈。
- **下游任务：** 使用 pose 与 pressure 两种模态预测 FIS points；同时检验多模态大语言模型是否能生成具有体育专项知识的 coaching。
- **数据组织价值：** 将客观运动学、传感器测量和主观专家评价纳入同一个时间对齐体系，支持按动作片段评估，而非仅给整段视频一个动作类别。
- **重要核验边界：** 官方项目页只总结了 baseline 的任务和方向，没有给出各模型、数据划分与评价表中的具体数值；ACM 正文 PDF 曾返回 403，不能从摘要猜测 RMSE/MAE 或模型排名。

## 数据集与评价指标

- **受试者/场景：** 20 名有经验的滑雪者；单一受控室内滑雪模拟器，slalom turns 与不同速度设置。
- **数据量：** **71 条高质量滑雪序列、约 160K 视频帧**；另有 **305 条 coaching clips**，其中 **213 条包含视觉手绘标注**，反馈总时长约 **136 分钟**。
- **输入/输出：** 同步视频/3D pose/足压 → FIS points 估计；或与教练标注结合，生成/评价个性化 coaching。
- **基线与评价：** 作者报告 pose 与 pressure 的 skill-score prediction baselines，以及基于 MLLM 的 coaching 评估。**具体模型、FIS 预测误差定义和数值：暂无可核验数据**，不填造出来的 MAE、RMSE 或更好模态结论。

## 主要结果

- 最重要的可验证定量发现是 **20 人 / 71 runs / 160K frames / 305 coaching clips / 213 visual clips / 136 min** 的多模态数据覆盖。
- 作者项目页给出的基线结论：pose 与 pressure 都能用于 FIS points 估计任务；现有 MLLM coach 缺乏精细高山滑雪领域知识，作者建议专项微调。**尚未核验到具体 baseline 分数和显著性检验结果，故不声称某模态定量优于另一模态。**
- 论文正式来源为 ACM MMSports 2025、pp.39–46，DOI 10.1145/3728423.3759413；这和 2026 年其他 insole-only skiing coaching 论文是不同工作。

## 优点

- 同步的外部 3D motion 与足压（proxy 受力）使技术评价超越传统 MPJPE/动作识别。
- 既含 FIS points 等客观关联变量，又含教练语言与手绘反馈，可用于可解释 sports AI。
- 作者项目提供规模与采集设置的清晰统计，较适合用来设计采集规范。

## 局限

- 仅 20 名有经验运动员和室内模拟器，不能直接推断户外不同坡面、雪况、器材及入门者泛化性能。
- FIS points 属于运动员竞技能力/成绩相关指标，不是当前某次技术动作的无偏黄金真值；教练反馈也可能存在个体差异。
- 作者 GitHub 当前只有 README/图片而无实际代码/数据下载；ACM PDF 暂不可直接访问，关键 baseline 数值与训练/测试 split **未核验**。
- **推断：**鞋内局部压力不能直接等同完整 GRF/CoP 或人体关节力矩，若进行物理评价仍需力台或已校准的外部参照。

## 个人评价

**推断：**这篇对目前仅比较单/双视角 3D MPJPE、速度和 jerk 的滑雪研究，是很直接的任务层参考：真实滑雪研究既需要世界坐标姿态，也需要面向 skill/pressure 的独立指标。数据集虽然采用受控模拟器，采集多模态标签和教练反馈的方式比单独拍大量视频更值得借鉴。

## 与我的研究关联

**推断：**对于双视角 360° 滑雪采集，可以设计 `RGB/pose-only → +temporal pose → +insole pressure → +expert feedback` 消融，并将每个转弯周期对齐至压力峰值、骨盆 roll、膝屈伸和重心迁移。对计划中的 12 台 RX0 II 系统，可加入同步的鞋垫/IMU 作为独立运动学及动力学交叉检验；建议额外记录 expert technical assessment 和 turning events，并采用 subject-disjoint 测试，不把 FIS points 当作逐帧 ground truth。

## 后续阅读

- **优先阅读：** 论文中数据采集、FIS points baseline、MLLM coaching 实验及失效分析（ACM 正文可访问后需补齐具体表格和评价协议）。
- **拟做实验（推断）：** 比较 pose-only 与 pose+pressure 对转弯分段、技术分级和教练一致性的增益；在静态模拟器与户外移动 360 camera 场景间做 domain-shift 对比。
- **待核验：** ACM 正文中的样本划分、具体回归误差与 MLLM 评测量化；作者仓库未来是否发布数据/代码；标注者间一致性。
