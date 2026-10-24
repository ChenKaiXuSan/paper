---
title: "World Motion Models: Flexible Sequence Modeling of SE(3) Trajectories"
authors: "Jiahui Lei, Qianqian Wang, Trevor Darrell, Angjoo Kanazawa"
venue: "NeurIPS 2026 (Spotlight); arXiv:2610.01742"
year: 2026
reading_date: 2026-10-25
status: skimmed
tags:
  - global-human-motion
  - se3-trajectories
  - world-model
  - motion-representation
  - generative-motion
  - camera-human
---

# World Motion Models: Flexible Sequence Modeling of SE(3) Trajectories

## 基本信息

- **作者：** Jiahui Lei, Qianqian Wang, Trevor Darrell, Angjoo Kanazawa
- **会议/期刊：** NeurIPS 2026 (Spotlight)，arXiv:2610.01742；2026-10-01 首次提交
- **年份：** 2026
- **阅读日期：** 2026-10-25
- **阅读状态：** `skimmed`
- **标签：** `global-human-motion`, `se3-trajectories`, `world-model`, `camera-human`, `motion-representation`
- **论文：** https://arxiv.org/abs/2610.01742
- **DOI：** https://doi.org/10.48550/arXiv.2610.01742
- **代码：** https://github.com/JiahuiLei/WorldMotionModels （官方仓库存在，README 说明代码尚待发布）
- **数据集：** 暂无（论文评测采用已有任务数据集，不存在已核验的统一独立发布集）
- **项目主页：** https://jiahuilei.com/projects/wmm/
- **论文状态：** arXiv 公开；论文和作者项目主页均标注 NeurIPS 2026 Spotlight
- **价值类型：** Method Module / Related Work
- **阅读优先级：** A（世界坐标轨迹表征方面优先，非直接 HMR reconstruction baseline）

## 一句话总结

用统一的 SE(3) 物体、人体关节、相机及机器人轨迹 token 表征 4D 世界，以逐 token 独立噪声的 flow-matching 生成模型统一预测、补全、运动重定向和控制。

## 研究问题与动机

移动相机与人体、物体的时空状态通常分配给独立模型，缺少可共同建模多实体、不同时间跨度和任意条件观测的运动先验。本文假设动态世界能够用稀疏刚体运动基元组成：每个实体以随时间变化的 6-DoF SE(3) 位姿表示。人体关节、相机和物体因而可以作为同一表示空间里的不同轨迹。研究重点是**结构化运动生成与补全**，而不是从原始 RGB 图像估计三维人或相机。

## 核心方法

1. **Where / when / what 的实体—时间 token 表：** 各实体的 SE(3) pose 与身份特征组成二维 `M × N` token 网格，M 为轨迹实体数，N 为时间步；非因果上下文还可放到虚拟时刻。
2. **Flexible sequence modeling：** 训练阶段独立扰动各 token 的噪声水平，结合 flow matching 学习恢复干净姿态；推理时将已观测 token 固定，以不同的观测掩码执行未来预测、任意缺失段补全、条件生成或动作预测。
3. **Transformer 与因子化注意力：** 使用沿时间和实体方向分解的 attention、registers 和针对噪声水平的条件化模块，支撑成百上千的时空 token。
4. **六类应用：** Human–humanoid retargeting、TACO hand–object、机器人 motion planning、LangTable policy/MPC、OMOMO 人—物交互、DROID 与 EPIC-KITCHENS 的稠密三维场景轨迹预测。

## 数据集与评价指标

- **任务/数据集：** humanoid retargeting（GMR、PHUMA 对照）、TACO、LangTable、OMOMO、DROID、EPIC-KITCHENS；六类应用的输入都是已组织的轨迹或场景状态，稠密未来预测可以由首帧 RGB-D 生成查询轨迹。
- **样本量：** 官网及摘要未给出六种任务可相加的统一独立测试样本数；不以任务数量替代样本量。
- **输入/输出：** 输入为部分实体—时间 SE(3) token、目标/上下文；输出为缺失或未来实体的位姿轨迹，部分应用解码为机器人动作。
- **指标：** retargeting 推理 FPS、物理模拟中 tracking success rate；3D 轨迹预测 MSE、MAE、endpoint error；控制任务 success rate 等。不同任务指标不可直接比较。

## 主要结果

- **Humanoid retargeting：** 官方作者项目列出 87.0 FPS，对照 GMR 35.0 FPS、PHUMA 2.3 FPS；在冻结的低层物理控制器 rollout 中，tracking success rate 0.87，对照分别 0.79 / 0.83。
- **DROID 3D future trajectory：** MSE（官方表以 `×100` 报告）0.151，优于 TraceGen 0.206；endpoint error 0.200 对 0.285。
- **EPIC-KITCHENS：** 同评价表的 MSE（`×100`）0.322，优于 TraceGen 0.445；endpoint error 0.556 对 0.791。
- 这些数字是各自应用条件下的实验结果，不能当作 camera ATE、人体 MPJPE 或真实世界的重建误差。

## 优点

- 让相机、人体关节和场景元素的运动具备统一 SE(3) 表示，适合设计跨实体的时序一致性。
- 任意条件 mask 支持不同视角缺帧、观测稀疏和未来轨迹预测，无需为每种任务独立设计网络。
- 官方提供多种跨领域评价，其中稠密场景轨迹任务与几何运动研究有直接接点。

## 局限

- **关键边界：** 不是从多视角 RGB 或 fisheye/ERP 重建 camera/HMR 的 estimator；也未提供 human-to-camera bundle adjustment。
- 世界轨迹本身需由外部传感器或重建前端供给，推断的轨迹先验难以自动校准相机尺度、坐标基准或成像几何。
- Robotics / interaction 为主要验证领域；雪地、高动态跟拍、临床病理步态、未知相机内参和长时间漂移均缺少直接验证。
- 截至阅读日，官方 GitHub 仓库标明“code release soon”，尚不能直接复现完整模型。

## 个人评价

本论文最适合作为 **Method Module** 的设计参考而非替代 WHAM、PACE、VGGT 等视频重建 baseline。其统一实体—时间 token 结构比直接对姿态序列做单实体 Transformer 更符合 camera/human/scene 的耦合问题。但它尚未验证生成轨迹是否能降低真实相机误差，不能把跨机器人速度优势转换为 HMR 优势。

## 与我的研究关联

**推断：** 对双 360° 跟拍，可为两个物理相机的 SE(3) 轨迹、人体 root 及关节轨迹、雪板和场景接触状态分别建立实体 token，利用可观测 mask 处理视野遮挡与暂时跟踪丢失，再以几何重投影和人体先验迭代修正估计。与单独相机轨迹和人体运动的时序 Transformer 比较，检验 camera ATE/RPE、scale drift、W-MPJPE、缺失段 trajectory error、jitter。方法模块应被明确称为“推断性扩展”，并优先保持相机几何显式。

## 后续阅读

- 阅读论文方法中 per-token noise / flow matching、entity-time attention 与任意条件 masking 实现。
- **推断：** 先对 `known camera + missing human`、`known human + missing camera` 的 SE(3) 轨迹补全做合成消融，再接入真实双 360 视频。
- 与仓库既有 PACE、SynCHMR、Human3R、HTD-Refine、Anybody moving-camera 类方法对比；记录是否真正改善 camera pose，而不是只改善生成运动观感。
- 官方代码发布后再复核配置、训练集分割及可复现参数。
