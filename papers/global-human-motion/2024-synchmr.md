---
title: "Synergistic Global-space Camera and Human Reconstruction from Videos"
authors: "Yizhou Zhao, Tuanfeng Y. Wang, Bhiksha Raj, Min Xu, Jimei Yang, Chun-Hao Paul Huang"
venue: "CVPR 2024"
year: 2024
reading_date: 2026-09-09
status: skimmed
tags:
  - global-human-motion
  - moving-camera
  - human-aware-slam
  - metric-scale
  - scene-human-camera
  - monocular-video
---

# Synergistic Global-space Camera and Human Reconstruction from Videos

## 基本信息

- **作者：** Yizhou Zhao, Tuanfeng Y. Wang, Bhiksha Raj, Min Xu, Jimei Yang, Chun-Hao Paul Huang
- **会议/期刊：** IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR) 2024, pp. 1216–1226
- **年份：** 2024
- **正式发表日期：** 2024-06-17
- **阅读日期：** 2026-09-09
- **阅读状态：** `skimmed`
- **标签：** `global-human-motion`, `moving-camera`, `human-aware-slam`, `metric-scale`, `scene-human-camera`, `monocular-video`
- **DOI：** 10.48550/arXiv.2405.14855
- **论文：** https://openaccess.thecvf.com/content/CVPR2024/html/Zhao_Synergistic_Global-space_Camera_and_Human_Reconstruction_from_Videos_CVPR_2024_paper.html
- **代码：** 暂无
- **数据集：** https://sanweiliti.github.io/egobody/egobody.html （主要 world-space 评测数据之一；论文未发布专属新数据集）
- **项目主页：** https://paulchhuang.github.io/synchmr/

## 一句话总结

SynCHMR 用 camera-frame HMR 的人体尺度和位置线索校准 monocular depth 并构建 human-aware metric SLAM，再让恢复出的 dense scene 反向条件化 SMPL 时序去噪，从单目移动视频中在同一世界坐标恢复 camera trajectory、人体运动和场景几何。

## 研究问题与动机

单目 SLAM 通常只能得到 up-to-scale 的 camera trajectory / scene geometry，而 HMR 虽然含有人体 metric-scale prior，却常与 camera 和 scene 独立运行。直接串联时，SLAM 的尺度、深度与动态前景误差会传递到 world-space human motion；同时人体自身携带的尺寸和空间位置线索没有被充分用于 camera estimation。

SynCHMR 因此建立双向协同：先用人体恢复结果帮助 camera/scene 获得 metric scale，再用更可靠的 camera 与 dense scene 对人体世界坐标运动进行 scene-aware refinement。

## 核心方法

### Human-aware Metric SLAM

方法以 DROID-SLAM 为 camera/geometry backbone，并引入视频适配后的 ZoeDepth 与人体语义 mask。camera-frame HMR 提供人体的 metric 尺寸和空间位置，用于校准 monocular depth 的尺度/范围，从而构造更接近 metric 的深度约束，缓解 monocular SLAM 的 scale、depth 和 dynamic ambiguity，最终得到 metric-scale camera trajectory 与 dense scene point cloud。

这一步使 human information 实际参与 scene depth / camera trajectory 的 metric calibration，因此属于明确的 human-aware camera estimation 先例。

### Scene-aware SMPL Denoiser

初始 camera-frame SMPL 通过估计的 camera extrinsics 放入 world frame，再由 scene-conditioned temporal denoiser refinement。论文使用 6-layer Transformer Decoder，并将 RGB、XYZ point cloud 与 subject mask 等 scene evidence 编码后条件化 SMPL 参数残差预测，使人体运动同时满足时间连续性和动态场景约束。

## 数据集与评价指标

- **3DPW：**用于 local pose 训练/评估，主要报告 PA-MPJPE。
- **EgoBody：**用于 SMPL denoiser 训练以及 human/camera world-space 评估。EgoBody 官方数据包含 125 个序列、36 名受试者、15 个室内场景。
- **EMDB：**用于增加 motion diversity，并利用其 global camera trajectory 评价 SLAM。
- **输入：**moving-camera monocular RGB video。
- **输出：**metric camera trajectory、dense scene point cloud、world-frame SMPL human motion。
- **人体指标：**PA-MPJPE、FA-MPJPE、WA-MPJPE、Acceleration Error。
- **相机/场景指标：**camera ATE，以及 depth 的 δ1、REL、RMSE。论文 Table 3 的 ATE 表头未显式给出单位，因此这里不自行推断单位。

Scene-aware SMPL Denoiser 在 3DPW-Train、EgoBody-Train 与 EMDB 联合数据上训练 100k steps，batch size 16；训练 temporal window 为 64–128，推理 T=100。

