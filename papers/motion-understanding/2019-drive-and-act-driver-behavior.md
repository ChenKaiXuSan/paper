---
title: "Drive&Act: A Multi-Modal Dataset for Fine-Grained Driver Behavior Recognition in Autonomous Vehicles"
authors: "Manuel Martin, Alina Roitberg, Monica Haurilet, Matthias Horne, Simon Reiß, Michael Voit, Rainer Stiefelhagen"
venue: "IEEE/CVF International Conference on Computer Vision (ICCV) 2019, pp. 2801–2810"
year: 2019
reading_date: 2026-11-04
status: skimmed
tags:
  - motion-understanding
  - driver-monitoring
  - fine-grained-action
  - multi-modal
  - multi-view
  - 3d-skeleton
  - head-pose
---

# Drive&Act: A Multi-Modal Dataset for Fine-Grained Driver Behavior Recognition in Autonomous Vehicles

## 基本信息

- **作者：** Manuel Martin, Alina Roitberg, Monica Haurilet, Matthias Horne, Simon Reiß, Michael Voit, Rainer Stiefelhagen
- **会议/期刊：** IEEE/CVF International Conference on Computer Vision (ICCV), October 2019, pp. 2801–2810（正式会议论文；重要经典补读）
- **年份：** 2019；官方项目论文页日期：2019-10
- **阅读日期：** 2026-11-04
- **阅读状态：** skimmed
- **标签：** motion-understanding, driver-monitoring, fine-grained-action, multi-modal, multi-view, 3d-skeleton, head-pose
- **论文：** https://www.driveandact.com/publication/2019_iccv_drive_and_act/2019_iccv_drive_and_act.pdf
- **代码：** 暂无（原论文提到评价脚本；当前未核验独立官方代码仓库）
- **数据集：** https://www.driveandact.com/
- **项目主页：** https://driveandact.com/publication/2019_iccv_drive_and_act/
- **价值类型：** Dataset / Baseline / Related Work
- **阅读优先级：** 中高（经典多模态驾驶员行为数据集；对 Drive-CHM3D 的跨条件评价有参考价值）

## 一句话总结

Drive&Act 建立驾驶舱内**六视角、RGB/NIR/Depth、3D 上半身骨架和 head pose** 的分层标注数据集，证明相机视角/模态变化会大幅影响细粒度驾驶员动作识别，而多视角模态融合可提高测试准确率。

## 研究问题与动机

驾驶员监测不能只识别粗粒度“注意/分心”，还需捕捉正在接触哪个物体、哪种操作和位置。传统驾驶舱视频数据缺少多视角、低照度模态、结构化三维人体与复杂动作标注，使得模型容易依赖固定传感器位置而不能泛化到未知车型或视角。本文把数据收集和固定 benchmark 协议作为主要贡献。

## 核心方法

1. 在受控驾驶模拟环境安排人工/自动驾驶与接管场景，给受试者下达多种操作任务。
2. 采集 5 台近红外相机 + 1 台 Kinect v2（RGB/IR/depth），共 **6 个物理视角**；其中 Kinect 一处集成 3 个模态，故官方网站的“5 views”图示是五台 NIR 位置，而论文按六视角计数。全局时间戳和标定支持跨传感器比较。
3. 从 3 个前向视角的 OpenPose 2D joints 三角化 13 个 3D 上半身关节，对缺失点插值；用多视角 OpenFace 获取 head rotation（挑选最正面的可用预测并转至 world frame）。这不属于独立的 marker-based GT。
4. 标注三层：12 类长期 task、34 类 semantic activities，以及 action/object/location 的原子语义组合（整体共 83 类）；按受试者 disjoint 划分 train/val/test，建立 pose-based、多流与 I3D 等 baseline，并测试跨视角和跨模态迁移。

## 数据集与评价指标

