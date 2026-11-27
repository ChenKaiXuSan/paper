---
title: "BST: Badminton Stroke-type Transformer for Skeleton-based Action Recognition in Racket Sports"
authors: "Jing-Yuan Chang"
venue: "CVPR 2026 Workshops (CVsports)"
year: 2026
reading_date: 2026-11-28
status: skimmed
tags:
  - motion-understanding
  - sports
  - skeleton
  - cross-attention
  - object-trajectory
  - fine-grained-action-recognition
---

# BST: Badminton Stroke-type Transformer for Skeleton-based Action Recognition in Racket Sports

## 基本信息

- **作者：** Jing-Yuan Chang
- **会议/期刊：** CVPR 2026 Workshops（CVsports），pp. 9889–9898
- **年份：** 2026
- **发表时间：** 2026-06；arXiv 首次提交 2025-02-28
- **阅读日期：** 2026-11-28
- **阅读状态：** `skimmed`
- **标签：** `motion-understanding`、`sports`、`skeleton`、`cross-attention`、`object-trajectory`
- **论文：** [CVF](https://openaccess.thecvf.com/content/CVPR2026W/CVsports/html/Chang_BST_Badminton_Stroke-type_Transformer_for_Skeleton-based_Action_Recognition_in_Racket_CVPRW_2026_paper.html)、[arXiv:2502.21085](https://arxiv.org/abs/2502.21085)
- **代码：** https://github.com/Va6lue/BST-Badminton-Stroke-type-Transformer
- **数据集：** [ShuttleSet](https://github.com/wywyWang/CoachAI-Projects/blob/main/ShuttleSet/README.md)；BadmintonDB、TenniSet 的论文实验已核验，独立官方入口本次暂无核验
- **项目主页：** 暂无
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 中高

## 一句话总结

利用球轨迹作为关键运动证据，通过双人骨架、球场位置与球轨迹的 cross-attention 和条件门控实现细粒度球拍运动识别。

## 研究问题与动机

通用骨架动作识别主要区分差异较大的日常动作，难以识别短暂、相似的挥拍类别。仅从人体姿态无法充分区分对手干扰、假动作及实际击球结果，因此作者将球的运动和球场位置纳入分类。

## 核心方法

1. 依据击球帧组织 clip，保留目标动作与球轨迹相关时段，减少无关动作。
2. 用 HPE 提取人体骨架、TrackNetV3 提取球轨迹，并使用球场几何确定运动员位置。输入包括 joints/bones、shuttlecock position、player position。
3. Cross Transformer 使用人体 pose/position latent 作为 Q，球轨迹 latent 作为 K/V；随后沿时间处理各球员相关的轨迹表示，由 MLP 分类。
4. Pose Position Fusion (PPF) 用 MLP 根据位置生成乘性系数调制 joints；Clean Gate (CG) 抑制对方击球干扰；Aim Player (AP) 通过与整体轨迹的余弦相似性确定两名球员的融合贡献。

## 数据集与评价指标

- ShuttleSet 原始 44 场、27 位运动员、36,492 次击球；清洗后 40 场、33,481 次击球，按 30/5/5 场划分 train/val/test；含 25 类合并版本与 35 类版本。
- BadmintonDB：清洗后 8 场、7,658 次击球，来自两名球员，按类别 8:1:1 划分。TenniSet：5 场、3,569 次击球，沿用数据集划分。
- 评价 clip-level Accuracy、Macro-F1、Min-F1、Top-2 Accuracy；对照 ST-GCN、BlockGCN、SkateFormer、ProtoGCN、TemPose 等。
- 以下具体数值来自可读取的 arXiv 正文表格；CVF 官方条目确认正式 workshop 发表。

## 主要结果

- Table 2（ShuttleSet 25 类，自定义 clipping）：TemPose-TF Accuracy 82.35%、Macro-F1 80.28%；BST-CG-AP 83.22%、80.97%。BST-0 最低类别 F1 为 58.22%，高于完整模型的 57.62%。
- Table 1（固定宽度）：ST-GCN Accuracy 77.58%，TemPose-SF 81.00%，BST-CG-AP 82.54%。不同剪辑方式、输入模态需分别比较。
- Table 3（25% ShuttleSet 训练，固定宽度）：TemPose-TF Accuracy 66.36%，BST-AP 70.38%，BST-CG-AP 69.24%；小样本下组合模块不必然最优。
- Appendix B.2：该研究中估计 3D joints 的分类表现不如 2D joints；作者将其与通用 HPE 的领域偏差和 lifting 误差联系起来。

## 优点

- 将球轨迹视为主要证据而不只是末端拼接信息，人体 Q 与轨迹 K/V 构成明确交互。
- 同时测试 clip 策略、不同模态、门控、较少训练数据和其他球拍运动数据。
- 官方代码提供权重与预处理说明，便于建立复现基线。

## 局限

- 作者指出依赖准确击球帧检测及球轨迹追踪；事件定位错误会污染输入。BadmintonDB 仅两名球员，按类别切分不等于运动员独立验证。
- **推断：**比赛级划分不能自动保证运动员独立；球轨迹在击球后可含明显结果信息，若改做在线预测必须禁止未来帧。
- **推断：**3D 表现较差仅说明当前 HPE/数据条件，不能推出所有 3D 运动表征均不如 2D。球场投影与滑雪坡面几何不相同。

## 个人评价

**推断：**最值得借鉴的不是增加 Transformer，而是显式定义运动主体与装备/环境轨迹的证据关系。PPF 是乘性条件调制，与 FiLM 的 scale-only 特例形式相似，但不等价于完整 gamma/beta FiLM。必须用参数匹配的 concat、gate、cross-attention 做对照。

## 与我的研究关联

**推断：**滑雪可将人体 joints/bones 作为 Q，将雪板、雪杖、坡面或转弯轨迹作为 K/V，按实际转弯阶段裁剪；驾驶员研究可把头部 3D 运动与手/方向盘或交通事件特征作条件交互，但不能把头姿等同于 gaze。临床步态可用步态相位与 pelvis trajectory 调制局部骨架特征，比较 2D、3D 与 RGB/flow，确保患者独立测试，防止将疾病标签或检测误差当作条件捷径。

## 后续阅读

- Sections 3.1、3.3.1–3.3.4：复现事件中心剪辑、Cross Transformer、PPF、CG/AP。
- Tables 1–3 与 Appendix B：对照样本效率、最低类 F1、2D/3D 差异。
- 结合已有 SLiM、PhaseMix、ClinicalGated 与 CanonFuse3D：先比较 pose-only、trajectory-only、concat、scale-only FiLM、完整 FiLM、cross-attention；保持同样时间窗口和患者/运动员划分。
