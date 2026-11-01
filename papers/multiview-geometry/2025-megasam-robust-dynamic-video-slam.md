---
title: "MegaSaM: Accurate, Fast and Robust Structure and Motion from Casual Dynamic Videos"
authors: "Zhengqi Li, Richard Tucker, Forrester Cole, Qianqian Wang, Linyi Jin, Vickie Ye, Angjoo Kanazawa, Aleksander Holynski, Noah Snavely"
venue: "CVPR 2025"
year: 2025
reading_date: 2026-11-02
status: skimmed
tags:
  - camera-pose
  - visual-slam
  - dynamic-scene
  - unknown-intrinsics
  - uncertainty-aware-ba
  - moving-camera
---

# MegaSaM: Accurate, Fast and Robust Structure and Motion from Casual Dynamic Videos

## 基本信息

- **作者：** Zhengqi Li, Richard Tucker, Forrester Cole, Qianqian Wang, Linyi Jin, Vickie Ye, Angjoo Kanazawa, Aleksander Holynski, Noah Snavely
- **会议/期刊：** IEEE/CVF CVPR 2025（正式发表，pp. 10486–10496；作者项目页标注 Best Paper Honorable Mention）
- **首次提交：** 2024-12-05（arXiv:2412.04463）
- **年份：** 2025
- **阅读日期：** 2026-11-02
- **阅读状态：** `skimmed`
- **标签：** `camera-pose`、`visual-slam`、`dynamic-scene`、`unknown-intrinsics`、`uncertainty-aware-ba`、`moving-camera`
- **论文：** https://openaccess.thecvf.com/content/CVPR2025/papers/Li_MegaSaM_Accurate_Fast_and_Robust_Structure_and_Motion_from_Casual_CVPR_2025_paper.pdf
- **arXiv：** https://arxiv.org/abs/2412.04463
- **DOI：** 暂无（本次未从出版方核验）
- **代码：** https://github.com/mega-sam/mega-sam
- **数据集：** 暂无（论文评测使用 MPI Sintel、DyCheck 及 in-the-wild 动态视频，未核验到单独的 MegaSaM 数据集发布页）
- **项目主页：** https://mega-sam.github.io/
- **论文状态：** CVPR 2025 正式发表
- **主要价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高（2025 年重要 camera-only baseline 补读）

## 一句话总结

基于 DROID-SLAM 的可微 bundle adjustment，同时结合单目深度初始化、动态前景运动概率和可观测性驱动的 global BA，在低视差、动态内容与未知焦距的普通移动相机视频中估计 camera pose 和一致深度。

## 研究问题与动机

传统 SfM/SLAM 通常依赖稳定静态场景和足够平移视差；单目拍摄可能以旋转为主、包含大量动态人/物体、焦距未知，产生深度与相机运动歧义。MegaSaM 的问题不是 world-coordinate HMR，而是先提供更稳健的 camera-only 几何基线，避免上游轨迹错误污染后续人体世界坐标重建。

## 核心方法

1. **可微 camera BA**：沿用 DROID-SLAM 的 correspondence / disparity / pose 联合求解，扩展为可联合优化相机姿态、焦距与 disparity 的重投影目标，并使用 Schur complement 求解。
2. **动态运动概率**：多帧网络预测 object movement probability map，对动态前景在 BA 中降权；先在静态合成视频预训练，再用动态 Kubric 视频训练运动分支，而不是直接让全部动态干扰进入相机跟踪。
3. **Mono-depth 初始化**：结合 DepthAnything 的相对深度与 UniDepth 的 metric scale/shift/focal 先验，对 disparity 做对齐和初始化，再进行局部 BA。
4. **可观测性驱动 global BA**：利用 Hessian 的 disparity/focal 信息判断低视差、焦距不可观测情形，动态决定是否添加 monocular depth regularization、是否关闭焦距优化，而非总是强制一个深度 prior。
5. **一致视频深度**：在估计的 camera trajectory 条件下优化时序一致深度，无须每段视频都重新微调深度网络。

## 数据集与评价指标