- **人数：** 15 人（11 男、4 女），每人录制 2 次共 30 个 session；官网公开入口标识 **29 条长序列**（录制 session 与公布序列数需区分）。
- **数据量：** 约 12 小时、超过 **960 万帧（计入多视角/模态）**。动作片段按不超过 3 秒切分评估。
- **划分：** 每 split 以人划分 10 名训练、2 名验证、3 名测试，防止同一人出现在两边；三组不同划分。
- **输入：** RGB、NIR、Depth、3D upper-body pose、head pose、interior features；**输出：** 驾驶员 task / fine-grained activity / action-object-location 标签。
- **指标：** 分类 top-1 accuracy（%），按 held-out driver 和 held-out view 评价；多模态使用 I3D softmax score averaging。
- **来源：** 官方项目、作者原始 ICCV 2019 PDF Sections 3–6、Tables 4–5、Figure 6。

## 主要结果

- 在 fine-grained activity 任务中，I3D 的 Kinect IR 单视角测试准确率 **64.98%**；五个 NIR 视角融合 **67.17%**；Kinect 三模态融合 **68.51%**；所有视角/模态 score averaging **69.03%**（Table 5）。
- 单独 face-view NIR 测试仅 **42.98%**，体现只看脸不能充分识别涉及物体的动作。
- 跨模态/跨视角存在明显 generalization gap：Kinect IR → right-top NIR 仅 **27.49%**，Kinect Color → Kinect IR 仅 **24.74%**（Figure 6）。
- 粗粒度 task classification 最好测试准确率 **35.45%**（pose-based Three-Stream），I3D 为 31.80%（Table 4）；细粒度与场景级标签不能直接混用同一性能数字。

## 优点

- 将多视角、RGB/NIR/Depth 与 3D body/head pose 放在同一驾驶舱任务中，提供明确的跨 sensor 和跨 view 评价。
- 受试者分离和细粒度标签层级对驾驶行为研究有长期参考价值。
- 原始论文对 camera/view domain shift 的实际性能差距给出直接量化，适合反思“多视角融合必然泛化好”的假设。

## 局限

- **年代和规模：** 2019 年模拟驾驶场景，只有 15 人；场景内二次任务是提示驱动，无法覆盖真实道路复杂分布。
- 3D 骨架来自多视角 OpenPose triangulation，head pose 来自 OpenFace 视角选择，而非独立高精度运动捕捉真值；快速极端转头的 head pose 可能不稳定。
- 主要是动作分类与短片段实验，没有为视觉功能受限下的 yaw/pitch 角度准确度提供独立 GT；不能作为 Drive-CHM3D head pose accuracy 的直接 benchmark。
- 官网标注 29 条公开长序列，论文描述 30 次采集；下载版本和标签可用性应复查。不同摄像头输入不是同一视场，简单 softmax 融合也无法解决几何配准。

## 个人评价

这里最有用的不是 2019 年 I3D 的绝对精度，而是 **跨模态/跨视角/domain shift 的实验协议**。它能提醒我们，在视觉功能条件、day/night、前后/侧视角不同的驾驶研究中，单一 overall accuracy 可能掩盖灾难性的 subgroup degradation。

## 与我的研究关联

**推断：**对于 Drive-CHM3D 模拟青光眼驾驶（多相机、day/night、不同视野条件），可借鉴 person-disjoint evaluation、camera-view ablation、跨光照迁移与 fusion score normalization。重点还应按 yaw/pitch axis、head-rotation direction、遮挡和条件分层评估，并做对 OpenFace/单摄像机角度系统误差的独立校准。可将同一多视角不同摄像头的 confidence 用于可靠性加权；但原论文动作识别标签不能直接替代你的 8 方位 head-angle annotations。

**建议阅读/复现：** Sec. 3.2 的 head-pose multi-view selection、Sec. 3.3 的受试者划分、Sec. 6.2–6.3 和 Table 5/Fig. 6 的 view/modality transfer；先以你的 88 条已采集驾驶序列执行 leave-one-condition 或 leave-one-view 对比。

## 后续阅读

- 对比 NTU RGB+D 与驾驶舱 13-joint skeleton 的 joint mapping 与异常/缺失关节处理。
- 在已有 Drive-CHM3D 三相机视频上补做 view × condition × yaw/pitch 误差矩阵。
- **待核验：** 官方网站的 29 条公开序列和论文中的 30 次录制间的数据清洗/缺失差异；下载后是否有精确可用 head-pose ground truth。
