---
title: "IM360: Large-scale Indoor Mapping with 360 Cameras"
authors: "Dongki Jung, Jaehoon Choi, Yonghan Lee, Dinesh Manocha"
venue: "IEEE/CVF International Conference on Computer Vision (ICCV 2025), pp. 29040–29050"
year: 2025
reading_date: 2026-11-12
status: skimmed
tags:
  - 360-vision
  - spherical-sfm
  - camera-geometry
  - dense-matching
  - bundle-adjustment
---

# IM360: Large-scale Indoor Mapping with 360 Cameras

## 基本信息

- **作者：** Dongki Jung, Jaehoon Choi, Yonghan Lee, Dinesh Manocha
- **会议/期刊：** ICCV 2025，pp. 29040–29050（正式会议论文；2025 年 10 月）；arXiv:2502.12545 首次提交 2025-02-18
- **年份：** 2025
- **阅读日期：** 2026-11-12
- **阅读状态：** `skimmed`
- **标签：** `360-vision`, `spherical-sfm`, `camera-geometry`, `dense-matching`, `bundle-adjustment`
- **论文：** https://openaccess.thecvf.com/content/ICCV2025/papers/Jung_IM360_Large-scale_Indoor_Mapping_with_360_Cameras_ICCV_2025_paper.pdf
- **DOI：** https://doi.org/10.1109/ICCV51701.2025.02697
- **arXiv：** https://arxiv.org/abs/2502.12545
- **代码：** https://github.com/jdk9405/IM360
- **数据集：** https://niessner.github.io/Matterport/ （Matterport3D；Stanford2D3D 官方入口本次未核验，不附链接）
- **项目主页：** https://jdk9405.github.io/IM360/
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高（360° camera geometry 模块）

## 一句话总结

将 ERP 图像的 dense correspondence、球面本质矩阵、triangulation 和 bundle adjustment 统一到 spherical SfM，再接入隐式场景表面与神经纹理优化，在稀疏室内全景视角中显著提高相机注册覆盖率。

## 研究问题与动机

大视场 ERP 有利于扩大视角重叠，但平面相机 SfM 的透视近似、多透视 cubemap 的匹配繁复和室内低纹理、遮挡会引发配准失败。论文关注**静态室内、多张离散 360° 图像**的几何与纹理重建，而不是连续的人体移动视频。传统 SfM 在稀疏全景扫描上经常遗漏大量 camera poses；本文强调原生球面射线一致性。

## 核心方法

1. 对每个 ERP 像素建立单位 bearing vector `u∈S²`，在球面域运行 EDM dense matching，并通过匹配量化和合并构造跨图像 tracks。
2. 基于球面 epipolar 约束 `u1ᵀ E u2 = 0` 和归一化 8-point solver 估计相对旋转与平移方向。
3. Spherical incremental triangulation 后使用 unit-bearing reprojection 的球面 bundle adjustment，优化全局相机与 3D 点；此阶段只提供 SfM 固有的相对尺度。
4. 将 ERP 投影为 cubemap，结合预训练几何先验与 DebSDF 的 SDF surface reconstruction、Marching Cubes mesh 提取，再用 TexRecon 初始化并通过可微 rasterization 优化 diffuse/specular texture。

## 数据集与评价指标

- **Matterport3D：** 官方数据集共 90 个室内场景、10,800 张 panorama；论文实验从官方 val/test 中各选 3 个场景，共 6 个场景（不是用 90 个全部做完整评测）。
- **Stanford2D3D：** 官方数据集 6 个室内区域；论文挑选 3 个场景验证 SfM。
- **输入/输出：** 离散 ERP 图像和预设 overlap/matching pair 信息 → camera pose、稀疏点云、mesh、纹理。
- **比较：** OpenMVG、SPSG-COLMAP、DKM-COLMAP、SphereGlue-COLMAP；纹理对比 TexRecon、ZipNeRF、3DGS、SparseGS。
- **指标：** registered images 数、camera pose AUC@3°/5°/10°（角度阈值；勿视为 ATE 米）、渲染 PSNR/SSIM/LPIPS。

## 主要结果

- Matterport3D 场景 `2t7WUuJeko7`：相机注册 **37/37**，DKM-COLMAP 为 22/37；AUC@5° **69.05**，DKM-COLMAP 为 27.70（论文 Table 1）。
- Stanford2D3D area 5a：注册 **138/143**，DKM-COLMAP 为 78/143（Table 2）；不是全部室内区域都同样优于各 baseline 的每个指标。
- Matterport3D 六场景平均新视角渲染 PSNR **19.4 dB**，TexRecon **15.9 dB**，SparseGS **14.3 dB**；纹理优化中 diffuse-only 达 18.6，加入 specular 达 19.4 dB（Table 3）。
- 上述 camera 指标基于论文选择的离散室内视图；不包含长视频 camera ATE/RPE 或人体 MPJPE。

## 优点

- 区分原生 ERP 球面几何与仅把全景裁成六个透视面再做 SfM 的方法。
- 同时验证相机注册率和位姿角误差，并公开官方项目与代码。
- 球面 BA 的单位射线残差可以作为之后 human-to-camera reprojection 的可扩展优化骨架（此点为**推断**）。

## 局限

- 作者在 Conclusion 明确承认**匹配图像对依赖人工/预先给定的重叠信息**；Matterport3D 甚至使用房间分割标签，Stanford2D3D 使用 GT 几何与相机信息确定匹配对，不能直接认为端到端无监督。
- 静态室内、离散扫描而非多运动相机的同步长序列；相对 SfM 不自动给出世界公制尺度。
- 不估计人体，也没有 camera–human mutual refinement；对雪地纹理缺失、运动模糊和动态人体的泛化尚未验证。

## 个人评价

**推断：**这篇更适合作为双 360 项目的 spherical camera-only 几何参考，而不是完整 world-HMR baseline。最应复现的是 EDM track construction + spherical essential matrix + bearing-vector BA；纹理重建可暂不实现。应设计自动 pair retrieval 的消融，否则会高估在自由拍摄场景的可部署性。

## 与我的研究关联

**推断：**将 360 自身的 camera estimation 和 HMR 解耦，先对球面基础几何测试 tracking/registration 与角误差，再在 BA 中加入 2D-3D 人体 reprojection、可靠性权重、骨长、跨物理相机一致性等 factors。双 360 的物理平移基线与单 360 内同中心 virtual perspective view 是两类不同的约束，不能混用。报告 camera ATE/RPE/scale drift 与 W-MPJPE，重点验证 human→camera feedback 是否真正降低相机误差。

## 后续阅读

- **优先阅读：** Section 3.1（Spherical SfM），Section 4.1–4.3 与 Tables 1–3，Conclusion 的 matching-pair limitation。
- 与 [360DVO](../360-vision/2026-360dvo.md)、[PanoVGGT](../360-vision/2026-panovggt-feed-forward-panoramic-reconstruction.md)、[ViPE](../multiview-geometry/2025-vipe.md) 比较 ERP camera modeling。
- **推断：**复现实验：`ERP-native dense matching → +spherical BA → +multi-camera geometry → +human factors`，按 texture、overlap、synchronization error 分层评价。