- 训练：静态合成 TartanAir 的 163 个场景、Kubric 的 5K 静态视频；第二阶段使用 11K Kubric 动态视频、7 帧训练样本。
- Camera 测试：MPI Sintel 的 18 段序列（每段 20–50 帧）；DyCheck（真实手持动态视频，单段约 180–500 帧）；另外包括 12 段 in-the-wild 动态视频。
- 任务：单目 RGB 视频 → camera trajectory / focal length / disparity / temporally consistent depth；同时评价 calibrated 与 unknown-focal (uncalibrated)。
- 指标：camera ATE、RTE、RRE，depth Abs-Rel / log-RMSE / δ1.25，以及运行时间。**特别注意：**论文先把 GT camera trajectory 归一化为单位长度，再做全局 Sim(3) 对齐；表中的 ATE 不是米制 camera ATE，不能直接当 metric-scale accuracy。
- 基线：CasualSAM、LEAP-VO、ACE-Zero、Particle-SfM、RoDynRF、MonST3R；深度还比较 DepthAnything-v2、DepthCrafter。

## 主要结果

- MPI Sintel（Table 1，unknown focal）：MegaSaM ATE/RTE/RRE = **0.023 / 0.008 / 0.06**，MonST3R 为 **0.078 / 0.038 / 0.49**；calibrated 时 MegaSaM 为 **0.018 / 0.008 / 0.04**。
- DyCheck（Table 2，unknown focal）：MegaSaM ATE **0.020**，CasualSAM **0.209**、MonST3R **0.690**；对应 RTE **0.005**。
- 深度（Table 4）：Sintel Abs-Rel **0.21**（DepthCrafter 0.27），DyCheck Abs-Rel **0.11**（DepthAnything-v2 0.20）。
- 时间：Sintel 相机跟踪表格 unknown-focal 为约 **1.0 s/frame**；一致深度优化另报告 **1.3 FPS（336×144）**，不是整体实时 30 FPS 的保证。
- 消融（Table 5）：去掉 mono-depth 初始化、运动概率分支、两阶段训练或 uncertainty-aware BA 均恶化跟踪精度；表格消融设置与主评测设置略有差异，横向对比需保持相同 protocol。

## 优点

- 同时处理低视差、动态前景、未知焦距三类会破坏常规 SLAM 的条件，且主要模块可独立消融。
- Hessian observability → 自适应启停 depth prior 与 focal 优化，比固定权重深度正则更适合不确定 camera geometry。
- 有完整官方实现和项目示例，便于与 camera-only、human-aware camera refinement 对照。

## 局限

- 作者明确指出：动态物体占满画面、没有可跟踪纹理时仍可能失败；不支持视频内**变化的 focal length** 或**强径向畸变**。
- 主要采用透视相机几何，不能直接视为 360° ERP/fisheye 相机解法。
- ATE 使用轨迹归一化及 Sim(3) 对齐；**推断：**其优异 ATE 不代表已解决可独立验证的世界绝对尺度/scale drift。
- 依赖 DepthAnything/UniDepth 与合成训练先验；户外雪景、近距离人体、强动态遮挡及 360 多透视融合尚须专门验证。

## 个人评价

相比只把动态前景完全 mask 掉，本论文更重要的模块是根据几何可观测性决定何时信任 depth prior、何时不应优化 focal。对于相机、人、场景互相校正的研究，需要区分观测自身约束不足时的 prior-assisted improvement 与真正来自 human factor 的 camera correction。**推断：**可将其设为强 camera-only baseline，并对其输出轨迹施加人体因子检验额外的 ATE/RPE 收益。

## 与我的研究关联

- **Dual-360 / moving camera：**分别比较直接 ERP、perspective crops 或 rectified fisheye 的适用性；不能把同中心虚拟 views 当作独立 baseline。
- **World HMR：**`MegaSaM camera-only → HMR/world transformation → +human reprojection/bone/contact/cross-view uncertainty factor`；分别报告 camera ATE/RPE/scale drift 与 W-MPJPE/RTE。
- **滑雪：**按低纹理雪面、快速运动、rotation-dominant 与人占比四种条件分层；实验中使用真正 camera GT 或已校准 rig 验证，勿只凭视觉重投影误差认定轨迹准确。
- **最值得复现：**Sec. 3.2 object motion map、mono-depth initialization、uncertainty-aware global BA；Sec. 4.3 ablation。

## 后续阅读

- DROID-SLAM、MASt3R-SLAM、ViPE、AnyCam、360DVO；与 PACE / SynCHMR 的 human-assisted geometry 机制对照。
- **推断：**进行 `camera-only → +human metric cue → +full R/t joint refinement → +observability-adaptive weighting` 消融；另行报告无人体可见、人体误检及严重尺度漂移时的退化情况。
- 一手核验：CVPR 2025 官方论文 PDF、arXiv:2412.04463、作者项目 https://mega-sam.github.io/ 与官方代码仓库。
