---
title: "Quantitative Analysis of Elite Giant Slalom Turn Technique Using Broadcast Video Images and 3D CAD Model Matching"
authors: "Kosuke Nakazato, Shun Yoshikawa, Yohei Hoshino, Soichiro Suzuki"
venue: "Applied Sciences 2026, 16(14), 7041"
year: 2026
reading_date: 2026-11-11
status: skimmed
tags:
  - sports-biomechanics
  - alpine-skiing
  - broadcast-video
  - turn-phase
  - center-of-mass
  - kinematics
  - cad-model-matching
---

# Quantitative Analysis of Elite Giant Slalom Turn Technique Using Broadcast Video Images and 3D CAD Model Matching

## 基本信息

- **作者：** Kosuke Nakazato, Shun Yoshikawa, Yohei Hoshino, Soichiro Suzuki
- **会议/期刊：** Applied Sciences, 16(14), 7041；2026-07-14 正式在线发表。
- **年份：** 2026
- **阅读日期：** 2026-11-11
- **阅读状态：** skimmed
- **论文状态：** 2026 年正式同行评议期刊论文；近期研究补读，不是当天新论文。
- **标签：** sports-biomechanics, alpine-skiing, broadcast-video, 3d-cad, com, turn-phases
- **价值类型：** Method Module / Related Work
- **阅读优先级：** 中高（滑雪相位定义与运动学指标可直接借鉴；样本规模有限）
- **论文：** https://www.mdpi.com/2076-3417/16/14/7041
- **DOI：** https://doi.org/10.3390/app16147041
- **代码：** 暂无（未核验到作者专属可执行仓库）
- **数据集：** 暂无（出版方声明比赛转播素材因版权/伦理限制不公开）
- **项目主页：** 暂无（出版方全文即正式入口）

## 一句话总结

使用人工辅助的 15 段刚体 3D CAD 姿态匹配，从未经相机标定的 25 FPS 赛事转播视频提取大回转滑雪转弯五阶段、重心和下肢关节运动学，展示真实竞技录像可做可解释技术分析，但仅限 6 名运动员的单次转弯、且缺乏外部动态真值。

## 研究问题与动机

高山滑雪真实世锦赛无法自由布置 mocap、同步 GNSS/IMU 或力台，很多技术分析停留在计时和视觉判断。作者将已建立的 3D CAD model-matching 运用到 2021 FIS 世锦赛的公开转播视频，观察顶尖运动员与一名日本精英滑雪者在转弯时间、edge inclination、COM 和双膝/髋角方面的探索性差异，而非训练一个 pose learning benchmark。

## 核心方法

1. **数据截取（Section 2.1–2.2）：** 选取 2021-02-19 男子大回转第一轮第 47–49 旗门间的可见单次转弯；条件要求少遮挡、全转弯落在同一机位、坡况相对稳定；原视频 25 FPS，未知内参、焦距和真实 camera extrinsics。
2. **15 段 3D CAD model（Section 2.3–2.4）：** Inventor Professional 建立头、躯干、骨盆、双上臂、前臂、手、股段、小腿、足组成的刚体，人体尺寸用群体测量学统计，雪板长度统一假设 193 cm。操作者逐帧以视频叠加方式对齐骨盆、髋、膝、足、雪板及上身，人工检查解剖合理性和时间连续性，不使用 smoothing。
3. **变量定义（Section 2.6–2.7）：** 由 CAD 计算髋/膝关节角、edge angle 与 COM 在雪面坐标中的水平/垂直位置；COM 使用统一的 segment density 假设。转弯划分 NP（neutral）→ ES（edging start）→ LS（load start）→ TM（turn maximum）→ EF（edging finish）→ 下一 NP；分别比较关键相位时长和归一化波形。
4. **可靠性与描述统计：** 同一操作者对 10 个代表帧重复匹配，报告 ICC、重复 RMSE；5 位世界前五（WT）提供 mean ± SD，一名日本滑雪者（JP）提供观测值与相对于 WT 的 z-score。因单例比较，没有推断统计检验。

## 数据集与评价指标

