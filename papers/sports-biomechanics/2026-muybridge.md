---
title: "MuyBridge: Mobile Human Center-of-Mass Estimation from Monocular Video via Sparse Fusion"
authors: "Aidan Bradshaw, Marco Giordano, David Rode, Andreas Habersack, Elif Basokur, Annika Kruse, Markus Tilp, Michele Magno, Peter Wolf, Luca Benini, Christoph Leitner"
venue: "arXiv:2609.02854"
year: 2026
reading_date: 2026-09-05
status: skimmed
tags:
  - sports-biomechanics
  - center-of-mass
  - monocular-video
  - mobile
  - metric-depth
  - pose-estimation
---

# MuyBridge: Mobile Human Center-of-Mass Estimation from Monocular Video via Sparse Fusion

## 基本信息

- **作者：** Aidan Bradshaw, Marco Giordano, David Rode, Andreas Habersack, Elif Basokur, Annika Kruse, Markus Tilp, Michele Magno, Peter Wolf, Luca Benini, Christoph Leitner
- **会议/期刊：** arXiv preprint, arXiv:2609.02854
- **年份：** 2026
- **提交日期：** 2026-09-02
- **阅读日期：** 2026-09-05
- **阅读状态：** `skimmed`
- **标签：** `sports-biomechanics`, `center-of-mass`, `monocular-video`, `mobile`, `metric-depth`, `pose-estimation`
- **DOI：** 10.48550/arXiv.2609.02854
- **论文：** https://arxiv.org/abs/2609.02854
- **代码：** https://github.com/Abradshaw1/Muybridge
- **数据集：** 暂无（论文使用 AthletePose3D 进行评测，未发布新的专用数据集）
- **项目主页：** https://github.com/Abradshaw1/Muybridge

## 一句话总结

MuyBridge 将轻量 2D pose、低频单目 metric depth 与人体测量学/地面接触/弹道物理先验进行解析式稀疏融合，在 iPhone 15 上以 pose 63 FPS、depth 2.86 Hz 的异步方式恢复运动员 metric 3D segmental center-of-mass（CoM）轨迹。

## 研究问题与动机

体育与临床运动分析经常需要 whole-body center of mass，但现有方案往往依赖 marker-based mocap、多相机三角测量或高算力 3D HMR；即使单目 3D pose 的关节误差较低，也不一定能给出可靠的 metric distance 和具有解剖意义的 CoM。论文关注的是一个更部署导向的问题：能否只用手机单目视频，在不进行 3D/task-specific supervision 的情况下，把快速 2D pose 与较慢但提供 metric range 的 monocular depth 结合起来，再利用人体比例和简单物理先验稳定地恢复 CoM。

## 核心方法

系统包含三个阶段。第一阶段是 compact RTMPose-style 2D pose network，以 CSPNeXt backbone、gated attention 和 SimCC head 输出 Halpe-26 keypoints；网络经过 pruning 和 INT8 quantization/QAT 以适配 Apple Neural Engine。第二阶段是由 Marigold 蒸馏得到的 single-step monocular depth network，移除大部分文本条件并通过 latent-consistency distillation 将扩散推理压缩到单次 UNet evaluation。

第三阶段是关键的 analytic metric fusion：先通过 camera intrinsics 将 2D keypoint rays 与稀疏 depth samples 结合，再使用 stature-scaled de Leva anthropometric segment lengths 约束人体长度和 metric scale；同时利用 ground-contact cue 与 ballistic cue 对 camera-to-athlete range 进行物理锚定。最终恢复 metric joint positions，并按 segment mass fractions 计算 whole-body CoM。系统采用异步设计：pose 每帧运行，depth 低频更新，fusion 使用最近一次 depth field 保持高频输出。

## 数据集与评价指标

- **AthletePose3D：** 8 名运动员，约 1.3M synchronized multi-view frames；论文评测包含 running、track and field、figure skating。最终评测规模为 1,028 个 sequence-camera pairs、235,183 frames；2D pose 和 depth network 均未针对 AthletePose3D 做 task-specific tuning。
- **2D pose training：** COCO-WholeBody + HICO-DET。
- **Depth training：** distilled Marigold 使用约 1.2M synthetic RGB-D samples（Hypersim + VKITTI2）。
- **主要指标：** 3D CoM MAE、X/Y/Z MAE、camera-to-athlete range AbsRel，以及移动端 latency / FPS / energy。

## 主要结果

在 AthletePose3D 上，MuyBridge 的 running / track-and-field / figure-skating **3D CoM MAE 分别为 187 / 185 / 707 mm**；对应 camera-to-athlete range AbsRel 为 **3.6% / 2.3% / 6.6%**。尽管 skating 的绝对深度误差明显更大，三个运动场景的 vertical CoM error（Y MAE）仍保持在 **33–41 mm**。分轴误差分别为：running 44 / 33 / 166 mm，track and field 94 / 39 / 117 mm，figure skating 167 / 41 / 672 mm，说明主要误差来自深度/距离方向，而垂直 CoM 相对稳定。

