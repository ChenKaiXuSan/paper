---
title: "eSkiTB: A Synthetic Event-based Dataset for Tracking Skiers"
authors: "Krishna Vinod, Joseph Raj Vishal, Kaustav Chanda, Prithvi Jai Ramesh, Yezhou Yang, Bharatesh Chakravarthi"
venue: "WACV 2026 Workshops (Computer Vision for Winter Sports)"
year: 2026
reading_date: 2026-10-14
status: skimmed
tags:
  - sports-motion
  - skiing
  - event-camera
  - tracking
  - multimodal-learning
  - benchmark
---

# eSkiTB: A Synthetic Event-based Dataset for Tracking Skiers

## 基本信息

- **作者：** Krishna Vinod, Joseph Raj Vishal, Kaustav Chanda, Prithvi Jai Ramesh, Yezhou Yang, Bharatesh Chakravarthi
- **会议/期刊：** WACV 2026 Workshops / Computer Vision for Winter Sports，pp. 463–472（CVF 正式论文）
- **首次 arXiv 提交：** 2026-01-10
- **年份：** 2026
- **阅读日期：** 2026-10-14
- **阅读状态：** `skimmed`
- **标签：** `sports-motion`、`skiing`、`event-camera`、`tracking`、`benchmark`
- **论文：** https://openaccess.thecvf.com/content/WACV2026W/CV4WS/html/Vinod_eSkiTB_A_Synthetic_Event-based_Dataset_for_Tracking_Skiers_WACVW_2026_paper.html
- **arXiv：** https://arxiv.org/abs/2601.06647
- **DOI：** https://doi.org/10.48550/arXiv.2601.06647
- **代码：** https://github.com/eventbasedvision/eSkiTB
- **数据集：** https://www.dropbox.com/scl/fi/1fbsjthks57e3mucxrsz5/E-Ski_v2e.zip?rlkey=8hn6umsbo406eygoy7s81kg1d&st=0fntvbfr&dl=0 （作者 GitHub README 官方发布入口；下载内容未逐文件审核）
- **项目主页：** https://eventbasedvision.github.io/eSkiTB/
- **价值类型：** Dataset / Baseline / Related Work / Method Module
- **阅读优先级：** A（滑雪跟拍的 tracking robustness / domain-specific benchmark）

## 一句话总结

eSkiTB 将真实滑雪转播 RGB 直接转换成不含神经插帧的模拟事件流，为高速滑雪者跟踪构建 RGB–event 对照基准，并明确呈现 event motion cue 在视觉杂波中有益、但精调 RGB 仍可能更强的边界。

## 研究问题与动机

冬季运动中的运动员高速运动、尺度变化、低纹理雪面和电视台记分牌叠加，容易让 RGB tracker 漂移。事件相机只响应亮度变化，理论上能天然抑制固定屏幕叠加物，但缺少针对滑雪的受控数据与对照。作者强调 **iso-informational**：不通过神经插帧制造 RGB 原视频不存在的中间运动信息，以便更接近公平的 RGB vs event 模态比较。

## 核心方法

1. **构造：** 从现有 SkiTB 的 RGB 视频使用 v2e 进行模拟事件生成，contrast threshold `C_pos=C_neg=0.2`，不进行神经帧插值；生成 HDF5 事件 `(t,x,y,p)`。
2. **标注：** 保留原始每帧 skier bounding box，另外用三次样条将其插值为每 1 ms 一个 bbox，便于连续时间跟踪评价。**注意 1 ms 标签是插值，不是真实 1 kHz 独立人工 GT**。
3. **模型对照：** RGB 侧采用 STARK（generic / fine-tuned / ski-specific），事件侧采用 spiking Transformer SDTrack（pretrained / fine-tuned）。
4. **压力测试：** 高杂波子集与细分学科 AL / FS / JP 测试，分析 static overlays、low resolution、运动速度和 128×128 voxel-grid 限制。

## 数据集与评价指标

