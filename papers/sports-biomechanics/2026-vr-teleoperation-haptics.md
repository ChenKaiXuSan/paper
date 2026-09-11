---
title: "VR-Based Teleoperation Framework: Integration of Haptic Feedback and Singularity Management"
authors: "Seungnam Yu, Geegum Lee, Jeongmok Kim"
venue: "IEEE Access, Vol. 14, pp. 41946–41963"
year: 2026
reading_date: 2026-09-12
status: skimmed
tags:
  - haptics
  - wearable
  - teleoperation
  - human-in-the-loop
  - vr
  - motor-feedback
---

# VR-Based Teleoperation Framework: Integration of Haptic Feedback and Singularity Management

## 基本信息

- **作者：** Seungnam Yu, Geegum Lee, Jeongmok Kim
- **会议/期刊：** IEEE Access, Vol. 14, pp. 41946–41963
- **年份：** 2026
- **发表日期：** 2026-03-12
- **阅读日期：** 2026-09-12
- **阅读状态：** `skimmed`
- **标签：** `haptics`, `wearable`, `teleoperation`, `human-in-the-loop`, `vr`, `motor-feedback`
- **价值类型：** Method Module / Related Work
- **阅读优先级：** A（高）
- **论文：** https://ieeexplore.ieee.org/document/11431594/
- **DOI：** https://doi.org/10.1109/ACCESS.2026.3673307
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

该工作把 robot workspace boundary 与 path-deviation error 编码为双通道 vibrotactile feedback，并结合 hand tracking、Adaptive Damped Least Squares 与 workspace impedance control；在 10 人 path-following 实验中，触觉将平均轨迹误差从 10.03 mm 降至 7.55 mm，说明“结构化运动误差 → 实时触觉 cue”可以带来可量化的动作精度提升。

## 研究问题与动机

VR teleoperation 中，操作者通过视觉看到机器人和虚拟工作空间，但 robot singularity、workspace boundary 和细小 trajectory deviation 并不一定能直接从视觉感知。仅依赖视觉容易在接近运动学极限时产生不稳定控制，也可能使精细路径任务出现持续偏差。

本文希望把稳定控制与人类感知反馈结合：控制器负责避免 singularity 和越界，haptic glove 则把控制相关状态直接编码给操作者，使用户在不持续观察额外 UI 的情况下感知“接近边界”和“偏离目标路径”两种不同信息。

## 核心方法

系统由 VR hand tracking、robot inverse kinematics / control、workspace safety layer 与 haptic feedback 组成。

1. **Native hand tracking**：在 VR 中直接获取操作者手部位置/姿态，并映射到 collaborative robot end-effector command，减少外部 mocap 或手柄依赖。
2. **Adaptive Damped Least Squares (A-DLS)**：根据 Jacobian singularity 状态动态调节 damping，使机器人接近奇异位形时保持更稳定 manipulability。
3. **Workspace impedance control**：在虚拟/实际 robot workspace 边界附近产生约束反力，避免命令超出安全区域。
4. **Dual-modality vibrotactile feedback**：fingertip vibration 用于提示 workspace limit，wrist vibration 用于提示 path deviation，使不同 control-relevant errors 使用不同身体部位编码。
5. **Real-time integration**：论文报告 robot-PC TCP/IP round-trip latency 小于 50 ms，VR local rendering latency 小于 20 ms，haptic feedback 估计 latency 约 20–30 ms。

## 数据集与评价指标

本文不是公开数据集 benchmark，而是系统验证实验。

- **Experiment 1：**评价 A-DLS 在 singularity / workspace 操作中的 manipulability 与稳定性；
- **Experiment 2：** **10 名参与者**完成 VR path-following task，比对 haptic-enabled 与 haptic-disabled 条件；
- 主要指标包括 manipulability above critical threshold 的时间比例、mean path-following error、统计显著性，以及 task completion behavior。