在 iPhone 15 上，pose network 为 **15.58 ms / 63 FPS / 17.4 MB / 23.4 mJ**，depth network 为约 **349.9 ms / 2.86 Hz**。消融结果显示，如果直接使用 constant range，三个项目的 3D MAE 分别恶化到约 721 / 235 / 1890 mm；去掉 ground-contact cue 或 keypoint-depth cue 也会显著恶化，说明 metric range 估计来自多种物理/视觉线索的互补，而不是仅靠单目 depth。

## 优点

- 将 biomechanical target 从“3D joints”推进到直接具有运动学/力学解释价值的 **metric whole-body CoM**，评价目标更接近实际 coaching 与运动分析。
- 不是简单端到端黑盒回归，而是把 pose、depth、anthropometry、ground contact 与 ballistic prior 显式组合，模块边界清楚、便于消融和迁移。
- 给出了真实 iPhone 15 的 latency、FPS、能耗与 Core ML 部署，实现层面的可复现性很强。
- 在 figure skating 这类快速、大位移、高 range 的体育运动上进行评测，对户外高速体育有比普通步行 benchmark 更直接的参考价值。

## 局限

- AthletePose3D 只有 8 名运动员，且 reference 也属于 markerless 3D motion pipeline，样本人群与运动项目仍不足以证明广泛 biomechanics generalization。
- 方法依赖一次性的 camera/scene calibration，以及 stature-scaled、sex-specific population anthropometric proportions；个体体型偏离群体均值时可能引入系统误差。
- 作者明确指出 absolute localization 的主要瓶颈仍是 camera-to-athlete range，尤其在 sustained translation、long-range 和 airborne motion 下更加困难；figure skating 的 Z/depth MAE 达 672 mm，说明 metric depth 仍是主要误差源。
- **推断：**当前方案的相机基本是场边/手机式单目视角，而不是跟随运动员的 moving camera。若直接迁移到滑雪跟拍，ground-plane、ballistic range 和 camera-to-athlete geometry 会与 camera motion 耦合，需要把 camera trajectory 本身加入状态估计，而不能直接复用静态场景假设。

## 个人评价

这篇对体育研究最重要的价值不是“又一个 pose estimator”，而是提供了一个非常具体的中间生物力学量：CoM。相比只报告 MPJPE、PA-MPJPE 或 joint angle，把 metric CoM trajectory 独立作为 evaluation target 更容易连接 balance、turn transition、take-off/landing 和技术动作解释。

论文同时暴露了一个对滑雪很关键的事实：纵向/垂直 CoM 可以相对准确，但 camera-to-athlete depth 仍可能成为绝对 3D 最大误差源。因此如果现有双 360 / moving-camera pipeline 想做 world-space skiing biomechanics，必须把 range/scale error 与 pose error 分开报告，而不能把全部误差压缩到单一 MPJPE。

## 与我的研究关联

**推断：**可以把 MuyBridge 作为 `pose-only → pose + depth → anthropometry/contact-aware metric fusion → multi-view / moving-camera metric fusion` 的 baseline。对于滑雪，可以尝试从已有 3D skeleton 计算 segmental CoM，并使用双 360 三角测量、RTK/IMU、snow/ground plane 或 boot contact 替代/补充单目 range cue，再比较 CoM error、root trajectory、W-MPJPE 与 camera scale drift 是否同步改善。

另一个可复现方向是把 CoM 作为 world-HMR 的辅助约束：例如要求短时间内 CoM velocity/acceleration 与人体关节运动、foot contact、IMU acceleration 相互一致。**推断：**如果 CoM constraint 能反向稳定 camera scale 或 root translation，它可能成为 camera-human mutual refinement 中比单个 limb keypoint 更稳健的低频物理信号。

建议重点阅读 metric fusion / segmental CoM 部分，以及 range-error、figure-skating、ground-contact/keypoint-depth ablation 和 mobile deployment 章节。

## 后续阅读

- AthletePose3D：理解运动场景、参考 3D motion 和评测协议。
- OpenCap Monocular：比较单目视频到 kinematics/kinetics 的完整 biomechanics pipeline。
- BadmintonGRF / GRIP：比较 CoM、GRF、pressure 和 contact 等不同物理信号的可观测性。
- 后续实验：在滑雪数据上分开报告 vertical CoM、3D CoM、camera-to-athlete range、root trajectory 与 camera scale drift，并做 `pose-only → depth → contact → IMU/multi-view` 递进消融。