## 主要结果

- **3DPW-Test：**完整 SynCHMR PA-MPJPE 为 **52.4 mm**；DROID-SLAM + SLAHMR(PHALP+) 为 55.9 mm，SLAHMR(4DHumans init) 为 57.4 mm。
- **EgoBody-Val：**完整模型 PA/FA/WA-MPJPE 为 **57.7 / 115.1 / 81.1 mm**，Acceleration Error 为 64.8 mm/frame²；SLAHMR(PHALP+) 为 79.1 / 141.1 / 101.2 mm。
- **EgoBody-Test：**完整模型为 **61.3 / 122.1 / 84.6 mm**；SLAHMR(PHALP+) 为 63.1 / 163.9 / 99.4 mm。SynCHMR 的 test Acc Error 69.4 高于 SLAHMR 的 31.7，说明 global position 改善不等于所有动态指标同时改善。
- **Camera / depth ablation：**EgoBody 上 RGB-only SLAM 的 ATE 为 80.9；加入 ZoeDepth+、Mask 与 human-based depth calibration 后降为 **26.4**。EMDB 上对应 ATE 从 400.3 降为 **107.0**。
- **效率：**EgoBody 表中完整 pipeline 约 **5 min / 100 images**，而 SLAHMR 约 40 min / 100 images。

## 优点

- 明确展示 **human → camera/scene** 与 **scene/camera → human** 的双向协同，而不是简单串联 SLAM 与 HMR。
- 同时评价 camera ATE 与 world-space human errors，能检验人体先验是否真的改善 camera estimation。
- 同时输出 metric camera、dense scene 和 world-frame human，为后续 contact / scene penetration / biomechanics constraint 提供统一坐标基础。
- 相比 SLAHMR 的长时优化，运行成本明显降低。

## 局限

- 论文仍用 `(W+H)/2` 近似 focal length，因此并未真正联合解决未知或变化 intrinsics。
- 人体尺寸被用于 depth calibration；当 body model 对目标体型描述不足时，metric calibration 可能不可靠。
- 方法主要按 pinhole monocular video 设计，没有验证 fisheye / ERP / 360° projection。
- 动态 point cloud 如何作为稳定 scene constraint 仍是开放问题。
- EgoBody 主要是室内 head-mounted interaction，与长距离、高速、弱纹理 outdoor sports 存在明显 domain gap。
- **推断：**Human-aware depth calibration 虽能让人体改善 SLAM，但其反馈主要经 depth/scale calibration 进入 camera，不等价于直接优化逐帧 camera rotation/translation 的现代 learned residual 或 spherical factor。

## 个人评价

这是 camera-human mutual refinement 方向应补齐的强经典 baseline。它说明“用人体帮助 monocular camera metric reconstruction”在 CVPR 2024 已经有清晰证据，因此后续方法不宜把 novelty 写成首次 human-assisted camera estimation。

更值得推进的 gap 是：从 pinhole/近似 intrinsics 扩展到 dual-360 / fisheye，显式建模 camera uncertainty 与 human uncertainty，并在长距离高速场景中证明人体约束不仅改善 W-MPJPE，也能持续降低 ATE/RPE/scale drift。

## 与我的研究关联

对于 moving-camera skiing / dual-360，可将 SynCHMR 放在 `camera-only SLAM → human-aware metric SLAM → explicit camera-human joint refinement` 的核心 baseline 位置。

**推断：**一个直接可复现的实验是：`360DVO / PanoAir / MASt3R-SLAM camera-only → + human metric depth/scale calibration → + skeleton reprojection / bone-length / contact / velocity constraints → + dual-360 cross-view consistency`。应同时报告 camera ATE/RPE/scale drift 与 human W-MPJPE/RTE，并单独检查 high-order dynamics，避免只用最终 pose error 掩盖 camera 是否真正变好。

对于单 360 perspective crops，不能直接照搬其 pinhole camera model；更合理的是保留“人体提供 metric cue”的思想，将几何项改写为 spherical/fisheye-consistent formulation。

## 后续阅读

- BodySLAM++：经典 visual-inertial factor graph 中显式 human residual → camera state。
- SLAHMR：camera-human motion decoupling 与 world trajectory optimization。
- Humans as Checkerboards / WHAC：人体 contact 或 motion 用于 camera metric scale。
- Human3R / SHOW / JOSH：从 shared representation、feed-forward coupling 到 joint optimization 的后续路线。
- **计划实验：**人为注入 focal、rotation、translation、scale drift，测试 SynCHMR-style human metric calibration 与显式 human residual 对 camera ATE/RPE 的独立贡献。