由于论文没有发布独立 benchmark dataset，本次不将实验记录误写为公开数据集。

## 主要结果

Experiment 1 中，A-DLS 使 manipulability 保持在 critical threshold 以上的时间比例达到 **92%**，而没有 adaptive damping 时为 **78%**，说明动态 damping 对接近 singular configuration 时的系统稳定性有效。

Experiment 2 中，haptic-enabled 条件的平均 path-following error 为 **7.55 mm**，无触觉条件为 **10.03 mm**，相对下降 **24.7%**，`p = 0.001`。论文同时指出，触觉 guidance 会使 task time 略有增加，作者将其解释为用户为了更高精度而投入更多修正操作。

因此本文的核心证据不是“触觉让体验更真实”，而是结构化 vibrotactile cue 能在 precision-critical motor task 中显著降低 objective trajectory error。

## 优点

- 触觉 cue 与具体控制误差一一对应：workspace limit 与 path deviation 被分离编码，解释性比通用振动增强更强。
- 把 haptic feedback 与 singularity / workspace control 放在同一系统中评价，形成 perception–control closed loop。
- 使用 objective path-following error，而不是只依赖主观问卷，对运动反馈研究更有参考价值。
- 报告了系统通信和 haptic latency 范围，为实时可穿戴反馈的工程设计提供量级参考。

## 局限

- Path-following user study 仅 **10 名参与者**，样本量较小，也没有长期训练保持或跨人群验证。
- 实验是受控 VR collaborative-robot teleoperation，并非户外体育、临床 gait 或自由身体运动，因此无法直接证明对这些场景同样有效。
- Haptic feedback 虽降低误差，但 task time 略有增加，说明 precision 与 response speed 可能存在 trade-off。
- 反馈变量主要是 robot workspace / path deviation，尚未处理人体 biomechanics、动作 phase、关节角或 CoM 等复杂运动状态。
- 本次未核验到官方代码、公开数据集或独立项目主页，完整系统复现需要自行重建硬件与控制链。

## 个人评价

这篇和此前 Visual-to-Haptic XR Glove 的区别很重要：前者主要证明动态视觉内容可以增强主观 realism，而本文直接证明触觉 cue 可以改善**客观轨迹控制误差**。对于智能触觉方向，后者更接近体育 coaching 或康复反馈真正需要的 evaluation paradigm。

**推断：**可把论文中的 `path deviation → wrist vibration` 替换为 `pose / joint-angle / CoM / contact / turn-phase deviation → spatial-temporal haptic cue`，并用动作纠正量而非主观 realism 作为主指标。这会让触觉成为 human-in-the-loop motor correction module，而不是附加的多模态输出。

## 与我的研究关联

对于滑雪和临床 gait，可构造：

1. no feedback；
2. generic event vibration；
3. pose-error-aware haptic；
4. biomechanics-aware haptic；
5. uncertainty-gated haptic，仅在视觉/3D reconstruction 置信度足够时提示。

**推断：**滑雪可将 trunk lean、edge/turn phase、左右 CoM / load asymmetry、目标关节角或轨迹偏差编码成不同 actuator / vibration pattern；临床 gait 可提示左右步态不对称、ROM deviation 或 phase timing error。建议同时报告 joint-angle / CoM error、correction latency、success rate、task time 与 retention，避免只优化即时精度而牺牲动作流畅性。

## 后续阅读

- 与 Visual-to-Haptic Augmentation in XR 比较“主观 perceptual augmentation”与“客观 motor correction”两类触觉评价范式。
- 阅读 sports coaching、rehabilitation vibrotactile biofeedback 和 wearable haptics 中的 spatial encoding / cue learning 文献。
- 设计 `generic vibration → error magnitude → error direction → biomechanics-aware cue` 的递进实验。
- 建立完整 sensing → reconstruction → inference → wireless → actuation latency budget，特别关注高速体育场景中反馈是否及时。