- **来源和样本：** 2021 FIS Alpine World Ski Championships 男子 giant slalom，WT 前 5 名 + JP 1 名，6 名滑雪者各选一个完整转弯片段；仅一条赛事赛道的局部单机位 25 FPS 视频。
- **输入/输出：** 单机位 2D 广播视频 + 15 segment CAD 人体模型 → 手工拟合的 3D 运动、edge angle、膝髋角、相对雪面 COM 与转弯五阶段。
- **评价：** 重复拟合 ICC、角度 RMSE、时间（秒）、edge angle（°）、COM（mm）、膝髋角（°）；比赛 WT/JP 的描述性对照。不是全自动训练/测试系统，故不存在与深度 HMR SOTA 的公平 MPJPE baseline。
- **数据与代码：** 出版方声明比赛视频相关数据不公开；没有核验到官方训练/推理代码或下载集。

## 主要结果

- **重复可靠性（Section 3.1）：** ICC **0.81–0.97**；两次 3D 匹配间角度 RMSE **3.87°**、均值差 **0.015°**。这些是同操作者重复精度，不等于与独立 mocap 的当前比赛场景外部绝对误差。
- **转弯时间（Table 2）：** WT 完整周期 **1.43±0.07 s**，JP **1.48 s**；关键 LS→TM 为 WT **0.58±0.10 s** vs JP **0.68 s**。
- **edge / COM（Table 3）：** TM 峰值 edge angle WT **68.9±1.5°** vs JP **67.4°**；LS 时 COM 高度 WT **683.5±59.2 mm** vs JP **605.9 mm**。作者观察 WT 起弯时抬升并向弯内移动 COM，而 JP 更早降低 COM。
- **下肢（Section 3.4）：** JP 在 ES 阶段双侧髋膝角更小，提示更大早期屈曲，但没有多人统计显著性或因果证明。

## 优点

- 真实世界级赛事而不是室内滑雪模拟器；直接对齐转弯技术中的 timing、edge、COM、关节角，适合作为视频运动学任务定义来源。
- 五阶段定义和局部 snow-plane 参考，有利于把 3D pose 误差转化为可解释的滑雪技术量。
- 不夸大单人比较：作者明确采用描述统计，并公开重复操作者拟合可靠性。

## 局限

- 6 人、每人仅一个选定转弯；仅一个日本运动员，不能推广到国别或竞技水平的总体差异。
- 25 FPS 的时间分辨率约 ±0.02 s，转弯相位差很小；未知 camera intrinsics/pose，透视畸变可能污染 3D 与 COM。
- 需要人工 CAD matching；不是真正自动、免标定的 metric 3D 重建；统一密度假设和非个体化 segment lengths 影响 COM 可信度。
- 没有直接测量 skiing velocity、world racing line、GRF/pressure，也无同赛事独立逐帧 mocap 验证；不可用此论文证明单目视频自动运动学方法的准确度。
- 公开视频与数值数据不发布，代码也未核验到；利益冲突：论文声明作者无利益冲突。

## 个人评价

**推断：**论文作为 ski-specific evaluation protocol 比作为新 3D HPE baseline 更有价值。COM 与 edging 角度定义可让“数值更平滑”与“滑雪技术更正确”分开评价。需要先统一坡面法向、人体体段和滑雪板局部坐标，并对角度/COM 添加不确定度，避免人工拟合的先验被当作真值。

## 与我的研究关联

**推断：**你的双视角/双 360° 长视频系统已有 3D skeleton 和时序融合，可以复现本文 NP/ES/LS/TM/EF segmentation，在每个相位记录 pelvis/COM vertical、lateral shift、edge angle 近似、knee/hip ROM 与 angular velocity。将 15 关节轨迹转到 body-centered 坐标和 slope frame，对比单视角、平均融合、per-axis reliability 与 temporal refinement。对 12 台 RX0 II 的室内/室外验证，优先增加 3D 参考、坡面标定和鞋垫压力/IMU，以实证检验这些变量是否准确。

## 后续阅读

- **重点：** Section 2.2–2.7（选段/CAD/变量/阶段定义）、Tables 2–4（phase、edge、COM、关节角）、Section 4.4（局限）。
- **实验建议（推断）：** 视频标注至少两位人工评估者的转弯事件与关节角；比较 25/30/60 FPS 下相位误差，检查人/雪板遮挡、camera panning、坡面法向误差和不同滑雪者的泛化。
- **对比阅读：** [SkiTechCoach](../sports-biomechanics/2025-skitechcoach-alpine-skiing-multimodal.md)、[OpenCap](../sports-biomechanics/2023-opencap-human-movement-dynamics.md)、[AthleticsPose](../sports-biomechanics/2025-athleticspose-authentic-sports-motion.md)。
