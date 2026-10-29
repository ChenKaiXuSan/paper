---
title: "PanoVGGT: Feed-Forward 3D Reconstruction from Panoramic Imagery"
authors: "Yijing Guo, Mengjun Chao, Luo Wang, Tianyang Zhao, Haizhao Dai, Yingliang Zhang, Jingyi Yu, Yujiao Shi"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-10-30
status: skimmed
tags:
  - 360-vision
  - panorama
  - camera-pose
  - multiview-geometry
  - depth-estimation
  - point-cloud
  - feed-forward
---

# PanoVGGT: Feed-Forward 3D Reconstruction from Panoramic Imagery

## 基本信息

- **作者：** Yijing Guo, Mengjun Chao, Luo Wang, Tianyang Zhao, Haizhao Dai, Yingliang Zhang, Jingyi Yu, Yujiao Shi
- **会议/期刊：** IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR 2026), pp. 36444–36453
- **年份：** 2026；arXiv v1 首次提交 2026-03-18；CVPR 正式会议录 2026-06
- **阅读日期：** 2026-10-30
- **阅读状态：** skimmed
- **标签：** 360-vision、panorama、camera-pose、multiview-geometry、depth-estimation、point-cloud
- **论文：** https://openaccess.thecvf.com/content/CVPR2026/html/Guo_PanoVGGT_Feed-Forward_3D_Reconstruction_from_Panoramic_Imagery_CVPR_2026_paper.html
- **arXiv：** https://arxiv.org/html/2603.17571v1
- **DOI：** https://doi.org/10.48550/arXiv.2603.17571 （arXiv DOI；不是 CVPR 正式 DOI）
- **代码：** https://github.com/YijingGuo-June/PanoVGGT
- **数据集：** https://huggingface.co/datasets/YijingGuo/PanoCity
- **项目主页：** 暂无
- **论文状态：** 正式同行评审会议论文；代码、权重、PanoCity 已公开
- **主要价值类型：** Baseline / Method Module / Dataset / Related Work
- **阅读优先级：** A+

## 一句话总结

直接输入一张或多张 ERP 全景，利用球面位置编码、SO(3) 增强和随机世界锚点，在单次前向传播中联合恢复 camera 6DoF、depth 与 pointmaps，可作为 360° 移动相机研究的重要 camera-only geometry baseline。

## 研究问题与动机

VGGT、π³ 等多视角前馈三维重建模型主要围绕 pinhole 透视图像训练。直接输入 ERP 会遇到纬度相关畸变、左右边界断裂和特征统计偏移；切分透视图再拼接则可能带来跨视图不一致。传统 panoramic depth 或 VO 又多是单一任务，缺少直接统一预测姿态、深度和全局三维点图的框架。

论文主张在球面投影域原生学习跨视角几何，而不是将 360° 图像视为拼接的多个无关 pinhole 图像。

## 核心方法

1. **输入和骨干：** 无序 ERP 图像集合，经 DINOv2 patch encoder 提取图像 token；交替使用单视图与跨视图 self-attention 的 Geometry Aggregator，实现 permutation-equivariant 的多视角几何推断。
2. **Spherical-aware Position Embedding：** 从全景 patch 的两个球面角构造 sin/cos 编码，经 MLP 得到位置向量，以处理 ERP 周期边界与纬度畸变；与分支适配器一起支持相机、local/global pointmap 预测。
3. **Spherical SO(3) augmentation：** 同时旋转全景 RGB、depth、camera pose，而不只沿水平轴平移，提高 roll/pitch/yaw 的几何鲁棒性。
4. **Stochastic anchoring：** 每个训练样本随机选参考全景，所有姿态和点图相对该锚点重表达，避免把“第一张输入”为固定世界原点的顺序偏置写入模型。
5. **多任务输出与损失：** 预测每视角的 SE(3) camera-to-world、range/depth、local 和 global pointmaps；使用共享最优尺度对齐的点图损失、表面 normal consistency、relative rotation / translation supervision。

**注意：**训练时对几何和相机平移采用统一 scale alignment，不可把输出解释为无外部锚定时严格可辨识的绝对米制尺度。

## 数据集与评价指标

- **PanoCity：** 作者公开的 UE5 + AirSim 合成城市数据，官方数据卡明确为 119,537 张 4096×2048 RGB 全景、同数目的深度图与相机 6DoF 位姿；三个城市环境，五种天气/照明条件；按 trajectory 划分训练/验证/测试，约 18:1:1。
- **额外评测：** Matterport3D、Stanford2D3D、Structured3D、Pano3D/GibsonV2。相机 pose 实验主要对 Matterport3D、Stanford2D3D、PanoCity 采用 3-view 方案；PanoCity 点云重建表采用 10 panoramas。未公布可简单合并的统一测试视频/受试者总数。
- **输入输出：** 1 或多张 ERP RGB → camera SE(3)、per-pixel depth、共享世界点图；论文训练阶段典型输入 336×672，表中评测全景为 512×1024。
- **评价：** 相对姿态 AUC@30、旋转角误差、平移方向角误差（单位：度）；depth Abs Rel / δ；pointmap Accuracy、Completeness、Overall。相机平移方向角不是 metric camera translation ATE/RPE。
- **对照：** VGGT、π³、在全景数据重训的 π³*、Bifusev2；depth 与 UniFuse、BiFuse++、EGFormer 等比较。

