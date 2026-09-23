---
title: "Fish2Mesh Transformer: 3D Human Mesh Recovery from Egocentric Vision"
authors: "Tianma Shen, Aditya Puranik, James Vong, Vrushabh Deogirikar, Ryan Fell, Julianna Dietrich, Maria Kyrarini, Christopher Kitts, David C. Jeong"
venue: "ICCV 2025"
year: 2025
reading_date: 2026-09-24
status: skimmed
tags:
  - egocentric-hmr
  - fisheye
  - 360-vision
  - smpl
  - transformer
---

# Fish2Mesh Transformer: 3D Human Mesh Recovery from Egocentric Vision

## 基本信息

- **作者：** Tianma Shen, Aditya Puranik, James Vong, Vrushabh Deogirikar, Ryan Fell, Julianna Dietrich, Maria Kyrarini, Christopher Kitts, David C. Jeong
- **会议/期刊：** Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV 2025), pp. 6498–6507
- **年份：** 2025
- **首次提交：** 2025-03-08（arXiv v1）
- **正式发表：** 2025-10（ICCV 2025 proceedings）
- **阅读日期：** 2026-09-24
- **阅读状态：** `skimmed`
- **标签：** `egocentric-hmr`, `fisheye`, `360-vision`, `smpl`, `transformer`
- **论文状态：** 正式同行评审会议论文
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** A+
- **DOI：** https://doi.org/10.48550/arXiv.2503.06089
- **论文：** https://openaccess.thecvf.com/content/ICCV2025/html/Shen_Fish2Mesh_Transformer_3D_Human_Mesh_Recovery_from_Egocentric_Vision_ICCV_2025_paper.html
- **代码：** https://github.com/Santa-Clara-Imaginarium-Lab/Fish2Mesh-Human-Mesh-Recovery
- **数据集：** 暂无
- **项目主页：** https://fish2mesh.github.io/

> 注：arXiv v1 的作者列表与 ICCV 正式版不同；本笔记按 CVF 正式 proceedings 页面记录作者。

## 一句话总结

Fish2Mesh 将 180° fisheye 的球面几何直接编码为 Egocentric Position Embedding，并与轻量 Swin Transformer、SMPL/camera/2D-3D multi-task heads 联合训练，为“直接在原始 fisheye 域做人体系数恢复”提供了比先校正/切 perspective crop 更直接的 HMR baseline。

## 研究问题与动机

头戴式 fisheye 相机能够覆盖更大人体区域，但同时引入强非线性畸变、自遮挡和身体部位离开视野等问题。既有 egocentric research 更集中在 3D pose，HMR 方法则常把 fisheye 输入当作普通图像，或者依赖额外 projection/preprocessing，因此难以稳定恢复 SMPL body shape、pose 与相机相对位置。

Fish2Mesh 的核心观点是不把 fisheye distortion 仅看成需要预先“消除”的噪声，而是把它对应的 spherical geometry 作为 Transformer positional prior。论文还通过 prompt-based ECHP extension 与 4DHuman weak supervision 增加自然头部运动、遮挡和 out-of-frame 情况，缓解 egocentric HMR 数据不足。

## 核心方法

### 1. Egocentric Position Embedding（EPE）

对于 180° fisheye 输入，作者依据 equirectangular / spherical mapping 将 2D pixel 映射到离散的 3D spherical coordinates，再使用 `POS[x3D, y3D, z3D]` learnable table 构造 position embedding。该 embedding 注入 Swin Transformer，目标是在 raw distorted input 上保留方向与球面空间关系，而不是依赖普通 grid positional bias。

### 2. Swin Transformer backbone

输入图像被划分为 patches，经过 patch merging 与 window / shifted-window attention。由于 EPE 已提供位置编码，作者去除标准 Swin relative position bias，使模型主要依赖 fisheye-aware spherical positional information 与视觉特征。

### 3. Multi-task heads

网络同时预测 SMPL shape、SMPL pose、global orientation、camera transformation、3D joints 和 2D joints。总损失包含 SMPL、orientation、3D joint 与 2D reprojection losses，通过 2D/3D auxiliary supervision 约束 egocentric geometry。

### 4. Prompt-based data expansion

作者在 ECHP 设置上增加更自然的 prompt-driven movements，并使用预训练 4DHuman 生成 weak supervision，重点增加自然 head motion、self-occlusion 和 body-part-out-of-frame 等原 ECHP 较少覆盖的情况。

## 数据集与评价指标

- **Ego4D：**论文引用的完整数据规模为 3,670 h、923 名参与者、9 个国家、74 个地点。论文没有汇总 Fish2Mesh 实际用于 HMR 训练/测试的唯一 frame / subject 数，因此不进一步猜测。
- **ECHP：**真实 fisheye egocentric 3D human pose/HMR evaluation setting，包含多相机 ground truth；论文正文没有汇总唯一受试者/帧数。
- **Extended ECHP / Our：**prompt-based natural-motion extension，使用 4DHuman weak supervision；具体扩展规模位于 supplementary，正文没有给出总样本量。
- **输入：**单帧 head-mounted fisheye RGB。
- **输出：**SMPL shape/pose、global orientation、camera transformation、3D/2D joints。
- **主要指标：**MPJPE、MPVPE、PA-MPJPE、PA-MPVPE，单位 mm。作者把 PA-MPJPE / PA-MPVPE 作为主要跨数据集指标；PA 会对 scale、rotation 和 translation 做 Procrustes alignment。

