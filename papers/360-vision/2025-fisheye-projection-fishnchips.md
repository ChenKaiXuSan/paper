---
title: "Systematic Comparison of Projection Methods for Monocular 3D Human Pose Estimation on Fisheye Images"
authors: "Stephanie Käs, Sven Peter, Henrik Thillmann, Anton Burenko, David Benjamin Adrian, Dennis Mack, Timm Linder, Bastian Leibe"
venue: "IEEE International Conference on Robotics and Automation (ICRA 2025); arXiv:2506.19747"
year: 2025
reading_date: 2026-11-16
status: skimmed
tags:
  - 360-vision
  - fisheye
  - camera-projection
  - 3d-human-pose
  - multi-view-benchmark
  - dataset
---

# Systematic Comparison of Projection Methods for Monocular 3D Human Pose Estimation on Fisheye Images

## 基本信息

- **作者：** Stephanie Käs, Sven Peter, Henrik Thillmann, Anton Burenko, David Benjamin Adrian, Dennis Mack, Timm Linder, Bastian Leibe
- **会议/期刊：** IEEE International Conference on Robotics and Automation (ICRA 2025)，正式会议论文；arXiv 版本于 2025-06-24 首次提交
- **年份：** 2025
- **阅读日期：** 2026-11-16
- **阅读状态：** `skimmed`
- **标签：** `fisheye`、`360-vision`、`projection`、`3d-human-pose`、`dataset`
- **论文：** https://arxiv.org/abs/2506.19747 （官方全文及 arXiv DOI: https://doi.org/10.48550/arXiv.2506.19747）
- **代码：** 暂无（未核验到与本文一致的官方实现）
- **数据集：** https://www.vision.rwth-aachen.de/page/fishnchips （作者所在实验室官方 FISHnCHIPS 页面；指向 https://huggingface.co/datasets/skaes/FISHnCHIPS/ ，目前读取提示 401，未验证下载权限）
- **项目主页：** https://www.vision.rwth-aachen.de/page/fishnchips
- **论文状态：** 已在 ICRA 2025 发表；2025 年重要方法/数据集补读
- **价值类型：** Baseline / Method Module / Dataset / Related Work
- **阅读优先级：** 高

## 一句话总结

对于广角人体，固定使用 pinhole perspective crops 不能保证 3D HPE 精度：作者将 equidistant / double-sphere / cylindrical 等投影接入 MeTRAbs，在 FISHnCHIPS 实测视场依赖的相对/绝对定位权衡，并给出按人体画面范围切换投影的启发式方案。

## 研究问题与动机

360°/fisheye 相机提供高覆盖率，却使近距离人体跨越较大角视场（FoV），造成 pinhole 裁剪的几何畸变和人体位置失真。现有 3D HPE 多建立于透视训练数据；仅从 ERP 或 fisheye 图像截出透视图，并不意味着大 FoV 关节已得到合理建模。本文系统比较多种 projection model，并提供低机位、仰视、近距离、多人的 fisheye 人体评价资源。

## 核心方法

1. 以已知 fisheye 相机内参为前提，检测/使用人体 2D bounding box，将原始 fisheye 图像按选定 output camera model 重投影为人体 crop。
2. 在 MeTRAbs 3D HPE 中支持 pinhole（PH）、equidistant fisheye（EF）、double sphere（DS）、central cylindrical（CC）与 equidistant cylindrical（EC）；不要求重新训练 MeTRAbs 主干以处理 fisheye，只需兼容相机投影/输出与标注关节映射。
3. 定义 MPJA（关节最大两两张角）及 MBBA（bounding box 的角度展开）刻画人体所覆盖的角视场。Hybrid（H）方法按阈值 110° 或 135° 在 PH 与 DS 间切换；MBBA 可以由 bbox 获得，而 MPJA 依赖 GT joints，实际使用应优先评估前者。
4. 输入：带内参的 fisheye RGB + person bounding box；输出：相机坐标/绝对的 3D joints，并以 MPJPE/PCK 评价相对与绝对误差。

## 数据集与评价指标

