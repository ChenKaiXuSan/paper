---
title: "EgoHumans: An Egocentric 3D Multi-Human Benchmark"
authors: "Rawal Khirodkar, Aayush Bansal, Lingni Ma, Richard Newcombe, Minh Vo, Kris Kitani"
venue: "ICCV 2023 (Oral), pp. 19807–19819"
year: 2023
reading_date: 2026-11-11
status: skimmed
tags:
  - global-human-motion
  - egocentric
  - multi-moving-camera
  - multiview-calibration
  - world-coordinate-pose
  - dataset
  - multi-human-tracking
---

# EgoHumans: An Egocentric 3D Multi-Human Benchmark

## 基本信息

- **作者：** Rawal Khirodkar, Aayush Bansal, Lingni Ma, Richard Newcombe, Minh Vo, Kris Kitani
- **会议/期刊：** ICCV 2023 (Oral), pp. 19807–19819；arXiv 首次提交 2023-05-25；正式会议论文，重要经典补读。
- **年份：** 2023
- **阅读日期：** 2026-11-11
- **阅读状态：** skimmed
- **标签：** global-human-motion, egocentric, multi-moving-camera, multiview-calibration, world-coordinate-pose, tracking
- **论文状态：** ICCV 2023 正式会议论文；2023 年经典数据集补读。
- **价值类型：** Dataset / Baseline / Method Module / Related Work
- **阅读优先级：** 高（可直接验证多移动相机世界人体轨迹与身份关联）
- **论文：** https://openaccess.thecvf.com/content/ICCV2023/papers/Khirodkar_Ego-Humans_An_Ego-Centric_3D_Multi-Human_Benchmark_ICCV_2023_paper.pdf
- **DOI：** https://doi.org/10.48550/arXiv.2305.16487（arXiv DOI；不冒充正式会议 DOI）
- **代码：** https://github.com/rawalkhirodkar/egohumans
- **数据集：** https://drive.google.com/drive/folders/1JD963urzuzV_R_6FOVOtlx8UupwUuknR?usp=sharing （作者项目页指向的官方文件夹；具体文件访问权限需要实际检查）
- **项目主页：** https://rawalkhirodkar.github.io/egohumans/
- **其他一手来源：** https://arxiv.org/abs/2305.16487

## 一句话总结

EgoHumans 将多个佩戴式移动 Aria 相机与固定 GoPro 网络时空对齐，构建具相机参数、世界坐标 3D joints、SMPL、身份跟踪的真实体育多人物 benchmark，并以 EgoFormer 验证世界坐标 3D 身份关联的价值。

## 研究问题与动机

现有 egocentric 数据集常局限于室内、单人、较慢的相机运动或有限遮挡；对自由体育动作的移动拍摄难以可靠提供长期多人物世界坐标真值。论文将数据采集、相机统一标定、鲁棒 3D 标注与多人物跟踪放在同一 benchmark 体系，而非只评价固定视角的姿态估计。这里的“世界坐标”是统一重建参考系，不应直接理解为地理坐标。

## 核心方法

1. **多设备采集与相机对齐（Section 3）：** 使用 Aria 眼镜 RGB + 双灰度相机、8–15 台固定 GoPro；同步多路视频。Aria 内参取设备标定，逐帧外参由 VIO 获取；使用 Procrustes alignment + SfM 将多个移动相机轨迹和静态相机置于共同世界坐标。
2. **多人物关联和 2D 标注：** 从 3D cylinder 人体区域初始化跨相机 identity，投影形成检测区域，再以 Faster R-CNN、HRNet-WholeBody 与人工修正标注 bbox、ID 和 2D 关键点。
3. **3D 真值流水线：** 由各相机投影矩阵与 2D joints 执行 RANSAC 多视角 triangulation，随后使用 limb-length、left–right symmetry、temporal smoothness、regularization 优化 3D joints；最后用 CLIFF 初始化，经三阶段 SMPL 拟合获得全局人体 mesh。
4. **EgoFormer（Section 4）：** 同时编码 RGB、左右灰度视图，估计人体 depth/root 与 2D pose/SMPL；利用相机轨迹转换到静态世界参考系并以 3D 位置进行 multi-object tracking。此处并不将人体误差反馈到 VIO 相机轨迹。

## 数据集与评价指标

