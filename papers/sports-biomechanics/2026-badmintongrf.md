---
title: "BadmintonGRF: A Multimodal Dataset and Benchmark for Markerless Ground Reaction Force Estimation in Badminton"
authors: "Kuoye Niu, Jianwei Li, Shengze Cai, Yong Ma, Mengyao Jia, Lishun Shen, Zhenheng Zhang, Yuxin Peng, Xian Song"
venue: "ACM Multimedia 2026 / arXiv:2605.01876"
year: 2026
reading_date: 2026-08-29
status: skimmed
tags:
  - sports-biomechanics
  - ground-reaction-force
  - badminton
  - multiview
  - dataset
  - pose
  - fatigue
---

# BadmintonGRF: A Multimodal Dataset and Benchmark for Markerless Ground Reaction Force Estimation in Badminton

## 基本信息

- **作者：** Kuoye Niu, Jianwei Li, Shengze Cai, Yong Ma, Mengyao Jia, Lishun Shen, Zhenheng Zhang, Yuxin Peng, Xian Song
- **会议/期刊：** ACM Multimedia 2026；arXiv:2605.01876
- **年份：** 2026
- **阅读日期：** 2026-08-29
- **阅读状态：** `skimmed`
- **标签：** `sports-biomechanics`, `ground-reaction-force`, `badminton`, `multiview`, `dataset`, `pose`, `fatigue`
- **论文：** https://arxiv.org/abs/2605.01876
- **代码：** https://github.com/KenyaNiu/BadmintonGRF
- **数据集：** https://doi.org/10.5281/zenodo.19277566
- **项目主页：** https://KenyaNiu.github.io/BadmintonGRF/

## 一句话总结

BadmintonGRF 建立了一个面向高速、非周期体育落地动作的 markerless GRF benchmark，将约 120 FPS 的 8 视角 RGB、Kistler force plates、Vicon mocap 和疲劳阶段协议结合起来，并提供从 2D pose 估计 body-weight-normalized vertical GRF 的 LOSO 基准。

## 研究问题与动机

体育动作中的落地冲击、制动和方向变化与伤害风险、训练负荷和技术表现密切相关，但 laboratory force plate 与 mocap 难以扩展到普通训练场景。现有视频人体姿态数据集通常没有同步 GRF，而公开 kinetics 数据又很少覆盖羽毛球这类高速、非周期、impact-centric 动作。BadmintonGRF 因此试图建立一个可复现的视频到 GRF 研究基准，同时把跨设备同步、质量控制和跨受试者评价纳入公开 protocol。

## 核心方法

数据采集采用 **8 台固定 RGB 相机（约 120 FPS）**、**4 块 Kistler 6-axis force plates（1000/1200 Hz）** 和 **8-camera Vicon**。由于 RGB 与 force/Vicon 没有硬件 genlock，作者使用人工确认事件 + per-camera temporal offset + 自动 QA 完成对齐，并保留 offset uncertainty metadata。

公开 Tier 1 benchmark 不直接使用 raw RGB，而是从视频提取 COCO-17 2D keypoints 与 confidence，并构造每帧 **119-D** 特征，包括位置、有限差分 velocity/acceleration 和置信度。主要任务是在已知 impact alignment 的约 1 秒窗口内预测与 video rate 对齐的 BW-normalized vertical GRF $F_z$。作者比较 PatchTST、ST-GCN+Transformer、TCN+BiGRU、TSMixer、sequence Transformer 等 10 个 baseline，并使用 subject-disjoint LOSO。

## 数据集与评价指标

采集池包含 **17 名 instrumented subjects**；冻结的公开 benchmark 使用其中 **10 名受试者**进行 LOSO。Tier 1 共包含 **17,425** 个 view-specific impact-segment archives，经质量门控保留 **12,867** 个实例，对多视角去重后对应 **1,732 个 unique physical impacts**，来自 **156 个 instrumented trials**。

