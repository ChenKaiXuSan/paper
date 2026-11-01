---
title: "AthleticsPose: Authentic Sports Motion Dataset on Athletic Field and Evaluation of Monocular 3D Pose Estimation Ability"
authors: "Tomohiro Suzuki, Ryota Tanaka, Calvin Yeung, Keisuke Fujii"
venue: "MMSports @ ACM Multimedia 2025 (author-reported); arXiv:2507.12905"
year: 2025
reading_date: 2026-11-02
status: skimmed
tags:
  - sports-pose
  - outdoor-athletics
  - multiview-capture
  - markerless-mocap
  - motion-kinematics
  - domain-shift
---

# AthleticsPose: Authentic Sports Motion Dataset on Athletic Field and Evaluation of Monocular 3D Pose Estimation Ability

## 基本信息

- **作者：** Tomohiro Suzuki, Ryota Tanaka, Calvin Yeung, Keisuke Fujii
- **会议/期刊：** MMSports @ ACM Multimedia 2025（作者官方仓库明确标注；出版方页面此次未成功访问），arXiv 预印本
- **首次提交：** 2025-07-17（arXiv:2507.12905）
- **年份：** 2025
- **阅读日期：** 2026-11-02
- **阅读状态：** `skimmed`
- **标签：** `sports-pose`、`outdoor-athletics`、`multiview-capture`、`markerless-mocap`、`motion-kinematics`、`domain-shift`
- **论文：** https://arxiv.org/abs/2507.12905
- **DOI：** 暂无（本次无法从出版方直接核验正式出版 DOI）
- **代码：** https://github.com/SZucchini/AthleticsPose
- **数据集：** https://github.com/SZucchini/AthleticsPose/releases
- **项目主页：** 暂无（官方代码仓库已单列）
- **论文状态：** 作者标注已发表于 MMSports @ ACM MM 2025；可直接核验 arXiv 原文与作者发布资源
- **主要价值类型：** Dataset / Baseline / Related Work
- **阅读优先级：** 高（2025 年重要真实体育动作 GT / kinematics 补读）

## 一句话总结

构建真实田径竞技动作的室外八机位人体姿态数据集，证明在一般/模仿动作上训练的 3D pose 模型难以迁移到真实高速运动，并且位置误差小不意味着速度与技术动作指标可靠。

## 研究问题与动机

通用 HPE 数据集主要是日常活动；部分体育数据的动作是模拟/表演，与竞技训练中的高速、加速度、遮挡和尺度变化仍有差异。论文同时质疑仅用 MPJPE/P-MPJPE 评价 sports kinematics 的充分性，提出真实田径场采集与从预测 3D joints 派生的膝角/驱膝速度独立验证。

## 核心方法

1. **真实竞技动作采集**：户外田径场，8 台 Qualisys Miqus Video HD 高速相机做硬件同步与标定，使用 Theia3D 光学 markerless mocap 生成 3D 关节参考，非穿戴 marker-based ground truth。
2. **数据质量控制**：对 83 个关节位置做完整性和骨长一致性检查，随后用已标定内外参将 3D joints 投影到各视角；面向常见 HPE 训练时映射为 17 个 COCO keypoints。
3. **估计模型**：ViTPose 提取 2D；MotionAGFormer 从 **81 帧 2D keypoint 序列**回归 3D，训练时按 subject-disjoint 划分；与 Human3.6M、AthletePose3D 训练的模型比较，并区分 GT 2D / detector 2D / detector fine-tuning。
4. **运动学验证**：从髋-膝-踝估计支撑腿膝角，从双膝间距时序估计最大 knee-drive velocity；同时按 view 和人体框相对尺寸分层比较误差。

## 数据集与评价指标

- 原文数据收集描述为招募 **24 名**大学运动员，摘要及公开数据统计为 **23 名运动员**；应保留这一口径差异，不推断剔除原因。
- 真实田径运动 **8 类**：starting dash、sprint、distance running、racewalk、hurdle、shot put、discus、javelin；约 **500,000 帧**、**83 个 3D joints**、8 路同步标定视频；javelin 因只有 1 人不纳入所述分类评估。
- 输入：单机位 2D joints（GT 或 ViTPose DET）及 81 帧时窗；输出：相对于人体根节点的 3D joints 及衍生 kinematic variables。
- 指标：2D mAP、3D MPJPE/P-MPJPE（mm）、按关节/动作/view/scale 分层误差，以及膝角 RMSE 与速度指标的预测偏差。
- **数据可用性：**作者 GitHub 提供数据归档、脚本和 checkpoints；明确写明**原始视频文件不计划公开**，下载的包不能等同于全部原始同步视频。