## 主要结果

- PanoCity 3-view camera AUC@30 = **0.949**；全景重训 π³* 为 **0.682**，直接输入的 VGGT 为 **0.205**。PanoVGGT 平均旋转角误差 **0.873°**、平移方向角误差 **2.168°**。
- Matterport3D / Stanford2D3D 的 AUC@30 分别为 **0.459 / 0.556**；说明合成训练优势不能不加区分地推广到所有室内域。
- PanoCity 10-view global pointmap 的 Overall Mean **0.737**，全景重训 π³* 为 **1.074**（沿用原论文表格单位，不自行换算）。
- Pano3D/GibsonV2 zero-shot depth、仅 scale 对齐时，Abs Rel **0.0869**，优于经 dodecahedral 重投影的 VGGT **0.1266**。
- 消融：在 Matterport3D 上，π³* baseline AUC@30 **0.228**，加入 PanoCity、SO(3) augment、global points、球面 embedding 后，完整组合 **0.427**（该表的实验配置与主结果表不同，不应混用）。
- **Checkpoint 复现注意：**官方代码明确指出论文表格对应第一阶段 **model_lowres.pt**；后发布 **model.pt** 是高分辨率第二阶段版本，官方建议一般使用后者，但不能用它的结果声称原表格复现。

**一手来源：** https://arxiv.org/html/2603.17571v1；https://github.com/YijingGuo-June/PanoVGGT；https://huggingface.co/datasets/YijingGuo/PanoCity 。

## 优点

- 不依赖预先切割 ERP 为多个透视图，预测 camera/depth/global pointmaps 的流程完整。
- 球面边界、三轴旋转以及无序多视角世界原点都有明确建模。
- 具有公开的合成 panoramic pose+depth 数据、正式会议论文和可用 checkpoint；适合建立可复现 camera-only 对照。

## 局限

- PanoCity 来自合成城市，不包含滑雪者的全身 3D 动作与人体—相机耦合监督；动态大前景、雪地低纹理、鱼眼拼接和高速度泛化尚未被独立验证。
- 主 pose 评价为 **rotation / translation direction angular error** 与 AUC，而不是完整 metric ATE/RPE 或长期 scale drift。
- 模型本身不恢复 human pose，没有 human-to-camera mutual refinement，也不直接解决长序列 loop closure。
- 模型的 scale-aligned objective 与 3-view 评价协议限制了绝对尺度和任意长序列结论。

## 个人评价

**推断：**PanoVGGT 是目前研究链里“原生球面输入 → camera + depth + pointmap”的非常有意义的独立 baseline。应将它与 ViPE、360DVO、Wid3R、以及 perspective-crop VGGT 并列；若对其推理结果再加入人体约束，必须单独展示 camera ATE/RPE 而非只看人体 W-MPJPE。

**推断：**真实双 360° 滑雪中，应首先对静态背景做 dynamic masking，并严格比较两个物理相机的轨迹，避免将同一全景裁出的虚拟同中心视角当作有 stereo baseline 的独立相机。

## 与我的研究关联

- **双 360° 滑雪：**可先作为无人体先验的 panorama scene/camera 前端；人体从 perspective crops 的 SAM 3D Body 得到局部关节点后，再研究 joint-wise / axis-wise 可靠性是否反向改善物理相机姿态。
- **多视角几何：**评估 ERP 原生、perspective crop、dual-fisheye 与基于真实多相机基线的差别；从相机的 R/t/scale、重投影、depth 到 W-MPJPE 建立分层误差表。
- **world HMR：**随机 anchor 可用于输入排序不敏感的多相机观察融合，但**推断：**最终时间序列仍需固定世界参考系和轨迹对齐。
- **建议实验：**GT-camera / PanoVGGT-camera / Wid3R-camera / 360DVO-camera × pose-only / +human-assisted camera correction 的正交消融。
- **推荐精读：**Sec. 4.2–4.5、Sec. 5.1、5.4；优先复现 PanoCity pose AUC 和 camera geometry 消融，并记录 checkpoint 差异。

## 后续阅读

- [Wid3R](../360-vision/2026-wid3r-wide-fov-camera-model-conditioning.md)：球面/鱼眼相机模型 conditioning。
- [360DVO](../360-vision/2026-360dvo.md)：ERP visual odometry 与 omnidirectional BA。
- [ViPE](../multiview-geometry/2025-vipe.md)：真实多种相机模型的 camera geometry。
- [Holo360D](../360-vision/2026-holo360d.md)：真实 panorama/LiDAR 深度、轨迹数据。
- **待核验/复现：**双 360 动态人体遮挡条件下的 camera RPE、scale drift、长序列处理时间和不同 checkpoint 对误差的影响。
