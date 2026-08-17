---
title: "AnyCam: Learning to Recover Camera Poses and Intrinsics from Casual Videos"
authors: "Felix Wimbauer, Weirong Chen, Dominik Muhle, Christian Rupprecht, Daniel Cremers"
venue: "CVPR 2025"
year: 2025
reading_date: 2026-08-18
status: skimmed
tags:
  - multiview
  - camera-pose
  - camera-calibration
  - dynamic-video
  - sfm-slam
---

# AnyCam: Learning to Recover Camera Poses and Intrinsics from Casual Videos

## 基本信息

- **作者：** Felix Wimbauer, Weirong Chen, Dominik Muhle, Christian Rupprecht, Daniel Cremers
- **会议/期刊：** CVPR 2025
- **年份：** 2025
- **阅读日期：** 2026-08-18
- **阅读状态：** `skimmed`
- **标签：** `multiview`, `camera-pose`, `camera-calibration`, `dynamic-video`, `sfm-slam`
- **论文：** https://openaccess.thecvf.com/content/CVPR2025/html/Wimbauer_AnyCam_Learning_to_Recover_Camera_Poses_and_Intrinsics_from_Casual_CVPR_2025_paper.html
- **代码：** https://github.com/Brummi/anycam
- **数据集：** 暂无（未发布专属数据集；训练使用 YouTube-VOS、RealEstate10K、WalkingTours、OpenDV、EpicKitchens）
- **项目主页：** https://fwmb.github.io/anycam/

## 一句话总结

AnyCam 用一个基于 Transformer 的前馈网络从动态 casual video 直接估计相对 6DoF camera poses、单序列焦距和逐像素不确定性，并利用预训练 depth/flow 与 uncertainty-aware reprojection loss 在无相机轨迹标签条件下训练，最后用轻量 bundle adjustment 抑制长时漂移。

## 研究问题与动机

传统 SfM/SLAM 在静态、纹理充分且相机内参已知时非常可靠，但 casual video 往往同时包含大面积动态人物/车辆、未知焦距、模糊和不规则相机运动。已有动态场景方法常要求监督轨迹、已知内参，或在测试时进行耗时的逐视频优化。AnyCam 的核心问题是：能否直接从大量无标注真实视频中学习“合理相机运动”的数据先验，使模型在不同动态场景中 zero-shot 恢复 camera extrinsics 和 intrinsics，并减少动态对象对几何估计的干扰。

## 核心方法

输入是一段短视频及由现成模型预测的 metric depth 和 optical flow。每帧经过 DINOv2/ViT 特征提取后，跨帧 Transformer 交换信息，输出 `n-1` 个 pose tokens 和一个 sequence token。pose head 预测相邻帧的 6DoF 相对位姿以及逐像素 uncertainty map；sequence head 从 32 个焦距候选中估计整段视频最可能的单一 focal length。

训练不使用 GT camera trajectory。模型根据 depth、相机变换和 focal length 将像素重投影到另一帧，与 optical flow 建立一致性损失；uncertainty 对动态或不可靠像素自动降权。同时加入 forward/backward pose consistency 和用于焦距假设选择的分布损失。推理阶段再以 uncertainty-weighted pixel tracks 做 sliding-window bundle adjustment，降低相对位姿连续累积产生的 drift。

该设计与显式“先做人/车 segmentation 再去掉动态区域”不同：uncertainty 是从几何不一致性中学习，理论上可对未知类别的动态物体降权。

## 数据集与评价指标

训练混合五个无 3D GT 数据集：YouTube-VOS 3471 sequences / 87K frames、RealEstate10K 6929 / 90K、WalkingTours 278 / 53K、OpenDV 532 / 105K、EpicKitchens 167 / 100K；按论文 Table 2 直接相加，合计约 11,377 sequences、435K frames。测试数据均未参与训练：Sintel 14 sequences / 629 frames、TUM-RGBD dynamics 8 / 720；另在 Davis 90 / 6118、Aria Everyday Activities 30 / 19,200、Waymo 64 / 3067 上做定性泛化展示。

相机轨迹使用 ATE、RPE-trans 和 RPE-rot；焦距估计使用 absolute focal error（AFE）与 relative focal error（RFE）。输入统一采样为 336×336，训练先使用长度 2 再长度 8 的序列；推理一次最多输入 100 帧。模型使用 32 个从 0.1H 到 3.5H 的焦距候选。

## 主要结果

