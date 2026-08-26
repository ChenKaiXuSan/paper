---
title: "WristP²: A Wrist-Worn System for Hand Pose and Pressure Estimation"
authors: "Ziheng Xi, Zihang Ao, Yitao Wang, Mingeze Gao, Wanmei Zhang, Jianjiang Feng, Jie Zhou"
venue: "CHI 2026"
year: 2026
reading_date: 2026-08-27
status: skimmed
tags:
  - hand-pose
  - pressure-estimation
  - wearable
  - hci
  - fisheye
  - tactile-interaction
---

# WristP²: A Wrist-Worn System for Hand Pose and Pressure Estimation

## 基本信息

- **作者：** Ziheng Xi, Zihang Ao, Yitao Wang, Mingeze Gao, Wanmei Zhang, Jianjiang Feng, Jie Zhou
- **会议/期刊：** CHI 2026
- **年份：** 2026
- **阅读日期：** 2026-08-27
- **阅读状态：** `skimmed`
- **标签：** `hand-pose`, `pressure-estimation`, `wearable`, `hci`, `fisheye`, `tactile-interaction`
- **论文：** https://arxiv.org/abs/2603.00606
- **代码：** https://github.com/zhenqis123/WristPP_code （官方页面显示仍在发布准备中）
- **数据集：** 暂无（官方项目页标注 Coming Soon）
- **项目主页：** https://zhenqis123.github.io/WristPP/
- **DOI：** https://doi.org/10.1145/3772318.3790626

## 一句话总结

WristP² 用单个腕部 180° fisheye RGB 相机同时恢复 3D hand mesh 与逐顶点接触压力，把“姿态”和“力”统一为可穿戴视觉感知问题，为智能触觉、手部交互和 biomechanics-aware vision 提供了很有启发性的多任务框架。

## 研究问题与动机

传统手部姿态估计通常只回答“手在哪里、手指怎么弯曲”，而真实交互还需要知道“手是否接触物体、接触在哪里、施加多大压力”。压力传感器、手套或头戴相机可以提供部分信息，但往往增加设备负担，且很难同时获得自然手部运动和密集压力分布。

WristP² 的目标是在一个低成本腕部设备中同时恢复 **3D hand pose/mesh 与 pressure distribution**。作者使用超广角 fisheye 相机观察整只手，通过统一网络把视觉外观、关节几何与接触压力联系起来，使系统既可服务于 pose tracking，也可用于 pressure-aware interaction。

## 核心方法

硬件端使用腕部安装的 **180° FOV fisheye RGB camera**，配合 Raspberry Pi Zero 2W，整体成本约 50 美元。宽视场能够在手指大幅运动时保持较完整的可见性。

模型端包含两条相互关联的任务路径：

1. **Hand pose / mesh representation**：使用 Hand-VQ-VAE 将手部几何压缩到离散 codebook 表示，再由 ViT 根据腕部 RGB 图像预测与关节对齐的 tokens，从而恢复 3D hand mesh。
2. **Pressure estimation**：通过结合 hand geometry / extrinsic information 的特征和 cross-attention 分支，预测 mesh 顶点上的 pressure / contact 信息。
3. **Joint learning**：压力不是独立的二维热图，而是与恢复出的 3D hand surface 对齐，因此可以直接表达哪个手部区域正在接触以及受力大小。

这种设计的关键点是把 **geometry 与 interaction force 共用人体表面表示**，比“先做姿态、再单独判断接触”更适合形成统一的身体交互模型。

## 数据集与评价指标

作者采集了约 **133,000 帧、20 名参与者**的数据，覆盖 **48 种平面接触手势**和 **28 种 mid-air gestures**。采集系统结合 FZMotion infrared motion capture、Kinect 和 Sensel Morph pressure sensor，并通过多目标优化获得 3D hand mesh 与逐顶点 pressure supervision。