## 主要结果

- Table 2：Human3.6M 训练的模型 **149.38 mm MPJPE / 98.80 mm P-MPJPE**；AthleticsPose 训练（同类 DET pre-trained 输入）为 **41.99 / 31.40 mm**。用同一套 AthleticsPose 训练后继续 fine-tune ViTPose，MPJPE 可到 **40.13 mm**。
- 在 **GT 2D 输入**条件下，AthleticsPose 训练的 3D 模型 **29.45 mm MPJPE**，而 AthletePose3D 训练模型 **115.91 mm**；这项比较不能与 detector 输入的 41.99 mm 直接混算。
- Table 4/5：侧视 **31.42 mm**，正/背视 **36.34 mm**；人体框大时 **33.27 mm**，小时 **42.63 mm**，展示视角/目标尺度影响。
- 由预测 3D joints 推导的支撑腿膝角 RMSE **6.43°**（侧视 4.82°、正/背视 7.18°）；最大 knee-drive velocity 存在明显预测偏差，位置准确不能直接保证速度可靠。
- 2D ViTPose fine-tuning 将 mAP 从 **0.875** 提升到 **0.979**。

## 优点

- 同步、标定、多视角且在真实户外竞技条件下采样，提供一般数据集和模拟体育动作未覆盖的高速动作域。
- 区分 2D GT / detector 精度和 3D lifting 精度，支持衡量不同误差来源。
- 评价延伸到膝角与速度，并分析 view / scale / action，而不止一个总体 MPJPE。
- 数据/训练脚本/模型权重由作者公开，便于重现实验的一部分。

## 局限

- 真实人数只有约 23 名、主要大学生田径运动员，领域与环境跨度有限，不代表滑雪、临床步态或其它体育项目。
- 原始视频不公开，限制对 2D 检测、相机几何或新的 end-to-end video HMR 的完整复现。
- 3D “GT” 由 Theia3D **markerless** 系统产生，非独立 marker-based 参考；**推断：**标定和骨长检查不能替代标记式系统/测力/IMU 等对高速动作的独立误差验证。
- 静态布置的多机位仅用于生成 GT；论文主要评价单视角 camera-relative pose，未测试自由移动相机、ERP 或 world-scale camera trajectory。

## 个人评价

这篇对研究设计最重要的启示不是 MotionAGFormer 的某个单独模块，而是**真实动作数据域、相机视角/像素尺度和下游运动学指标**会独立影响结论。较低 MPJPE 不能替代角度、速度/加速度及技术指标的验证。尤其要区分 gt-derived 3D labels 与真正独立的 biomechanics GT。

## 与我的研究关联

- **滑雪 dual-view：**把专业/普通滑雪动作的真实域作为重要实验因子；按视角、人物像素高度、遮挡、转弯 phase 和动作速度报告 per-axis MPJPE 或人工 2D 误差。可借鉴 8 相机硬件同步/标定与骨长质量控制来设计 12 台 RX0 II 的采集。
- **动态评价：**在现有 speed/jerk/bone CV 外增加膝角、躯干倾斜、角速度、左右不对称及 phase-aware 评估；尽可能用独立标注/IMU/marker-based GT 验证，而非只比不同估计器的一致性。
- **临床 ASD 步态：**`subject-disjoint` 划分、ROI/关节分层错误、相机视角与人物尺度 sensitivity 可直接迁移，但田径运动员的训练域不等于脊柱病理人群。
- **优先阅读/复现：**Sec. 3.1–3.2 多机位数据采集/QC；Sec. 5.4 Table 2–5 与膝角/速度测试。

## 后续阅读

- AthletePose3D、SportsPose、BadmintonGRF、MuyBridge、OpenCap Monocular、MotionAGFormer 与真实运动学验证工作。
- **推断：**验证 `synthetic skiing only → authentic skiing fine-tune → cross-subject/cross-view test`，分开报告 3D joint、kinematic variable、camera metric accuracy 和真实比赛/练习 domain gap。
- 一手核验：arXiv:2507.12905 全文和作者官方 https://github.com/SZucchini/AthleticsPose（含 Releases、数据许可与原始视频不公开说明）；正式出版 DOI 未用无法核验的二手页面填充。
