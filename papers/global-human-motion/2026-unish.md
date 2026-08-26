---
title: "UniSH: Unifying Scene and Human Reconstruction in a Feed-Forward Pass"
authors: "Mengfei Li, Peng Li, Zheng Zhang, Jiahao Lu, Chengfeng Zhao, Wei Xue, Qifeng Liu, Sida Peng, Wenxiao Zhang, Wenhan Luo, Yuan Liu, Yike Guo"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-24
status: skimmed
tags:
  - 3d-human-pose
  - human-scene-reconstruction
  - camera-pose
  - world-coordinate
  - metric-scale
  - feed-forward
---

# UniSH: Unifying Scene and Human Reconstruction in a Feed-Forward Pass

## 基本信息

- **作者：** Mengfei Li, Peng Li, Zheng Zhang, Jiahao Lu, Chengfeng Zhao, Wei Xue, Qifeng Liu, Sida Peng, Wenxiao Zhang, Wenhan Luo, Yuan Liu, Yike Guo
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-24
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `human-scene-reconstruction`, `camera-pose`, `world-coordinate`, `metric-scale`, `feed-forward`
- **论文：** [CVF Open Access](https://openaccess.thecvf.com/content/CVPR2026/html/Li_UniSH_Unifying_Scene_and_Human_Reconstruction_in_a_Feed-Forward_Pass_CVPR_2026_paper.html) / [arXiv:2601.01222](https://arxiv.org/abs/2601.01222)
- **代码：** [murphylmf/UniSH](https://github.com/murphylmf/UniSH)
- **数据集：** 暂无专属 UniSH 数据集下载页；训练使用 BEDLAM 与作者整理的未标注网络视频，评测使用 Bonn、EMDB-2、RICH
- **项目主页：** [UniSH Project Page](https://murphylmf.github.io/UniSH/)
- **DOI：** [10.48550/arXiv.2601.01222](https://doi.org/10.48550/arXiv.2601.01222)

## 一句话总结

UniSH 将场景重建、相机参数估计和 Human Mesh Recovery 统一到一个 feed-forward 框架中，并通过 metric-scale AlignNet 与无标注真实视频的粗到细几何监督，使单目视频中的 SMPL 人体、相机和场景能够在同一世界坐标系中对齐。

## 研究问题与动机

现有 3D scene reconstruction 与 HMR 通常分开处理：前者擅长恢复场景几何与相机，但人体表面粗糙且容易受动态前景影响；后者能估计人体姿态与形状，却往往停留在 camera-relative 坐标，难以得到与场景一致的 metric-scale global placement。优化式 human-scene 方法虽然可联合求解，但推理代价高；已有 feed-forward 联合方法又存在两帧推理带来的累积漂移以及真实联合标注不足的问题。

UniSH 的目标是：从一段 monocular video 一次前向预测 scene point maps、camera extrinsics/intrinsics、SMPL pose/shape 和人体在场景中的绝对位置，并尽量利用无标注真实视频缓解 synthetic-to-real domain gap。

## 核心方法

### 1. Scene Reconstruction Branch

场景分支继承 $\pi^3$ 的 permutation-equivariant 3D reconstruction prior。输入整段图像序列后，跨帧 Transformer 同时预测每帧 camera extrinsics、point map 和 confidence map，并由 point map 推导 camera intrinsics。

### 2. Human Body Branch

人体分支采用 CameraHMR。每帧的人体 crop、bounding box 和由场景分支预测的 focal length 一起输入，得到 per-frame SMPL pose $\theta_i$ 与 body shape $\beta$。最终 shape 在序列维度平均，以增强时间一致性。

### 3. AlignNet

论文新增轻量的两层 Transformer decoder AlignNet，将 scene geometric features 作为 key/value，将 scale token 与 human features 作为 query，预测全局 metric scale $s$ 和每帧 SMPL translation $t_i$，负责把场景与人体放进同一坐标系。

### 4. 三阶段训练

- **Stage 1：Human Surface Refinement。** 用 MoGe-2 为未标注真实视频提供 pseudo-depth，通过 confidence-aware local human loss 蒸馏人体局部高频几何，并用原模型 point map regularization 防止遗忘。
- **Stage 2：Coarse-grained Alignment。** 在 BEDLAM synthetic data 上利用 SMPL vertex、3D/2D keypoint、pose、shape、translation 与全局 scale supervision 学习初始 metric placement。
- **Stage 3：Fine-grained Alignment。** 在无标注真实视频上，只优化 AlignNet；使用 visible SMPL 与 reconstructed human point cloud 的单向 Chamfer alignment、depth-ordering regularization，以及 2D keypoint reprojection loss 做 sim-to-real 对齐。

## 数据集与评价指标

作者另外整理了面向人体的网络舞蹈视频集合，经镜头连续性、单人检测、人体尺寸与无遮挡等自动筛选后得到 **1,354 条序列、约 1.2 million frames**。训练时每次采样 5 秒片段，并以 6 FPS 取 30 帧。

实验包括：

- **Bonn**：human-centric video depth estimation，指标为 Abs Rel 与 $\delta<1.25$。
- **EMDB-2、RICH**：global human motion estimation。每条序列切成 100-frame segment，报告 WA-MPJPE、W-MPJPE 和 Root Translation Error (RTE)。
- 输入为 monocular RGB video；输出包括 metric point maps、camera pose/intrinsics、SMPL pose/shape 与 world-space translation。

## 主要结果

在 Bonn 上，UniSH 的 **Abs Rel = 0.035，$\delta<1.25=0.980$**；相比 $\pi^3$ 的 0.049/0.975、VGGT 的 0.057/0.966，人体中心场景深度更准确。

在 EMDB-2 上，UniSH 的 **WA-MPJPE/W-MPJPE/RTE = 118.5 mm / 270.1 mm / 5.8%**，显著优于另一个 feed-forward 联合重建 baseline JOSH3R 的 220.0/661.7/13.1，但仍弱于优化式 JOSH 的 68.9/174.7/1.3 和 specialized HMR 方法的部分指标。

在 RICH 上，UniSH 为 **118.1 mm / 183.2 mm / 4.8%**。这些结果说明其主要优势不是在所有 HMR 数值上绝对最优，而是在“不做 test-time optimization 的同时联合输出 scene + camera + metric human”这一组合能力。

## 优点

- 把 scene reconstruction、camera recovery 与 HMR 放入同一个 feed-forward 系统，而不是简单串联多个独立结果。
- AlignNet 明确预测 scene scale 和 SMPL global translation，使 metric alignment 成为网络学习目标。
- 用无标注真实视频和 expert depth pseudo-label 缓解 synthetic-only training 的 domain gap。
- 实验同时评价 scene depth 与 global human motion，能够看到方法在几何与人体两个方向的真实 trade-off。
- 官方代码已经公开，具有较好的复现实用性。

## 局限

- 作者明确指出自建网络数据主要来自舞蹈视频，可能存在 demographic/body-type bias。
- 数据筛选会去除边界截断、多人以及明显环境遮挡，因此真实复杂遮挡下的泛化仍需进一步验证。
- 系统依赖 $\pi^3$、CameraHMR、MoGe-2、SAM2 等多个强预训练组件，性能下降时较难完全分离误差来源。
- 从结果看，UniSH 在 global human motion 的绝对精度上仍未超过 JOSH、GVHMR 等专门方法。
- **推断：**该方法更多是通过 learned alignment 将 camera/scene 与 human 对齐，而不是显式做 camera-human mutual geometric refinement；因此在人类 3D 几何可反向校正 camera trajectory 的场景中仍有研究空间。

## 个人评价

这篇论文最值得关注的不是单独的 HMR backbone，而是把“相机、场景尺度、人体绝对位置”视为一个统一对齐问题。AlignNet 很轻量，却承担了从 camera-relative HMR 到 metric world-space reconstruction 的关键桥接作用。对近期 moving-camera 研究而言，它是一个非常好的 feed-forward baseline，也能帮助区分“camera prior 更准”和“human-camera coupling 更有效”这两类提升。

我认为需要特别注意其定量结果：UniSH 的联合能力很强，但 global motion 指标并没有全面胜过 specialized HMR。这意味着后续若提出 camera-human joint optimization，实验不能只比较是否“联合”，而应明确证明 camera trajectory、root trajectory 与 world-space joints 是否同时改善。

## 与我的研究关联

与移动相机下 3D 人体重建高度相关。可直接借鉴以下实验设计：

1. 将 ViPE / AnyCam 等外部 camera trajectory 与 UniSH-style scene branch 做比较，设置 `GT camera / predicted camera / learned alignment / jointly refined camera`。
2. 把多 perspective 3D keypoints 或 skeleton features 作为 human branch，保留 AlignNet 的 metric scale / translation 预测思路。
3. 在 3DPW、EgoHumans 或真实 360 自拍视频上做 cross-dataset zero-shot，重点报告 camera ATE/RPE、W-MPJPE、RTE 与 root trajectory error。
4. **推断：**相比 UniSH 的单向 scene→human alignment，可以进一步让人体多视角/多透视几何 residual 反向更新 camera trajectory，形成真正的 camera-human mutual refinement。

## 后续阅读

- JOSH: Joint Optimization for 4D Human-Scene Reconstruction in the Wild。
- OnlineHMR: Video-based Online World-Grounded Human Mesh Recovery。
- AnyCam: Learning to Recover Camera Poses and Intrinsics from Casual Videos。
- Human3R / JOSH3R 等 feed-forward human-scene reconstruction 方法。