- FISHnCHIPS：三处室内场景、7 名受试者、56 条序列、202,240 张 fisheye 图像、138,298 张 pinhole/Kinect 图像；16 个 fisheye 与 11 个 pinhole 相机视角，含近距离、遮挡、低机位、倾斜镜头。
- 3D pseudo-ground truth 由**同步多视角 triangulation** 等方法构建，并非逐关节完全独立 mocap。
- 数据集外还在 MuPoTS-3D 评估重实现的 MeTRAbs，PCK150 absolute/relative 分别 59.5%/89.4%；这不应与 FISHnCHIPS 的成绩混为一谈。
- FISHnCHIPS 主指标：MPJPE、A-MPJPE（mm），PCK150、A-PCK150（%）；后两者带 A 的指标用于绝对位姿评估。主对比使用 **GT person bounding boxes**，排除检测器误差影响。

## 主要结果

- FISHnCHIPS Table IV：PH 的 MPJPE / A-MPJPE 为 **151.6 / 219.4 mm**，DS 为 **147.7 / 205.2 mm**，EF 为 **147.8 / 205.3 mm**，EC 为 **147.4 / 204.4 mm**。
- DS 的 A-PCK150 **45.5%**，PH 为 **41.5%**；但 PH 在该测试集的相对 PCK150 **65.1%**，高于 DS 的 **63.6%**，因此不能简单认为 DS 在所有指标都最佳。
- Hybrid H（MBBA 阈值 110°）的相对 MPJPE **145.1 mm**，但 A-MPJPE 为 **210.2 mm**；相对误差改善不能等同绝对定位改善。
- 按人体关节张角分组：较小 FoV 时 PH 合适，较大 FoV（论文分析约 >120°）时 fisheye 模型更稳健。论文展示 PH 在大视场区域误差增长明显。

## 优点

- 直接针对 spherical/fisheye 几何和 3D HPE 的接口问题，提供可操作的 camera projection 消融，而非仅定性讨论 ERP distortion。
- 同时报告 relative/absolute HPE 指标，揭示相对姿态改善与绝对定位可能不一致。
- FISHnCHIPS 包含非标准低机位与近距离人体，使训练于透视数据的 HPE 泛化边界更容易暴露。

## 局限

- 作者方法需要**已知原始 fisheye 相机内参**；并未恢复移动 camera 的 R/t、intrinsics 或 metric scale。
- 主实验采用 GT bounding boxes，无法代表自动检测、多人跟踪与高速模糊下的端到端性能。
- 3D 标签基于多相机 triangulation 的 pseudo-GT；只有 7 名受试者、以室内受控场景为主，非户外雪地/滑雪 360 视频。
- 官方 dataset 页面可访问，但 Hugging Face 资源本次请求返回 401，不能声称已下载或独立复现；本文代码未核验到公开入口。

## 个人评价

**事实：**DS 和 EF 在大 FoV 的绝对定位上较 PH 有优势，但混合式 PH/DS 在相对 MPJPE 上最好；必须明确评价坐标系，不能用单一 PCK 推广所有情形。**推断：**若只选固定 FoV 透视 crop，较近的人体或跨 ERP seam 的姿态可能成为世界坐标重建的主要误差源。

## 与我的研究关联

**推断：**对双 360° 滑雪/自拍多透视研究，应把 `ERP → virtual pinhole → SAM3D/MeTRAbs` 作为可复现基线，并增加 `double-sphere/equidistant ray-consistent crop` 与 `bbox-angle adaptive crop`。分离同中心虚拟视角与不同物理 360 相机真实 baseline；对同一帧记录人体角视场、遮挡、seam 距离、局部/全局 MPJPE、world root/尺度误差。特别需要在双物理相机融合前检查几何一致性，避免 reliability weighting 只是在补偿投影模型错误。

## 后续阅读

- 优先复现：Section III（投影与 MeTRAbs 接口）、Section V（FISHnCHIPS 数据/骨架适配）、Sections VI–VII 的 Table IV 与按 FoV 分桶评价。
- **推断：**在球面视角中测试 `PH crop → DS crop → adaptive crop → cross-view fusion`，并按 <90°、90–120°、>120° 及人体距离分层报告误差；增加 predicted bbox、相机内参扰动与运动模糊消融。
- 核验是否有新的官方代码公开；确认 Hugging Face 数据集的申请/下载权限及真实标注精度。
- 一手来源：arXiv https://arxiv.org/html/2506.19747v1；作者实验室 https://www.vision.rwth-aachen.de/page/fishnchips。