主要评价包括：
- 3D pose：MPJPE、Mean Joint Angle Error；
- contact / pressure：Contact IoU、Volumetric IoU、pressure MAE 等；
- 系统交互：触摸输入效率、多指压力控制、应用任务成功率与主观疲劳。

论文还通过三组用户研究及应用实验验证设备不仅能离线预测姿态和压力，也能用于实际交互。

## 主要结果

WristP² 报告 **2.9 mm MPJPE** 与 **3.2° Mean Joint Angle Error**。压力相关指标达到约 **71.2% Contact IoU**、**61.8% Volumetric IoU**，在 foreground positive-pressure vertices 上的 pressure MAE 为 **10.4 g**。

论文的系统实验显示，设备可以支持 multi-finger pressure interaction，并在 Whac-A-Mole 类型应用中相较 head-mounted baseline 获得更高任务成功表现和更低手臂疲劳。作者因此强调 wrist-mounted wide-FOV view 对自然手部交互具有实际优势。

## 优点

- **Pose + pressure 的联合问题设定很新颖。** 输出不仅是 kinematics，还包含 interaction force，适合拓展到 biomechanics 和触觉理解。
- **每顶点压力与 3D mesh 对齐。** 便于把 contact / pressure 与人体表面、关节角度和运动时序共同建模。
- **设备简单、视场大。** 单个 fisheye 相机即可覆盖手部，硬件成本较低。
- **有真实用户研究。** 不只报告视觉指标，还验证 pressure-aware interaction 的实际可用性。

## 局限

- 数据由 **20 名参与者**采集，规模对于深度模型尚有限；不同肤色、手型、佩戴位置和复杂物体接触下的泛化仍需要更多验证。
- 官方项目页目前将代码和数据标记为 **Coming Soon / 发布准备中**，因此完整复现条件尚未完全开放。
- 当前压力监督依赖专门采集系统和压力传感器生成 GT，训练数据扩展成本仍高。
- **推断：**对于被物体大面积遮挡的手指、非平面软物体或快速动态冲击，仅依赖腕部 RGB appearance 可能难以稳定恢复真实压力幅值。

## 个人评价

这篇论文与人体全身 3D reconstruction 不属于同一主线，但对“智能触觉 + 视觉人体理解”非常有参考价值。最值得借鉴的不是具体 hand backbone，而是它把 **pose、surface contact、pressure** 放在同一个 3D body representation 上进行预测。

**推断：**这个思路可以扩展到运动分析，例如在滑雪中把 3D body pose 与 ski/ground contact、足底压力或受力状态联合起来；在临床 gait 中也可以把 pose latent 与 ground reaction / plantar pressure 联合学习。相比单纯增加 RGB 或 optical flow 模态，这种物理交互信号可能更容易形成具有解释性的 biomechanics representation。

## 与我的研究关联

有三个可迁移点：

1. **Fisheye / wide-FOV body observation**：腕部超广角成像与 360°/fisheye 相机存在共同的畸变、视场和局部人体尺度问题，可参考其视角设计。
2. **Geometry + physical interaction joint representation**：可以借鉴“mesh vertex + pressure”的形式，把全身 3D pose 与 contact / force / ground reaction 统一到身体表面或关节表示中。
3. **多任务评价**：除了 MPJPE，还同时评价 pressure/contact 和最终交互任务，为体育或临床研究设计“姿态准确 ≠ 应用有效”的多层评价提供示例。

建议重点阅读硬件与标定、数据采集/GT 构建、Hand-VQ-VAE 与 pressure branch，以及用户研究部分。

## 后续阅读

- 关注官方代码和数据集正式开放后的训练协议与标注格式。
- 对比手部 contact/force estimation、wearable egocentric hand pose 与 tactile sensing 工作。
- **推断：**尝试把类似的 joint-aligned / surface-aligned physical tokens 引入全身动作模型，与 plantar pressure、IMU 或 ground contact 标签联合训练。
