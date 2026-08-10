---
title: "Mocap-2-to-3: Multi-view Lifting for Monocular Motion Recovery with 2D Pretraining"
authors: "Zhumei Wang, Zechen Hu, Ruoxi Guo, Huaijin Pi, Ziyong Feng, Liang Zhang, Mingtao Pei, Siyuan Huang"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-11
status: skimmed
tags:
  - 3d-human-pose
  - monocular-motion-recovery
  - multiview-lifting
  - diffusion
  - world-coordinate
  - camera-geometry
---

# Mocap-2-to-3: Multi-view Lifting for Monocular Motion Recovery with 2D Pretraining

## 基本信息

- **作者：** Zhumei Wang, Zechen Hu, Ruoxi Guo, Huaijin Pi, Ziyong Feng, Liang Zhang, Mingtao Pei, Siyuan Huang
- **会议/期刊：** CVPR 2026（Highlight）
- **年份：** 2026
- **arXiv：** arXiv:2503.03222；v1 提交于 2025-03-05，当前论文版本 v6 修订于 2026-03-13
- **阅读日期：** 2026-08-11
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `monocular-motion-recovery`, `multiview-lifting`, `diffusion`, `world-coordinate`, `camera-geometry`
- **论文：** [arXiv](https://arxiv.org/abs/2503.03222) · [CVPR 2026 Open Access](https://openaccess.thecvf.com/content/CVPR2026/html/Wang_Mocap-2-to-3_Multi-view_Lifting_for_Monocular_Motion_Recovery_with_2D_Pretraining_CVPR_2026_paper.html)
- **DOI：** [10.48550/arXiv.2503.03222](https://doi.org/10.48550/arXiv.2503.03222)
- **代码：** [WangZhumei/Mocap2to3](https://github.com/WangZhumei/Mocap2to3)
- **数据集：** 暂无（未发布专用数据集；训练/评估使用多个公开人体运动数据集）
- **项目主页：** [Mocap-2-to-3](https://wangzhumei.github.io/mocap-2-to-3/)

## 一句话总结

Mocap-2-to-3 针对单目 2D→3D lifting 容易受有限 3D 训练数据和深度/尺度歧义影响的问题，把三维运动恢复重写成“生成多个虚拟相机视角的 2D 运动并进行几何三角化”的过程，先用大规模 2D 动作进行 diffusion 预训练，再用有限 3D 数据约束多视角一致性，并通过局部姿态—全局运动解耦与 ground-plane pointmap 恢复 metric-scale 的世界坐标人体运动。

## 研究问题与动机

传统 monocular 3D human motion recovery 通常直接从图像或 2D skeleton 回归三维人体。此类模型一方面依赖昂贵、受控环境中的 3D mocap 数据，面对训练集之外的动作时容易失效；另一方面，单目输入天然存在深度与绝对尺度歧义，因此不少 world-grounded 方法只能恢复经过初始帧对齐的相对轨迹，而不是物理世界中的绝对位置。

作者的核心判断是：2D 动作数据比 3D mocap 更容易获得、动作覆盖更丰富，而“同一个 3D 动作在不同相机中的 2D 投影”本身构成了强几何约束。因此，与其直接学习 2D→3D 回归，不如先学习任意视角下的 2D motion distribution，再在多视角 fine-tuning 阶段学习跨视角一致性，最后利用已知 camera calibration 做 triangulation。

## 核心方法

### 1. 单视角 2D Motion Diffusion 预训练

第一阶段使用 Transformer-based diffusion model 学习 2D motion。模型输出 `T × J × 2` 的二维关键点序列，并通过随机相机视角投影获得不同 viewpoint 下的动作。该阶段的目标不是直接预测 3D，而是从大规模、动作多样的二维数据中学习跨动作和跨视角 motion prior。

预训练数据包括从 HumanML3D 投影得到的 2D joints，以及与目标测试域同源的二维数据，例如 RICH training set。论文的消融进一步表明，仅额外加入 **175 条 RICH 域内 2D sequences** 就能继续改善三维恢复结果。

### 2. Multi-view Diffusion Fine-tuning

第二阶段把单视角 diffusion 权重初始化到 Multi-view Diffusion Model，并使用 3D motion 投影得到的多视角 2D ground truth 进行 fine-tuning。训练时设定 **4 个视角**：一个主视角 `V0` 和三个随机采样的虚拟视角。

模型输入主视角 2D motion、Gaussian noise，以及每个视角的 camera intrinsics 与 extrinsics。作者加入 View Attention Layer，使不同虚拟视角的生成结果保持几何一致。因为训练输入不需要真实同步图像，可以对已有 3D motion 做 rotation、translation、pitch、yaw、roll 和 camera-distance 等随机增强，以有限 3D 数据合成大量 camera layouts。

### 3. 局部姿态与全局运动解耦

直接预测全局 2D 坐标时，人体位置变化的幅度远大于骨架细节，容易让 loss 被 root translation 主导。作者因此把运动拆成：

- **local pose：** 在人体 bbox 内归一化并以 root 为中心的关节结构；
- **global movement：** bbox/root trajectory 与 scale。

各视角分别生成这两部分，再还原成 global 2D coordinates。这一表示使网络可以分别学习动作细节与世界空间移动。

### 4. Ground-plane Pointmap Encoding

仅有 camera embedding 时，全局深度/尺度学习仍收敛较慢。作者利用已知 camera intrinsics/extrinsics 与 ground plane，计算每个像素射线与地面的交点，将 `(u,v)` 映射为对应 world-coordinate `(x,y,z)`，形成 pointmap。

Pointmap 经 ResNet-18 编码后，通过 View Attention 与 Cross Attention 注入 multi-view diffusion。它不需要场景 scan 或深度传感器，只依赖 calibration。消融显示 pointmap 可以使 global movement learning 的训练时间降低超过 **50%**。

### 5. 虚拟多视角三角化

推理阶段输入仍然只有一个真实视角的 2D pose sequence。网络生成其他虚拟视角中的一致 2D motions，并恢复各视角 global coordinates；随后使用已知虚拟相机参数进行 multi-view triangulation，得到带 absolute position 的世界坐标 3D skeleton。需要 SMPL 参数时，可以在最终关节上再运行 SMPLify。

## 数据集与评价指标

### 训练数据

- **2D pretraining：** HumanML3D 投影后的二维 joints，以及目标域的二维数据（论文以 RICH training set 为例）。
- **multi-view fine-tuning：** HumanML3D、BEDLAM、Human3.6M；论文说明 HumanML3D 中包含 HumanAct12 和 AMASS 来源的数据。
- 论文没有在实验章节统一给出上述训练集合计的 sequence/frame 总数，因此不对总训练样本量作额外推断。

### 评估数据

- **RICH：** 同时包含室内/室外真实场景，用 SMPL-format ground-truth 2D keypoints 进行主实验；覆盖 sitting、lying down、handstand 等在训练集中相对少见的动作。
- **AIST++：** 用于 COCO-format skeleton 实验，输入由 ViTPose 检出的 2D keypoints，测试高动态 dance motions 与跨 keypoint format 泛化。

### 指标

- camera coordinates：`PA-MPJPE`、root-aligned `MPJPE`；
- world coordinates：`W-MPJPE`（前两帧对齐）、`WA-MPJPE`（全序列对齐）、`Abs-MPJPE`（完全不对齐）；
- trajectory/motion quality：`Troot`、`Accel`、`Jitter`、`FS`（foot sliding）。
- 论文注明所有 position errors 的单位均为 mm。

## 主要结果

### RICH：绝对世界坐标恢复

在 RICH 上，Mocap-2-to-3 报告：

| 方法 | PA-MPJPE ↓ | MPJPE ↓ | W-MPJPE ↓ | WA-MPJPE ↓ | Abs-MPJPE ↓ | Accel ↓ | Jitter ↓ | FS ↓ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GVHMR | 33.6 | 58.9 | 110.0 | 68.4 | - | 3.8 | 10.5 | 2.5 |
| GVHMR + SMPLify | 30.7 | 58.7 | 109.4 | 68.6 | 430.4 | 3.7 | 9.4 | 5.6 |
| **Mocap-2-to-3** | **26.2** | **39.6** | **82.6** | **50.1** | **156.8** | **2.5** | **8.0** | 3.5 |

相较带 calibrated camera pose 和 SMPLify 的 GVHMR+SMPLify，W-MPJPE 从 109.4 降到 **82.6 mm**，完全不做 world alignment 的 Abs-MPJPE 从 430.4 降到 **156.8 mm**。这说明它的优势不仅是 pelvis/root-aligned pose，而是 metric world localization。

### AIST++：检测器输入的 COCO skeleton

使用 ViTPose 2D detections 时，Mocap-2-to-3 在 AIST++ 上达到 **60.1 mm PA-MPJPE、90.9 mm MPJPE、61.8 mm Troot**。作为同类 2D-to-3D lifting 方法，MVLift 分别为 79.2、110.7 和 67.6；GVHMR+SMPLify 为 62.2、102.8 和 112.3。

### 关键消融

RICH 上：

- 不做 local/global decoupling：PA-MPJPE 65.1，W-MPJPE 161.2；
- 无 pointmap，3.5k epochs：45.8 / 121.8；
- 无 pointmap，8k epochs：33.4 / 103.7；
- 有 pointmap，3.5k epochs：30.5 / 88.6；
- 再加入 175 条 RICH 2D sequences：**26.2 / 82.6**。

因此 pointmap 主要改善收敛效率，而少量域内二维动作可以进一步改善 OOD motion quality。

## 优点

- 把“单目 3D lifting”转换成“虚拟多视角 2D generation + triangulation”，把深度歧义转化为更明确的 camera geometry 问题。
- 能充分利用容易获得的 2D motion，而不是完全依赖昂贵的 3D mocap。
- 将 local pose 与 global trajectory/scale 解耦，避免世界位置学习压制动作细节。
- 明确评估 Abs-MPJPE、W-MPJPE 等 world-coordinate 指标，而不是只报告 PA-MPJPE。
- 虚拟相机增强可以系统随机化 camera rotation、translation、distance，为跨 camera-layout 泛化提供了直接训练机制。
- 官方代码与复现资产已公开。

## 局限

- **论文明确指出：** 最终 3D 质量明显依赖输入 2D skeleton；原始视频中的检测错误会向 lifting 阶段传播，当前方法不负责从 RGB 纠正严重错误的 2D detections。
- **适用条件：** 方法需要 calibrated camera poses 才能构造虚拟多视角几何、ground pointmaps 和 absolute world triangulation，因此并不是 calibration-free 方法。
- **论文明确指出：** 当前没有专门的 foot-sliding constraint，作者将其列为后续改进方向。
- **评估边界：** 主实验主要集中在 RICH 与 AIST++；虽然两者包含真实室内/室外与高动态动作，但仍不足以证明在医疗、360° 畸变或极端运动相机条件下直接泛化。

## 个人评价

这篇论文最有价值的地方不是 diffusion 本身，而是重新定义了“单目输入如何利用多视角几何”。它虽然推理时只有一个真实 camera view，却通过已知 virtual camera geometry 生成多视角 observations，再交给经典 triangulation，形成一个非常清晰的“learned prior + explicit geometry”组合。

与很多只追求 PA-MPJPE 的方法相比，论文把 Abs-MPJPE、W-MPJPE、root trajectory 和 motion quality 同时纳入评价，更适合真正要求世界坐标人体运动的体育、临床行为或人—环境交互任务。

阅读优先级：**很高**。

## 与我的研究关联

**直接关联。** 对多视角/360° 人体重建而言，这篇工作提供了一个与“每个 view 独立预测 3D pose 后再融合”完全不同的 baseline：先保留较可靠的 2D joint observations，再利用 camera geometry 生成/约束多视角观测，最后 triangulate 到统一 3D world。

值得直接借鉴或复现的部分包括：

- 比较 `per-view 3D pose fusion` 与 `2D virtual-view synthesis + triangulation`；
- 在 Unity 中随机化 camera yaw/pitch/roll、translation、distance，建立 camera-layout robustness benchmark；
- 在已有 fusion network 中加入 ground-plane pointmap 或 camera geometry feature，比较是否改善 world-coordinate error；
- 将评价从 PA-MPJPE 扩展为 Abs-MPJPE、W-MPJPE、root trajectory 和 foot sliding；
- 单独评估 2D detector noise 对最终 3D 的传播，并与论文“GT 2D vs detector 2D”的思路对应。

以上为基于论文方法与当前研究方向提出的**推断性实验建议**，不是论文已经验证过的 360° 或临床结论。

## 后续阅读

- 对比 Mocap-2-to-3、LAMP、传统 algebraic triangulation 和 pose-level learned fusion 在同一已标定 synthetic benchmark 上的误差来源。
- 阅读其 supplementary 中虚拟 camera sampling 与 pointmap 构造细节。
- 测试 camera calibration perturbation 对 Abs-MPJPE/W-MPJPE 的敏感性。
- 检查其 2D pretraining 思路能否用于体育动作中少见、快速或强遮挡的 motion prior 学习。