在 Sintel 上，AnyCam 不做 refinement 时 ATE / RPE-trans / RPE-rot 为 0.099 / 0.045 / 0.567；加入 uncertainty-aware refinement 后改善为 0.078 / 0.031 / 0.427。对比 LeapVO 为 0.089 / 0.066 / 1.250，MonST3R 为 0.108 / 0.042 / 0.732，CasualSAM 为 0.141 / 0.035 / 0.615。TUM-RGBD dynamics 上最终结果为 0.056 / 0.005 / 0.927，优于 MonST3R 的 0.063 / 0.009 / 1.217。

Sintel 焦距恢复中，AnyCam 的 AFE 为 252.2 px、RFE 为 0.181，而 UniDepth 为 447.4 px / 0.357，Dust3r 为 434.0 px / 0.364。消融显示简单 BA 并不能自动带来提升：长序列模型不 refinement 为 ATE 0.099，普通 refinement 反而为 0.136，只有加入 uncertainty weighting 后才下降到 0.078。

论文报告 50 帧视频中，depth+flow 预处理约 15 s，AnyCam 前馈 camera trajectory 约 5 s，test-time refinement 约 90 s；因此“前馈预测快”与“完整 refined pipeline 的总耗时”需要分开理解。

## 优点

- 不需要 GT camera trajectory 或 3D labels，就能利用大量真实动态视频训练 camera-motion prior。
- 同时恢复 extrinsics 和未知焦距，比只假设已标定 camera 的 VO 方法更适合互联网或随手拍视频。
- uncertainty 不只是置信度输出，而是同时用于训练和测试时 BA，消融证明其对动态场景优化有效。
- 训练域覆盖 indoor、outdoor、driving、egocentric 等多种相机运动，且主要定量测试是 unseen datasets，适合考察 cross-domain camera generalization。

## 局限

论文明确指出 AnyCam 依赖预训练 UniDepth 和 UniMatch；depth/flow 在极端光照或缺乏场景上下文时失败会传递到相机估计。即使有 refinement，长序列仍可能漂移，因为当前局部 tracks 长度有限；模型也可能在训练分布外的非常规相机运动上失败。

此外，方法采用整段视频内保持不变的简化 pinhole camera model，并将内参压缩为单一 focal length。**推断：**这意味着原始 fisheye 或 360° ERP 不能被当作普通 perspective frame 直接送入 AnyCam；对于 360° 自拍，更合理的做法是先切已知投影参数的 perspective crops，或重新设计 spherical/fisheye projection model。

## 个人评价

AnyCam 对当前研究最大的价值是提供了一个与 SLAM 不同的 camera estimator：它通过大规模视频学习动态场景中的运动先验，而不是完全依赖静态点几何。尤其值得注意的是，普通 BA 在消融中并不稳定，真正有效的是 uncertainty-aware BA，这提示“camera joint optimization”不能只写成追加一个优化器，而应明确说明哪些观测应该被信任、哪些动态人体区域应该降权。

## 与我的研究关联

可以把 AnyCam 作为 ViPE、SLAM/VGGT 类方法之外的 camera branch baseline，特别适合当前“移动相机 → camera trajectory → 3D human kpt/world reconstruction → camera-human joint refinement”的路线。建议统一比较 `GT camera / ViPE / AnyCam feed-forward / AnyCam refined / human-camera jointly refined`，同时报告 camera ATE/RPE 与人体 W-MPJPE/root trajectory error。

对于 360° 自拍，虚拟 perspective crop 的 intrinsic 和相对旋转其实是已知的，因此可以进一步做一个很关键的消融：让 AnyCam预测全部内外参，与固定已知 virtual intrinsics、只估计物理 camera trajectory 的版本比较。**推断：**如果把人体区域的 uncertainty 与 3D keypoint reprojection residual 联合起来作为权重，可能比纯背景 camera estimation 更适合高速体育场景，也能形成更明确的 camera-human mutual refinement 贡献。

## 后续阅读

- 优先精读 Sec. 3.2 的 Transformer camera prediction、动态区域 uncertainty training、test-time refinement，以及 Tables 1、3、4。
- 复现时至少拆分 `feed-forward / naive BA / uncertainty-aware BA` 三组，避免把优化本身与 uncertainty 的作用混为一谈。
- 在 3DPW、EgoHumans 或自采 moving-camera 数据上与 ViPE/SLAM 比较，并专门分析人物占画面比例、相机旋转速度、背景纹理和序列长度对 ATE/RPE 的影响。
