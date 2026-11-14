---
title: "EyeCue: Driver Cognitive Distraction Detection via Gaze-Empowered Egocentric Video Understanding"
authors: "Lang Zhang, JinYi Yoon, Matthew Corbett, Abhijit Sarkar, Bo Ji"
venue: "IJCAI 2026 Main Track, pages 3737–3745"
year: 2026
reading_date: 2026-11-15
status: skimmed
tags:
  - motion-understanding
  - driver-monitoring
  - multimodal-learning
  - eye-gaze
  - interpretability
  - domain-generalization
---

# EyeCue: Driver Cognitive Distraction Detection via Gaze-Empowered Egocentric Video Understanding

## 基本信息

- **作者：** Lang Zhang, JinYi Yoon, Matthew Corbett, Abhijit Sarkar, Bo Ji
- **会议/期刊：** IJCAI 2026 Main Track, pp. 3737–3745；arXiv 首次提交 2026-05-08；正式会议论文，正式在线具体日期暂无核验
- **年份：** 2026
- **阅读日期：** 2026-11-15
- **阅读状态：** `skimmed`
- **标签：** `motion-understanding`, `driver-monitoring`, `multimodal-learning`, `eye-gaze`, `domain-generalization`
- **论文：** https://doi.org/10.24963/ijcai.2026/416 （正式全文：https://ijcai.org/proceedings/2026/0416.pdf；arXiv：https://arxiv.org/abs/2605.07859）
- **代码：** https://github.com/langzhang2000/EyeCue
- **数据集：** https://drive.google.com/drive/folders/1m3Xh8aVtOiX9IGyhLH9TlP4nX6gSrWi3?usp=sharing （作者官方 GitHub README 公布的 CogDrive 下载入口）
- **项目主页：** 暂无（官方代码仓库兼作项目入口）
- **论文状态：** IJCAI 2026 正式发表；近期相邻专题论文
- **价值类型：** Dataset / Method Module / Baseline / Related Work
- **阅读优先级：** 中高
- **事实来源：** https://ijcai.org/proceedings/2026/416 ；https://ijcai.org/proceedings/2026/0416.pdf ；https://github.com/langzhang2000/EyeCue

## 一句话总结

EyeCue 将驾驶员 gaze coordinates 与第一视角驾驶视频进行 gaze-driven patch selection 和 cross-attention，以显式建模“视线所落之处的道路语义”，在多数据集认知分心分类任务上优于只用 gaze 或只用视频的模型。

## 研究问题与动机

认知分心与手部操作、视线明显离路不同：驾驶员可能注视前方却未正确处理道路信息。仅依赖眼动轨迹难以理解其观看内容，仅依赖 RGB 驾驶场景又难以推断注意力分配。论文将目光轨迹与第一视角环境语义建立时空对应，并为该问题创建跨场景基准 CogDrive。

## 核心方法

1. **视频编码器：** 采用 TimeSformer（K600 预训练）或 VideoMAE（K400 预训练），得到 clip-level token 和每帧 visual patch tokens。
2. **眼动编码器：** 输入每帧归一化 gaze 位置序列，使用轻量 Transformer 得到 global gaze token 与 frame-level gaze tokens。
3. **Gaze-Driven Semantic Query（GDSQ）：** 按每帧 gaze 坐标从视频特征图选择对齐 patch token，将 gaze tokens 作为 Query、注视位置附近 video tokens 作为 Key/Value，采用两层 cross-attention 形成 gaze-context interaction。
4. **决策：** 拼接视频 CLS、gaze CLS 和 GDSQ CLS，再由 MLP 输出 attentive / cognitively distracted 二分类。模型使用 gaze 的几何位置，而不直接处理眼球虹膜图像。

## 数据集与评价指标

- **CogDrive：** 由 DR(eye)VE（1,894 clips）、BDD-A（634）、DADA-2000（627）、TrafficGaze（507）构成，总计 **3,662 条、每条 16 帧**；其中 attentive **2,853**、distracted **809**。
- **标签与训练：** 两名标注者独立标注、专家审核，论文报告 inter-annotator agreement >98%。训练比较采用全部 809 distracted clips 和随机抽取的 809 attentive clips，按 70/30 切分。该类别平衡后的训练/测试子集不等于完整 3,662 条原始分布。
- **输入：** 第一视角 RGB 视频 + 同步的 gaze (x, y) 点；**输出：** clip-level cognitive-distraction binary label。
- **基线：** gaze-only SVM、TimeSformer、VideoMAE、EgoVideo、InternVideo2、Video-LLaVA、VideoLLaMA3、GazeGPT、Voila-A、GazeVQA、GazeLLM 等共 11 个 baseline，6 个模型家族。
- **评价：** Accuracy、distracted-class F1、leave-one-dataset-out（对 TrafficGaze 因严重不平衡未单独进行）。

