---
title: "Dense Dynamic Scene Reconstruction and Camera Pose Estimation from Multi-View Videos"
authors: "Shuo Sun, Unal Artan, Malcolm Mielle, Achim J. Lilienthal, Martin Magnusson"
venue: "ECCV 2026; arXiv:2603.12064v4"
year: 2026
reading_date: 2026-10-31
status: skimmed
tags:
  - multiview-geometry
  - moving-camera
  - multi-camera-slam
  - camera-pose
  - dynamic-reconstruction
  - scale-consistency
  - camera-human-refinement
---

# Dense Dynamic Scene Reconstruction and Camera Pose Estimation from Multi-View Videos

## 基本信息

- **作者：** Shuo Sun, Unal Artan, Malcolm Mielle, Achim J. Lilienthal, Martin Magnusson
- **会议/期刊：** ECCV 2026（arXiv 的 Journal reference）；arXiv:2603.12064v4
- **首次提交 / 修订：** 2026-03-12 / 2026-10-05
- **年份：** 2026
- **阅读日期：** 2026-10-31
- **阅读状态：** `skimmed`
- **标签：** `multiview-geometry`, `moving-camera`, `camera-pose`, `dynamic-reconstruction`
- **DOI：** https://doi.org/10.48550/arXiv.2603.12064（arXiv DOI）
- **论文：** https://arxiv.org/html/2603.12064v4
- **代码：** https://github.com/ljjTYJR/Multiple-view-Dynamic-Reconstruction
- **数据集：** https://huggingface.co/datasets/shuooru/MultiCamRobolab（官方演示子集；完整 24 条研究序列的开放情况尚未核实）
- **项目主页：** 暂无
- **主要价值类型：** Baseline / Method Module / Dataset / Related Work
- **阅读优先级：** A+（紧密相关）

## 一句话总结

将多台**自由移动、已知内参且同步的 RGB 相机**连接为同一个时空图，通过 VGGT 宽基线初始化、跨相机 SLAM 与光流/深度交替优化，联合恢复多相机轨迹与动态场景稠密几何，是双移动 360° 系统的重要 camera-only 几何对照。

## 研究问题与动机

- 单目视觉 SLAM 只利用本相机的时间邻接，容易累计漂移；固定多相机 SLAM 则常要求相机间已知刚性外参。
- 多台自由移动相机可能只间歇重叠，跨相机尺度与坐标系难以统一；动态人物/机器人还破坏静态场景假设。
- 论文解决的是**多移动相机 + 动态稠密场景**的 camera/depth 联合估计，而不是直接估计人体 SMPL 或人体对相机的反向优化。

## 核心方法

1. **输入与状态：** 同步的多路 RGB 序列与每台相机已知内参 `K_i`；输出每台每帧的 `SE(3)` 位姿 `T_i^t` 和深度图 `D_i^t`。
2. **Wide-baseline initialization：** 取各相机初始帧输入 VGGT，生成跨相机一致的位姿/深度初值；通过全局 affine 对齐把 UniDepth 的每帧深度先验统一到初始化尺度。
3. **Spatio-temporal connection graph：** 三种边为同一相机的 temporal links、同一时刻跨相机的 spatial links、不同时间跨相机的 spatio-temporal links。空间边基于重投影覆盖率判断，论文给出大于 75% 投影落入图像范围的启发式。
4. **Joint tracking：** 使用 MegaSAM 风格的加权光流重投影残差及深度先验进行滑动窗口优化，管理跨相机边数量，减少低重叠条件下的尺度漂移。
5. **Dense refinement：** Phase 1 固定相机优化每帧深度的 scale / offset 与光流置信度；Phase 2 固定 scale / offset，交替优化逐像素深度与相机 `SE(3)` 增量，并加入相机轨迹平滑和先验项。
6. **关键边界：** 先验模型将动态区域匹配权重降低，尚未把人体骨架、骨长或 contact 作为显式相机几何因子。

## 数据集与评价指标

- **MultiCamRobolab：** 24 条真实 RGB-D 序列，2–3 台 Microsoft Azure Kinect，30 FPS，每条 150–300 帧；19 条双相机和 5 条三相机序列；涵盖 RoboDog overlap / non-overlap、RoboArm、DynamicHuman 与 three-camera。Qualisys 提供独立相机位姿 GT，RGB-D 深度仅用于评价。
- **MultiCamVideo：** Unreal Engine 合成多相机数据；从每段 81 帧、10 视角的视频中抽取 50 段、每段 3 视角用于测试；图片裁剪至 512×384，轨迹归一化。
- **Camera：** ATE、RTE、RRE；真实相机平移指标以米表示、旋转以度表示；合成数据轨迹归一化，不宜直接解释为真实米制误差。论文相机轨迹可视化采用 Sim(3) 对齐，必须结合具体评价协议阅读。
- **Depth/scene：** Abs.Rel、δ1.25、三维点一致性的中位距离 `M_d`。
- **Baselines：** COLMAP、GLOMAP、VGGT / sparse VGGT、Fast3R、FastVGGT、CUT3R，部分传统模型另有 dynamic mask / NetVLAD 设置。

