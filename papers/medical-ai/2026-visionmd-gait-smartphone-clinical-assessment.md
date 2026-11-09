---
title: "VisionMD-Gait: scalable clinical gait assessment from smartphone videos"
authors: "Shuyu Liu, Alvin Wong, Si Chen, Patrick J. Antonelli, Diego L. Guarín"
venue: "Scientific Reports, 16:4711 (2026)"
year: 2026
reading_date: 2026-11-10
status: skimmed
tags:
  - medical-ai
  - clinical-gait
  - monocular-video
  - spatiotemporal-gait
  - 3d-pose
  - clinical-validation
  - uncertainty-and-bias
---

# VisionMD-Gait: scalable clinical gait assessment from smartphone videos

## 基本信息

- **作者：** Shuyu Liu, Alvin Wong, Si Chen, Patrick J. Antonelli, Diego L. Guarín
- **会议/期刊：** Scientific Reports 16, article 4711；2026-01-07 线上发表；Version of Record 2026-02-03；正式期刊论文。
- **年份：** 2026
- **阅读日期：** 2026-11-10
- **阅读状态：** skimmed
- **标签：** medical-ai, clinical-gait, monocular-video, 3d-pose, spatiotemporal-gait, external-reference-validation
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高（临床视频外部测量标准与实验协议）
- **论文：** https://doi.org/10.1038/s41598-025-34912-5
- **代码：** https://github.com/mea-lab/GaitValidation （官方论文配套 PosePipeline.ipynb、分析脚本；VisionMD 平台本体代码另见 https://github.com/mea-lab/VisionMD-DesktopApp）
- **数据集：** https://github.com/mea-lab/GaitValidation （研究中的传感器数据、分析文件与演示材料；不保证包含全部患者原始视频）
- **项目主页：** https://www.visionmd.ai/ （VisionMD 官方平台主页，为通用产品而非独立 Gait 子项目）

## 一句话总结

使用普通单台手机的纵向/正面行走视频，通过 3D pose + gait transformer + 步态事件检测提取可解释时空指标，并与可穿戴 IMU 参考测量比对，在小规模临床队列中得到较好的测量一致性。

## 研究问题与动机

多相机与 force plate 等实验室步态系统价格高、操作复杂，常见单目方案偏向侧视、缺少对另一类临床患者的参考数据验证。VisionMD-Gait 采用可本地处理的视频分析系统，让普通门诊走廊的一台手机视频输出 gait speed、cadence、step/stride duration、stance/swing/double support time 和 step length，避免把抽象 video classification 当作唯一临床终点。

## 核心方法

1. **视频录制和选择：** iPhone 12，1080×1920、60 fps，手机固定于三脚架，8 m 临床走廊；用户选择人物和步行片段，**去除转弯**。
2. **3D pose：** YOLO 人体框后调用 MeTRAbs，在每帧预测以相机为参考的 metric 3D skeleton；后续 gait transformer 输出时序运动和步态事件，feature extractor 计算临床时空 gait 变量。论文 Fig. 1、Fig. 3 与 Methods 给出三段式架构。
3. **减轻左右侧视模型偏差：** 对视频做水平镜像再次推理，重新映射左右身份，并平均原始与镜像预测；作者强调旨在抵消算法偏置而非抹平真实左右步态不对称。
4. **验证与统计：** 同时佩戴 Noraxon Ultium Motion IMUs，匹配视频片段与传感器步数，使用 Pearson r、相对 MAE%、Bland–Altman 分析测量一致性。患者/健康比较采用按 participant 设置随机截距的 Linear Mixed Model，控制 age、gender、height 并进行 Bonferroni 多重比较校正。

## 数据集与评价指标

- **受试者：** 24 名健康成年人（13 女 11 男）与 10 名门诊前庭性头晕成年人（8 女 2 男）；总计 34 人，单机构招募，非 ASD 队列。
- **视频与参考：** iPhone 12 固定采集 60 fps；同步 Noraxon IMUs 作为 gait measurements 参考。原文称筛选后 **269 个 laps**，但括号内健康 **187** + 头晕 **77** = **264**，存在需核对的计数不一致，不应自行改成任一数字。
- **输入/输出：** 单路 frontal-view 手机 RGB 视频 → 3D joints / gait events → gait speed、cadence、step/stride times、stance/swing/double-support、step length 等。
- **评价：** r、MAE%、Bland–Altman 及 mixed-model adjusted group difference；不提供与外部临床分类器直接可比的 ASD sensitivity / specificity。