## 主要结果

在 **ECHP** 上，Fish2Mesh 的 MPJPE / MPVPE / PA-MPJPE / PA-MPVPE 为 **79.699 / 98.111 / 57.671 / 75.322 mm**，EgoHMR 为 **84.332 / 99.983 / 64.112 / 79.031 mm**。

在 **Ego4D** 上，Fish2Mesh 达 **71.934 / 84.116 / 41.931 / 54.756 mm**，而 EgoHMR 为 **224.423 / 311.129 / 114.423 / 128.999 mm**。在作者扩展数据上，Fish2Mesh 为 **57.352 / 71.233 / 37.242 / 51.580 mm**。

EPE 的贡献很明显：ECHP 上去掉 EPE 后 PA-MPJPE 从 **57.671** 恶化到 **92.448 mm**；Ego4D 从 **41.931** 恶化到 **90.111 mm**。去掉作者扩展数据后，ECHP PA-MPJPE 为 **71.446 mm**。

官方项目页还报告 Fish2Mesh 约 **4.74 GFLOPs、7.5M parameters、48.19 MB、4.47 ms inference**，明显轻于其对比的 4DHumans、EgoHMR 和 FisheyeViT。

## 优点

- 把 fisheye spherical geometry 直接写进 representation，而不是只做 image rectification，和 360° / omnidirectional HMR 的核心几何问题高度一致。
- EPE ablation 跨三个测试集均有大幅改善，说明 geometry-aware positional encoding 并非装饰性模块。
- 模型较轻，适合作为多 perspective / multi-camera 系统里的 per-view HMR front-end。
- 同时预测 SMPL、camera transformation 和 2D/3D joints，为后续加入 spherical reprojection、cross-view consistency 或 camera refinement 留出接口。

## 局限

- 核心任务仍是 **camera-relative / egocentric HMR**，并没有估计物理相机的连续 world trajectory，也没有 camera-human mutual refinement。
- 论文主要强调 PA-MPJPE / PA-MPVPE，而 Procrustes alignment 会消去 scale、rotation、translation；因此这些结果不能证明 world-coordinate translation、metric scale 或 camera trajectory 准确。
- 方法是单帧 HMR，没有显式 temporal modeling；高速运动中的 blur、长时间遮挡与跨帧 consistency 仍需另行验证。
- 训练与测试集中使用特定 head-mounted fisheye setup；不同 fisheye lens model、dual-fisheye stitching、full ERP 与极端边缘畸变的 zero-shot generalization 没有被系统拆开评价。
- **推断：**EPE 依赖由 lens/spherical mapping 构造的位置表，因此迁移到 360° ERP、不同 optical center 或 stitched dual-fisheye 时，应重新定义 projection-aware embedding，而不能直接复用同一位置表。

## 个人评价

这篇更适合作为 **fisheye-aware human front-end baseline**，而不是 world-HMR 终点。它补齐了一个很实用的对照：当前 360° pipeline 不应只比较“ERP → perspective crops → 通用 HMR”，还应加入“raw fisheye / spherical-domain HMR”。如果 direct fisheye baseline 已经明显减少边缘身体误差，后续 multi-view fusion 与 camera refinement 才能更干净地分析自身贡献。

## 与我的研究关联

与当前 360° selfie / dual-360 skiing 直接相关。**推断：**可以设计以下递进实验：

1. `ERP / dual-fisheye raw input → generic HMR`；
2. `ERP → perspective crops → generic HMR`；
3. `raw fisheye → Fish2Mesh-style spherical EPE → SMPL`；
4. `+ multi-perspective / dual-360 confidence-aware fusion`；
5. `+ camera trajectory / scale / spherical reprojection refinement`；
6. `+ human residual → physical camera R/t/scale correction`。

建议特别复现 Sec. 3.2 EPE、Table 1 和 Table 2，并按 radial distance / view direction / joint visibility 分层报告误差。对于双 360，还应同时报告 camera ATE/RPE、scale drift、W-MPJPE/RTE，避免 PA-MPJPE 把真正的 global geometry error 对齐掉。

## 后续阅读

- EgoRear：比较真实 front/rear fisheye 的 visibility gain 与 cross-view fusion。
- ViPE、360DVO、PanoAir：补齐 physical panorama camera trajectory 与 depth/geometry。
- Kineo、CHROMM、Tele360：比较多中心 sparse-camera geometry 与 unposed human reconstruction。
- 后续实验重点：direct fisheye HMR vs perspective-crop HMR、projection-aware positional encoding、以及 fisheye/spherical human residual 是否能稳定反向修正物理 camera trajectory。
