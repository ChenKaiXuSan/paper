---
title: "VideoRun2D Demo: Markerless Body Tracking for Biomechanical Analysis of Running"
authors: "Luis F. Gomez, Julian Fierrez, Roberto Daza, Ruben Tolosana, Aythami Morales, Gonzalo Garrido, Javier Rueda, Enrique Navarro"
venue: "arXiv:2608.19480"
year: 2026
reading_date: 2026-09-16
status: skimmed
tags:
  - sports-biomechanics
  - running
  - markerless-motion-capture
  - 2d-pose
  - joint-angle
---

# VideoRun2D Demo: Markerless Body Tracking for Biomechanical Analysis of Running

## 基本信息

- **作者：** Luis F. Gomez, Julian Fierrez, Roberto Daza, Ruben Tolosana, Aythami Morales, Gonzalo Garrido, Javier Rueda, Enrique Navarro
- **会议/期刊：** arXiv:2608.19480
- **年份：** 2026
- **阅读日期：** 2026-09-16
- **阅读状态：** `skimmed`
- **标签：** `sports-biomechanics`, `running`, `markerless-motion-capture`, `2d-pose`, `joint-angle`
- **论文：** https://arxiv.org/abs/2608.19480
- **代码：** 暂无（未核验到与 2026 Demo 论文对应的代码更新）
- **数据集：** 暂无（官方 VideoRun2D 仓库的数据下载部分仍标注为 under construction）
- **项目主页：** https://github.com/BiDAlab/VideoRun2D （VideoRun2D 系统官方仓库；当前 README 主要对应 2024 版本）

## 一句话总结

VideoRun2D Demo 用多种 markerless human pose trackers 从冲刺视频估计 hip/knee sagittal joint angles，并以专家手工标注验证；在 44 名职业跑者、314 次冲刺上，较好的 tracker 的平均 RMSE 约为 5.83°–11.46°，加入 outlier post-processing 后可降至约 5.30°–9.87°。

## 研究问题与动机

通用 pose estimator 的 2D keypoint 精度越来越高，但体育生物力学真正关心的往往是 joint-angle trajectory，而不是 image-space keypoint AP。VideoRun2D Demo 因此直接检验不同人体 pose trackers 是否能支持 sprint biomechanics，并把 hip flexion/extension 与 knee flexion/extension 作为更接近运动分析需求的输出。

方法还加入 outlier detection / post-processing，用来处理 markerless tracking 中偶发的关节点跳变。其意义在于把“视觉模型看起来追踪得不错”转化为可与专家标注直接比较的角度误差，而不是只用 pose benchmark 的通用指标评价体育应用。

## 核心方法

框架输入 sprint videos，使用多种 human pose estimators 得到人体关键点，再转换为随时间变化的关节角信号，重点分析：

- hip flexion / extension；
- knee flexion / extension。

自动结果与 biomechanical experts 的 manual annotations 对比。系统还加入 outlier detection 后处理，以识别和修正 pose tracking 中的异常点。

2026 arXiv 摘要没有列出所有对比 tracker 的完整名称与逐模型表格，因此本笔记不根据二手页面补写具体模型排名。官方 VideoRun2D 仓库目前主要记录早期 2024 系统版本，不能直接把其中旧实验配置当作 2026 Demo 的完整实验设置。

## 数据集与评价指标

2026 Demo 使用：

- **44 名 professional runners**；
- **314 次 sprints**；
- expert manual annotations 作为参考；
- 关注 hip 与 knee 的 flexion/extension angles。

主要评价指标为关节角度 **RMSE（degrees）**。该设定直接衡量 markerless pose 对 biomechanical joint-angle estimation 的影响，而不是仅报告 2D keypoint detection accuracy。

## 主要结果

arXiv 摘要报告：

- 较好的 pose trackers 的平均 RMSE 范围约为 **11.46° 到 5.83°**；
- 加入 post-processing 后，对应误差可降低至约 **9.87° 和 5.30°**。

作者据此认为，现代 human pose tracking 可以成为 running biomechanics 的低成本分析工具，同时 post-processing 对处理自动 tracker 的异常点具有实际价值。

## 优点

- 使用 44 名职业跑者、314 次冲刺，规模明显大于许多小样本跑步验证实验。
- 评价对象是 biomechanical joint angles，而非只停留在 keypoint AP/PCK。
- 与专家 manual annotations 直接比较，结果容易被体育科学人员理解。
- outlier detection 显示工程后处理仍能为实际动作分析带来可量化改进。

## 局限

- 目前只关注 hip/knee 两类 sagittal-plane angles，不能代表完整 3D biomechanics。
- 参考标准是 expert manual annotations，而不是独立 optical mocap / force plate，因此主要验证的是与专家视觉标注的一致性，而非绝对 3D kinetic ground truth。
- 论文摘要没有公开完整 tracker-by-tracker 实验表；本次也未核验到 2026 Demo 对应的正式代码更新或可下载数据。
- **推断：**冲刺场景的高速、较规律矢状面动作与临床 gait、滑雪等多平面运动存在明显 domain gap，角度误差不能直接外推。

## 个人评价

这篇工作的价值在于把“pose estimator 是否更好”转换成“下游 biomechanics 是否更准”。对于实际体育/医疗视频，MPJPE、PCK 或 2D detection score 与真正关心的 hip/knee angle、ROM、phase timing 并不一定单调对应，因此这种 task-level validation 很重要。

**推断：**后续若比较多个 HMR/HPE backbone，应避免只按 pose benchmark 排名；更合理的是把每个模型转换到同一 biomechanical variable space，再比较 joint-angle RMSE、时序 phase error、outlier rate 和 subject-level repeatability。

## 与我的研究关联

对滑雪项目，可以把 VideoRun2D 的验证思路扩展到 trunk lean、pelvis rotation、hip/knee flexion、edge/turn phase 与左右 asymmetry；对临床 gait / ASD，则可以比较 trunk/pelvis/hip/knee 的 ROM 与 phase-specific deviations。

**推断：**一个有价值的实验链是 `2D keypoints → 3D pose/HMR → biomechanics variables → clinical/sports decision`，同时检查哪一层误差最能预测最终任务性能。还可以复现其 outlier post-processing 思路，并进一步加入 temporal confidence、multi-view consistency 或 uncertainty gating，验证是否比简单平滑更稳健。

## 后续阅读

- 对比 OpenCap Monocular、Biomechanical 3D Body、SynthGait-19K 等直接输出 kinematics / gait variables 的方法。
- 增加 optical mocap、force plate、IMU 或 pressure ground truth，区分 kinematic agreement 与真正的 kinetic validity。
- 在高速体育中测试 2D tracker failure、遮挡和运动模糊对 joint-angle signal 的影响。