## 主要结果

- 健康组多种 gait measures 与 IMU 参考的 Pearson **r = 0.76–1.00**；头晕组 **r = 0.70–1.00**（均多重比较校正后显著）。
- Step duration、stride duration 和 cadence 的相对 MAE **小于 1%**；群体平均的其它指标相对 MAE **小于 10%**，但若按左右腿分别统计，step length 相对误差约 **10%**，相关为 **r = 0.59–0.68**。
- 临床组差异：健康 vs 头晕 gait speed **1.29 ± 0.17 vs 1.05 ± 0.11 m/s**（p=0.004），cadence **108.75 ± 6.45 vs 100.66 ± 7.81 steps/min**（p=0.013）。Step length 差异未达显著（p=0.153）。
- 该研究是外部测量一致性和组间 gait 指标差异的验证，不是独立的疾病诊断敏感度/AUROC 实验。

## 优点

- 提供门诊可操作流程、明确的手机与 IMU 同步参考以及可核验的 PosePipeline 与统计脚本。
- 不把良好 3D pose 直接等同于临床有效性，而报告 gait speed、event timing 等具体临床量及一致性。
- 使用左右镜像重新推理的简单策略审计姿态算法侧别偏差；对视频/患者数据强调本地处理。

## 局限

- **单中心、小规模（34 人），仅一类头晕临床群体**；尚未证明在 ASD、脊柱畸形、帕金森或真实复杂门诊跨中心有效。
- 固定单手机、直线走路、选取稳态片段；论文明确排除 turning，对遮挡、低光、被遮挡腿、动态相机仍存在边界。
- 参考为可穿戴 IMU，而不是逐帧 marker-based 全 3D joints/force plate GT；良好时空 gait parameters 不等于准确的全身三维 mesh。
- **数据统计细节：** 正文总 laps 269 与两个子组 187+77=264 不一致，需用官方代码或补充材料进一步确认。
- **推断：**以 healthy-data pose prior 恢复病理人体时可能产生结构或动作偏置；应额外核查侧别对称性、关键临床 ROI、行走速度与疾病严重度的相互影响。

## 个人评价

**推断：**相比只报告 ASD/Non-ASD accuracy 和 AUROC，本文更适合作为“临床特征确实测准了吗”的外部验证范式。模型是否能区分患者，并不能直接说明其关节角、速度和时间参数可信。值得将其视作 clinically meaningful measurement baseline，而不是直接的 ASD 诊断分类 SOTA。

## 与我的研究关联

**推断：**对现有侧视 ASD / ClinicalGated 研究，可优先复现 clinical gait event / cadence / step duration 的独立标注和传感器或人工对照，以 patient-disjoint 统计每人的运动参数误差。用 mirror-consistency audit 检查临床 ROI gate 是否偏向画面某一边，再以 group-aware mixed models 对 age、gender、height 和 walking speed 做混杂控制。由于论文为 frontal-view 直线路径，不能将其官方误差数值直接外推至侧视的脊柱畸形与不同 camera setup。

## 后续阅读

- 优先阅读 Fig. 1/3、Methods: Data acquisition / Data analysis / Statistics 与 Tables 1–3，复现官方 GaitValidation 中 PosePipeline.ipynb、Bland–Altman 和 Linear Mixed Model 脚本。
- 对照 [GAVD](2025-gavd-clinical-gait-video-dataset.md)、[DrGait](2026-drgait-biomechanically-grounded-clinical-gait.md) 和 [OpenCap](../sports-biomechanics/2023-opencap-human-movement-dynamics.md)，分离 video-level diagnosis、运动学测量一致性与临床生物力学解释。
- 重点补测侧视、长时视频、病理患者、左右不对称、转身、遮挡与校准不确定性；明确统计单元是 laps 还是 participants。

## 核验来源

- https://www.nature.com/articles/s41598-025-34912-5
- https://doi.org/10.1038/s41598-025-34912-5
- https://github.com/mea-lab/GaitValidation
- https://github.com/mea-lab/VisionMD-DesktopApp
- https://www.visionmd.ai/