## 主要结果

- 合成 MultiCamVideo：ATE **0.005**、RTE **0.001**、RRE **0.011**；FastVGGT 为 **0.023 / 0.008 / 0.080**（论文 Table 1；数据轨迹归一化）。
- 真实 MultiCamRobolab RoboDog-overlap：ATE **0.011 m**，FastVGGT **0.021 m**；RoboDog-non-overlap：两者都约 **0.020 m**，说明少重叠时并非处处领先（Table 2）。
- 三相机实验：ATE **0.020 m**、RTE **0.011 m**、RRE **0.326°**；FastVGGT 为 **0.032 m / 0.017 m / 0.363°**（Table 3）。
- 移除 wide-baseline initialization 或 spatio-temporal graph 导致多个真实场景的 ATE 上升（Table 5）。Depth refinement 两阶段的贡献见 Table 6。
- 论文结果支持**跨相机连接能改善几何追踪**，不能由此宣称人体姿态或物理相机的绝对尺度已经得到独立验证。

## 优点

- 真正研究**相机各自运动**而不是已知刚性 rig；时空图把间歇视野重叠转化为可检验的跨相机约束。
- 有 Qualisys 独立相机真值、双/三相机及有限视野重叠分层实验，并公开可运行的代码流程。
- 以 camera-only 方法对比传统 SfM 和几何基础模型，适合检测将来 human-to-camera 约束的真实增量。

## 局限

- 已知相机内参、已有时间同步、实验持续约 5–10 秒，尚不能直接覆盖未知畸变、原生 ERP / fisheye 与高速滑雪长序列。
- 跨相机投影重叠的启发式规则在动态大遮挡、低纹理雪地和局部独立运动中可能不可靠（**推断**）。
- 部分视野不重叠条件下与强 baseline 持平，收益具有场景依赖。
- 官方仓库仍将完整数据处理与上传列为 TODO；公开 Hugging Face 示例不能当作全部 24 条序列已公开。
- 论文没有 human→camera feedback；尺度来自视觉先验和对齐，不可等同于外部米制 GT。

## 个人评价

- **主要价值：** 为“两个相机都在运动时怎么合并各自相机轨迹”提供强于独立 VO 再拼接的几何 baseline。
- **值得验证的问题：** 断开跨相机连接、降低有效重叠率、加入异步偏移时，camera ATE/RPE、相机间 scale consistency 如何退化；同时报告真实尺度是否稳定。
- **推断：** 引入人体约束之前，必须先对照此类强 camera-only 时空图方法，否则姿态融合的收益可能只是更好的相机初始化。

## 与我的研究关联

- **双 360° 滑雪：** 可借鉴 `temporal + inter-camera spatial + cross-time` 连接图；但同一 360 相机切出的 perspective crops 共享 camera center，不能当作具有真实平移 baseline 的独立物理相机。
- **推断：** 先在透视裁切上形成可运行 camera-only baseline，再以真实双 360 球面投影与人体关键点/SMPL 重投影、骨长、contact、velocity、joint/axis reliability 作为额外约束修正每台物理相机轨迹。
- **必要评价：** `camera-only → +cross-camera graph → +human residual → +uncertainty-weighted human residual`；独立报告 camera ATE/RPE、scale drift、W-MPJPE/RTE、MPJVE/MPJAE，并按 view overlap 和人体可见率分层。
- **采集建议（推断）：** 未来多台 RX0 II 的 12 机位标定实验可以验证已知内参/外参与时序连接图的分离效果；其静止 rig 实验并不能替代两台移动相机的评价。

## 后续阅读

- **必读：** Sec. 3.2 的三类连接边及 wide-baseline initialization；Sec. 3.3 两阶段 depth/pose refinement；Sec. 4–6、Tables 1–6。
- 与仓库中的 MASt3R-SLAM、AnyCam、ViPE、360DVO、PanoVGGT、PACE、BodySLAM++ 和 Everybody Tracking Every Body 作统一比较。
- 首先用官方 demo 跑通一对同步已知内参的透视视频；额外记录缺失时间同步、跨相机重叠下降、FOV/内参扰动引起的误差。
- 官方原文：https://arxiv.org/html/2603.12064v4；官方实现：https://github.com/ljjTYJR/Multiple-view-Dynamic-Reconstruction