协议包含 rally、不同 footwork stage 以及 fatigue_stage 条件。主要评价指标为 $r^2$、RMSE (BW)、peak force error (BW) 和 peak timing error (frames)。LOSO 每一 fold 在训练受试者内部再划分 15% validation，正式表格不使用 test-time augmentation。

## 主要结果

LOSO single-view benchmark 中，**PatchTST** 获得最高 $r^2=0.403$ 和最低 RMSE **0.510 BW**，peak error 为 0.226 BW、peak timing error 为 1.07 frames。**ST-GCN+Transformer** 的 $r^2=0.394$、RMSE=0.514 BW，但获得更好的 peak force error **0.221 BW** 和 peak timing error **0.96 frames**。结果说明不同模型对 waveform fit、peak magnitude 和 peak timing 的优势并不一致，因此只报告一个回归误差不足以评价运动冲击负荷估计。

同步 QA 方面，公开补充材料记录了 **1,247** 个 `(trial, camera)` offset records，其中 **65 个（5.2%）**需要人工 reconciliation；这使 synchronization uncertainty 本身也成为数据质量的一部分，而不是隐藏的预处理步骤。

## 优点

- 相比周期步态，羽毛球落地与变向更接近高速体育中的非周期冲击事件，对滑雪等户外运动的 biomechanics 研究更有迁移价值。
- 同时具备高帧率多视角 RGB、force plate 和 Vicon，可支持 pose、GRF、cross-view 与同步误差的多层研究。
- LOSO、冻结 split、公开 baseline 与详细 QA 文档使 benchmark 可复现性较强。
- fatigue-stage metadata 允许研究运动疲劳对视觉 kinetics estimation 的 domain shift。

## 局限

- 公共 Tier 1 的主任务是 **2D pose → BW-normalized vertical $F_z$**，并假定 impact 已经对齐；它并不是从任意 raw video 直接发现 impact 并恢复完整 6-axis GRF。
- 公开 LOSO benchmark 只有 10 名受试者，原始采集池也只有 17 人；跨场馆、跨相机和跨运动项目泛化仍待验证。
- raw multi-view RGB 与 C3D 属于 Tier 2 controlled access，因此只使用公开 Tier 1 时无法完整复现 raw-video 端到端 pipeline。
- 固定室内相机与 force-plate 区域和移动户外相机仍存在明显 domain gap。

## 个人评价

这篇工作的最大价值是把体育视频从“估姿态/识别动作”推进到“估计外部载荷”，并且没有只使用随机 train/test split，而是以 LOSO 检查跨运动员泛化。其 peak magnitude + peak timing 的评价设计很适合高速运动，因为一个模型即使整体 RMSE 较低，也可能在最关键的冲击瞬间产生错误。同步 QA 的公开方式也值得借鉴：视频与高频传感器不同步时，offset uncertainty 应成为显式实验变量。

## 与我的研究关联

`推断`：对于滑雪，可以把该 benchmark 的设计迁移为 `multi-view/360 video → pose/world motion → contact/load proxy`。即使暂时没有雪地 force plate，也可先使用 IMU、足底压力或 boot/ski sensor 构造局部 kinetics ground truth，并沿用 subject-disjoint、fatigue/condition stratification、peak magnitude/timing 与 alignment uncertainty 的评价框架。

另一条可直接借鉴的路线是比较 single-view、multi-view late fusion、3D pose 与 world-coordinate motion 对 GRF/load prediction 的贡献，从而把 3D reconstruction 的价值从 MPJPE 延伸到 downstream biomechanics。

## 后续阅读

- 复现 PatchTST 与 ST-GCN+Transformer 的 LOSO protocol，比较时序 Transformer 与 skeleton graph representation。
- 检查 multi-view late fusion 在 $r^2$、RMSE、peak error 和 timing 四类指标上的 trade-off。
- 探索将 3D joint angles、root velocity、contact state、IMU/pressure 与 2D pose 联合用于高速体育 load estimation。
- 进一步寻找 skiing / alpine sports 中带 GRF、boot pressure、IMU 或 force-related ground truth 的公开数据集。