- 300 个序列，约 **235 min**，涵盖 alpine、freestyle、ski jumping，来自 98 个地理地点、10 类天气。
- 训练/验证/测试序列数 = **240/30/30**；原始/事件流保存分辨率 1280×720；单序列 275–3582 帧，平均 1176 帧。
- 输入为 RGB 图像或从这些图像模拟的 event streams；输出为逐时刻滑雪者 2D bounding box 和 confidence，**不是 2D/3D 关节点或相机标定参数**。
- 指标：Mean IoU、Precision@20px、Success@0.5，以及按高杂波条件拆分的结果；较密集时间戳标注由插值生成。
- 原始论文/数据： https://arxiv.org/abs/2601.06647 和官方仓库 README。

## 主要结果

| 方法 | 模态 | Mean IoU ↑ | Precision@20px ↑ | Success@0.5 ↑ |
| --- | --- | ---: | ---: | ---: |
| STARK generic | RGB | 0.512 | 0.567 | 0.568 |
| STARK fine-tuned | RGB | 0.795 | 0.847 | 0.904 |
| STARK ski-specific | RGB | **0.829** | **0.887** | **0.935** |
| SDTrack pretrained | Event | 0.312 | 0.354 | 0.418 |
| SDTrack fine-tuned | Event | 0.711 | 0.720 | 0.873 |

高杂波 **9 个**序列子集：SDTrack Mean IoU **0.685**，比 **generic** STARK 高约 0.200，**但低于 ski-specific STARK 的 0.833**。不同项目存在明显差异：ski jumping 的事件追踪 IoU 0.974，freestyle 仅 0.569，低于后者 RGB fine-tuned 的 0.728。这些数字直接表明不能将“事件模态整体胜过 RGB”作为结论。

## 优点

- 领域与滑雪高速跟拍高度一致，包含雪地低纹理、高速尺度变化及转播干扰。
- 明确要求无插帧的模态公平约束；提供 300 序列 / 3 学科切分与事件格式。
- 强调域内 fine-tuning 的重要性，避免只与较弱的 RGB generic baseline 比较。
- 提供细分 high-clutter / freestyle 失败分析，使 tracking 可以作为后续 HPE/HMR 上游质量门控的评价对象。

## 局限

- **已验证事实：**事件是 25–60 Hz RGB 原视频的模拟结果，未通过真实事件硬件采集；真实微秒级运动仍不可从源视频恢复。
- **已验证事实：**密集 1 ms bbox 是样条插值，不能作为真实 1000 Hz ground truth；tracker 内部还将高分辨率事件降到 128×128 voxel-grid。
- **已验证事实：**全数据集上 fine-tuned Event SDTrack 的 0.711 IoU 低于 ski-specific RGB STARK 的 0.829；“高杂波优势”是相对 generic RGB baseline。
- **推断：**其 broadcast 场景、单目标检测框任务、光流与 camera-motion 模式与双 360° 自主跟拍不完全等价，也没有可直接使用的 3D pose GT。

## 个人评价

这篇文章最重要的价值在可复现实验设置和**诚实的域内强 RGB 对照**，而非无条件证明 event sensing 优于 RGB。对于当前依赖普通 RGB/360 相机的项目，优先迁移的是 temporal contrast、flow segmentation 和 tracker confidence 特征，而不必直接采购事件相机。

## 与我的研究关联

**推断：**可在双 360° 滑雪的人体识别/裁剪前端加入 RGB temporal-difference、event-simulation 或光流对比支路，特别关注低纹理、转弯加速、远距离小人体和阴影/背景运动。评估 `RGB tracker → RGB+temporal contrast → +cross-view consistency → 3D pose fusion`，以 bbox IoU/recall/ID switches、view dropout、downstream MPJPE 和 camera geometry 误差分别报告；不能将更好 bbox IoU 当成更准确 3D pose 的直接证据。

## 后续阅读

- 优先看 Sec. 3.1–3.7 的数据制作、Sec. 4 的 Table 3 和 high-clutter / discipline analysis。
- 核实官方 GitHub 数据下载入口、HDF5、splits、bbox annotation 的实际可用程度。
- **推断：**对自己的滑雪视频模拟事件流，不做插帧先作控制实验；再与 RGB/flow-based tracker 在相同训练量、同一 subject/scene-disjoint split 下比较。
- 用真实相机几何/关节点独立验证上游跟踪收益是否能迁移到 world-coordinate 3D pose。