## 主要结果

- **Table 2：** 全 CogDrive 设置下 EyeCue + TimeSformer-K600 **Accuracy 74.38%、F1 0.74**，优于基线 VideoMAE-K400 **67.21%** accuracy（+7.17 个百分点）；EyeCue + VideoMAE 为 **70.83% / 0.68**。
- **Leave-one-dataset-out：** EyeCue 在 BDD-A / DADA-2000 / DR(eye)VE 的 accuracy 为 **65.24% / 68.29% / 60.20%**，仍存在跨数据集下降。
- **Table 4 消融：** gaze encoder only **54.13%**、video encoder only **67.53%**、直接 video+gaze fusion **72.31%**，完整 GDSQ+fusion **74.38%**。
- **Table 3：** 16 帧且每帧只选择 gaze 所在的 1 个 video patch（h=1）效果最好，为 **74.38%**；更大局部 patch 数并非一定提升。
- **Table 5 场景：** evening accuracy **61.36%**，低于夜间的 **79.73%**，提示某些条件下存在分布敏感性。

## 优点

- 把眼动的“注视坐标”与环境语义绑定，采用可分析的 cross-attention，而不是简单全局向量拼接。
- 明确比较模型家族、输入模态、gaze patch 数量以及跨数据集协议。
- 作者 GitHub 已公开训练代码与 CogDrive 数据入口，数据字段和运行命令有明确说明。

## 局限

- CogDrive 是四个原本并非全部针对认知分心设计的数据集的**二次标注**；gaze-context 规则与真实心理状态之间存在 construct-validity 风险（后半句为**推断**）。
- 作者明确指出复杂交通环境、低质量 gaze 会让性能下降；TrafficGaze 的 distracted 占比仅 5.13%，因此未列入完整 leave-one-dataset-out。
- **推断：** 使用 clip-level 随机切分可能存在同一原始驾驶者/连续视频片段的相关性，若论文/代码没有明确 participant/video-disjoint，需要另外做严格分组验证。
- 本研究是 cognitive distraction detection，不是 3D 头部运动恢复；不能直接把其 Accuracy 视为 Driver-CHM3D 的 yaw/pitch 测量性能。

## 个人评价

**推断：** GDSQ 比强行把所有视觉 patch 用眼动 heatmap 限制更可取，因为保留 global scene CLS 同时建模 fixation-level semantics。相比直接 RGB+gaze concat，作者有清楚的消融，但 74.38% accuracy 并不代表模型已经能应对真实行车部署时的 domain shift、标签噪声和个体差异。

## 与我的研究关联

**推断：** 对驾驶模拟器下的头部扫描 / 视觉受限驾驶分析，可将 video contextual scene、head yaw/pitch trajectory 与 gaze（若有独立眼动仪）建模为互补模态，仿照 GDSQ 用 head direction 查询道路 ROI；需要明确 head orientation ≠ eye gaze。若无独立眼动标注，先构建 `RGB-only / head-pose-only / RGB+head-pose cross-attention` baseline，不要将估计头向当真实 gaze。对于临床步态视频，也可以借鉴 ROI-guided semantic query：用可信临床关节点/ROI 聚合局部视觉语义，并验证在少样本与跨主体划分下的增益。

## 后续阅读

- 重点阅读 **Sections 3–5，Figure 2，Tables 1–5**；复现 GDSQ 的 gaze patch sampler 和 cross-attention，并做 `global fusion` vs `ROI-guided cross-attention` 对照。
- 阅读官方 `new_train.py` 及 README 中 gaze 数据格式，检查数据拆分是否 video-/driver-disjoint、gaze 噪声鲁棒性与 prompt/训练条件。
- 与已有 [Drive&Act](../motion-understanding/2019-drive-and-act-driver-behavior.md) 和 [DriverGaze360](../motion-understanding/2026-drivergaze360-omnidirectional-attention.md) 区分任务：fine-grained behavior、对象级 attention 与 cognitive distraction。
