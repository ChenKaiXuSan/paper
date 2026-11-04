---
title: "EMDB: The Electromagnetic Database of Global 3D Human Pose and Shape in the Wild"
authors: "Manuel Kaufmann; Jie Song; Chen Guo; Kaiyue Shen; Tianjian Jiang; Chengcheng Tang; Juan José Zárate; Otmar Hilliges"
venue: "ICCV 2023; arXiv:2308.16894"
year: 2023
reading_date: 2026-11-05
status: skimmed
tags:
  - global-human-motion
  - moving-camera
  - benchmark
  - camera-trajectory
  - world-coordinate
  - smpl
  - wearable-sensors
  - dataset
---

# EMDB: The Electromagnetic Database of Global 3D Human Pose and Shape in the Wild

## 基本信息

- **作者：** Manuel Kaufmann, Jie Song, Chen Guo, Kaiyue Shen, Tianjian Jiang, Chengcheng Tang, Juan José Zárate, Otmar Hilliges
- **会议/期刊：** IEEE/CVF International Conference on Computer Vision (ICCV 2023)，正式会议论文，2023 年 10 月
- **arXiv 首次提交：** 2023-08-31
- **DOI：** https://doi.org/10.1109/ICCV51070.2023.01345
- **年份：** 2023
- **阅读日期：** 2026-11-05
- **阅读状态：** skimmed
- **价值类型：** Dataset / Baseline / Related Work
- **阅读优先级：** 高（重要经典基准补读，非近期新论文）
- **论文：** https://arxiv.org/abs/2308.16894
- **正式出版：** https://openaccess.thecvf.com/content/ICCV2023/html/Kaufmann_EMDB_The_Electromagnetic_Database_of_Global_3D_Human_Pose_and_ICCV_2023_paper.html
- **代码：** https://github.com/eth-ait/emdb
- **数据集：** https://eth-ait.github.io/emdb/ （申请批准后下载）
- **项目主页：** https://eth-ait.github.io/emdb/

## 一句话总结

EMDB 用可移动的电磁传感器与手持手机、SMPL 拟合及图像优化，建立同时含全局人体根轨迹和相机轨迹的真实野外基准，使 moving-camera world-HMR 不再只靠局部姿态和视觉平滑度评价。

## 四段式推荐摘要

移动相机下的三维人体方法需要同时恢复人体与相机的全局运动，但传统 3DPW 等真实数据缺少充分可靠的全局轨迹监督。EMDB 通过身体佩戴的无线电磁传感器、手机视频与多阶段 SMPL 拟合，构建独立于纯 RGB 模型的野外评估基准。

数据包含 10 名受试者、81 段序列、104,963 帧（30 fps，约 58.3 分钟）。EMP 先拟合电磁测量，再融合 2D 关键点、深度及手机相机位姿，最后用神经隐式人体模型做像素级细化。21 段多视角验证显示约 2.3 cm 位置误差、10.6° 角误差；EMDB-2 共 25 段、43,120 帧，GLAMR 的 G-MPJPE 达 3,193 mm。

**推断：**它是双移动 360° 相机的独立 GT 评测参照。优先阅读 Section 7.2 与官方 evaluation_engine，分别复现 camera ATE/RPE、global root trajectory、pelvis-aligned MPJPE、MPJAE、jitter，而非用平滑后曲线代替真实世界精度。

局限是仅 10 人、单人物，手机相机轨迹精度受 Apple ARKit 上限约束；脚部没有佩戴传感器，且所提供的原生视频不是 ERP/360°。**推断：**不能直接代表高速滑雪或两台自由移动球面相机的分布。

## 研究问题与动机

- 许多 HMR 基准主要提供 camera-relative pose；对相机运动与人体根运动的解耦及 metric/world 轨迹无法直接给出严格验证。
- 仅使用 IMU 容易积累位置漂移；固定多相机 mocap 不能覆盖真实野外移动镜头。
- EMDB 的定位是**评测数据集与构建标注的 EMP 方法**，不是仅凭单目 RGB 输入便恢复全局人体的推理网络。

## 核心方法

1. **佩戴传感器与形体校准：**个体多视角扫描得到 SMPL 体形和 sensor-to-body offsets；用至多 12 个 wireless electromagnetic 6-DoF sensors 记录受试者运动。
2. **Stage 1 / EM local：**在电磁传感器本地坐标下优化 SMPL 姿态，并融合已校准体形与传感器偏移。
3. **Stage 2 / World alignment：**采用手持 iPhone 的相机轨迹定义 global frame，联合优化电磁测量、2D keypoints、depth 与相机位姿，把 SMPL root 恢复到世界坐标。
4. **Stage 3 / Pixel refinement：**把人体建成 deformable implicit SDF，背景建成 radiance field，利用 RGB pixel-level loss 修正 mesh-to-image 对齐和时序平滑。
5. **公开评估代码：**官方仓库的 evaluate.py、evaluation_engine.py、evaluation_loaders.py、evaluation_metrics.py 提供基线导入和分序列统计，包含 MPJPE、MPJAE、MVE、jitter 及 PA 版本。