- **采集规模：** 20 名参与者、7 条活动序列、6 个地点（室内 3、室外 3），覆盖球类、击剑、羽毛球等运动；各序列佩戴式相机 2–4 台，固定相机 8–15 台（各场景不相同）。
- **数据：** 125k ego RGB、250k ego grayscale、446k GoPro images；约 410k 可见 2D 人体实例。Ego RGB train 77,260 / test 47,740 图像，地点不重叠；短片约 30 秒、20 FPS。
- **输入/输出：** 同步多视角 RGB/灰度视频、校准/VIO → world-coordinate 3D human joints、SMPL、bbox、tracking IDs、camera poses；基线跟踪模型主要由 egocentric 图像输出逐帧关联结果。
- **对比与指标：** ByteTrack、OCSORT、DeepSORT、QDTrack 等；IDF1、HOTA、MOTA、identity switches；以独立 LiDAR 点云对重建 mesh 用双向 Chamfer 距离（cm）检查标注误差。

## 主要结果

- **跟踪（论文 Table 2 的同一评估协议）：** EgoFormer IDF1 **63.1**，ByteTrack **49.7**（提高 13.4 个百分点）；IDs 从 **2,696** 减至 **741**。在另一个所有方法 finetune 的 Table 4 中，EgoFormer **63.1**、ByteTrack **51.5**，不可混为同一比较。
- **标注质量（Table 3）：** 与独立 LiDAR 点云相比，使用 4、8、12 台 secondary cameras 时双向 Chamfer 距离分别约 **10.4 / 8.3 / 5.8 cm**。移除 temporal loss，12 台相机误差从 5.8 升至 **7.9 cm**。
- **视角消融（Table 5）：** EgoFormer RGB-only 的 IDF1 **58.1**；加入左右灰度后达到 **63.1**，同时 IDs 由 2,139 降至 741。不要将上述 IDF1 当作 3D MPJPE 或相机定位误差。

## 优点

- 真实体育、多人物、快速移动眼镜相机，且有 ego/exo 同步观测、世界相机姿态与人体参考标注，适合研究 camera motion 与人运动耦合。
- 将相机跨设备坐标对齐、跨视角 2D 关联、triangulation、SMPL-fitting 的来源清楚拆开，并使用 LiDAR 独立检查 3D 质量。
- 基线明确显示世界坐标 3D 身份关联和多视角信息能改善高动态相机下的跟踪。

## 局限

- 所谓 3D GT 为多摄 triangulation + temporal/anthropometric constraints + SMPL-fitting 的估计结果，不是所有关节的独立 marker-based mocap；LiDAR 检查 mesh 距离也不等于逐关节绝对 GT（**推断**）。
- 相机轨迹依赖 Aria VIO、SfM 与已知设备内参；并非未知相机模型下的纯 RGB、完全 calibration-free 估计。
- 只有 7 条完整活动序列和 20 位参与者，外部相机密度高；不包括双 ERP 360° 或雪场长期高速轨迹。EgoFormer 不会反向修正相机 VIO。
- 作者项目有代码与数据入口，但下载权限和实际可获得的具体标注范围仍需在复现实验前核查。

## 个人评价

**推断：**把这篇作为 moving-camera world-HMR 的数据与评价参考，比把它仅当作 tracker 更有用。特别应区分“标注通过固定多机位获得”和“推理阶段仅能使用移动相机图像”两种实验设置，防止评估时 oracle camera/GT 信息泄露。

## 与我的研究关联

**推断：** 对双 360° 滑雪世界轨迹基准，可模仿其“固定密集相机辅助构建 3D reference → 动态移动相机独立运行”的数据组织策略；使用 12 台 RX0 II 离线重建 3D pose 后，再将两台移动 360° 视频接入做 world HMR。必须记录物理相机外参、2D人工复核比例、世界 3D joints 误差来源、人物遮挡及追踪 ID。先做 camera-only SLAM、per-view pose、3D tracking，再测 human→camera feedback 是否真正降低 ATE/RPE，而非只提高 IDF1。

## 后续阅读

- **重点：** Section 3（calibration / 3D triangulation / SMPL fitting）、Section 4（world-frame 3D tracking）、Table 2/3/5（tracking、LiDAR 对照和多视角消融）。
- **复现建议（推断）：** 评估静态辅助相机数量 2/4/8/12 对 GT 重建质量的影响，增加相机内外参噪声、时间偏移、2D 遮挡、动态相机速度分层测试。
- **对比阅读：** [Human Mesh Reconstruction of Sports Players with Multiple Dynamic Cameras](../global-human-motion/2025-sports-mesh-multiple-dynamic-cameras.md)、[Everybody Tracking Every Body](../global-human-motion/2026-everybody-tracking-every-body.md)、[EMDB](../global-human-motion/2023-emdb-global-3d-human-pose-and-shape.md)。