## 数据集与评价指标

- **总数据：**10 人（5 女/5 男）、81 条序列、104,963 帧，30 fps / 约 58.3 min；约 85% 在 studio 之外拍摄。
- **EMDB-1：**17 条、24,117 帧，更偏 camera-relative / pose 评估；8 个单目 RGB HMR baseline，采用 MPJPE、MPJPE-PA、MVE、MPJAE 和 jitter。
- **EMDB-2：**25 条、43,120 帧，强调 world/global trajectories；GLAMR 采用 10 s 窗口的起始帧对齐协议。**不能把 81 条总集直接算作 EMDB-2 测试数。**
- **标注来源：**EM / iPhone AR tracking + SMPL + RGB alignment，不是每一帧均由户外多相机 marker-based 系统独立测得 GT；21 条独立多视角参考序列用于校验标注质量。
- **单位与协议：**MPJPE/MVE/G-MPJPE 为 mm，MPJAE 为角度，世界轨迹误差应写明是否起始对齐/全局对齐/尺度对齐；camera ATE/RPE 需另按具体算法输出定义。

## 主要结果

- EMP 在 21 段 MVS 评估序列中达到约 **2.3 cm 位置误差、10.6° 关节角误差**；论文另报告 global SMPL root trajectory 对室内 MVS 的估计误差约 **5.1 cm**。
- EMDB-1 的最佳所测 baseline HybrIK 在论文对应协议上的 **MPJPE-PA 为 65.6 mm**、MPJPE 为 **103.0 mm**、MPJAE 为 **24.5°**，说明局部 3D pose 与角度估计依旧不可靠。
- EMDB-2 中，原论文测试的 GLAMR **G-MPJPE 为 3,193 mm、G-MVE 为 3,203 mm**（每个 10 s 窗口首帧对齐）；展示 global trajectory drift 的困难。不能将其与后来 TRAM/WHAM 的不同评估协议直接混为同一个 leaderboard。
- 官方数据需申请，GitHub 公开可修改的评价脚本；公开 code/data 不等于免审批直接下载。

## 优点

- **同时提供 camera 和 human global trajectories**，解决 camera-only、local-pose-only 评估无法确定全局误差归因的问题。
- 有可查询的真实场景、困难姿态和独立多视角验证子集；方法与 benchmark 数据源公开透明。
- 官方评估代码利于建立可复现的 error decomposition：camera / root / body pose / angular / jitter。

## 局限

- 10 人样本、无多人序列；单个 handheld iPhone，不含双移动 360° 或高速雪地场景。
- 脚部未佩戴 EM sensor，室内金属地面可能干扰测量；足部 contact、滑移及地面反力不是 EMDB 的强监督项。
- 作者明确指出 **camera trajectory GT 质量受 Apple AR toolkit 精度限制**，因此 EMDB 的 camera poses 也不是绝对零误差。
- 数据获取需审批，且 EMDB-1/2 划分及对齐协议不同，误用易导致不公平比较。

## 个人评价

这篇是应该单独归档的**独立世界坐标评测基准**，与 TRAM、WHAM、PACE 等模型论文互补。论文最重要的启发是：较好的 PA-MPJPE、per-joint smoothness 并不能证明世界坐标轨迹正确；需要兼顾 camera pose、SMPL root、joint orientation 与长时间漂移。

## 与我的研究关联

- **推断：**在目前 dual 360° skiing 与 future 12-camera capture 中，区分 ①物理 camera world trajectory ②人体 root global trajectory ③body-relative joints ④joint velocity/jerk；各层独立获得 reference / uncertainty。
- 以 EMDB-2 为公开世界人体评测入口，对 TRAM/WHAM/GVHMR、PACE/SynCHMR/BodySLAM++ 等做统一协议检查，特别注意 GT-scale / Sim(3) 与起始对齐的区别。
- EMDB 使用手机 AR tracking 提供外部 camera reference，说明人体与相机评价的独立传感器很重要；双 360° 场景可用已标定 static array/OptiTrack/RTK+IMU（若可行）作为 independent reference（**推断**）。
- 优先复现 Section 7.1–7.2 与官方 evaluation_metrics.py / evaluation_engine.py，再增加 camera ATE/RPE、MPJVE、MPJAE、root RTE、contact/foot sliding。

## 后续阅读

- [TRAM](2024-tram-global-trajectory-and-motion.md)（同仓库）与 WHAM/GVHMR/GLAMR 的 EMDB-2 evaluation。
- EMDB 原论文 Section 3–6 EMP 标注流水线、Section 7 Dataset/Baselines，官方 [benchmark 代码](https://github.com/eth-ait/emdb) 与 [数据申请](https://eth-ait.github.io/emdb/)。
- **推断：**构造可独立对照 camera 误差与人体误差的实验，报告 human-to-camera correction 是否同时减少 ATE/RPE 和 G-MPJPE。
