---
title: "HumanMM: Global Human Motion Recovery from Multi-shot Videos"
authors: "Yuhong Zhang, Guanlin Wu, Ling-Hao Chen, Zhuokai Zhao, Jing Lin, Xiaoke Jiang, Jiamin Wu, Zhuoheng Li, Hao Frank Yang, Haoqian Wang, Lei Zhang"
venue: "CVPR 2025, pp. 1973–1983"
year: 2025
reading_date: 2026-11-14
status: skimmed
tags:
  - global-human-motion
  - moving-camera
  - multi-shot
  - world-coordinate
  - camera-geometry
  - temporal-alignment
  - human-motion
---

# HumanMM: Global Human Motion Recovery from Multi-shot Videos

## 基本信息

- **作者：** Yuhong Zhang, Guanlin Wu, Ling-Hao Chen, Zhuokai Zhao, Jing Lin, Xiaoke Jiang, Jiamin Wu, Zhuoheng Li, Hao Frank Yang, Haoqian Wang, Lei Zhang
- **会议/期刊：** CVPR 2025，pp. 1973–1983（正式会议论文）
- **首次提交：** 2025-03-10（arXiv）
- **年份：** 2025
- **阅读日期：** 2026-11-14
- **阅读状态：** `skimmed`
- **标签：** `global-human-motion`, `moving-camera`, `multi-shot`, `world-coordinate`, `camera-geometry`, `temporal-alignment`
- **论文：** https://arxiv.org/abs/2503.07597
- **CVPR 正式页面：** https://openaccess.thecvf.com/content/CVPR2025/html/Zhang_HumanMM_Global_Human_Motion_Recovery_from_Multi-shot_Videos_CVPR_2025_paper.html
- **DOI（arXiv）：** https://doi.org/10.48550/arXiv.2503.07597
- **代码：** https://github.com/zhangyuhong01/HumanMM-code （作者官方仓库；截至本次核验，仓库只见 README，尚无可复现训练/推理实现）
- **数据集：** https://huggingface.co/datasets/YuhongZhang/ms-Motion （作者发布的 ms-Motion，600 条）
- **项目主页：** https://zhangyuhong01.github.io/HumanMM/
- **价值类型：** Baseline / Method Module / Dataset / Related Work
- **阅读优先级：** 高；2025 年重要工作补读，不是当日新论文

## 一句话总结

将镜头切换检测、Masked LEAP-VO、GVHMR、跨镜头人体朝向/姿态对齐和时序轨迹积分连接起来，从切换视角的长视频中恢复连续的 world-coordinate 人体运动。

## 研究问题与动机

传统 world HMR 通常假设一个连续镜头；体育直播、表演和剪辑视频中的 shot transition 会造成 camera frame 突变、人物遮挡、root orientation 错位及 world trajectory 断裂。单镜头 SLAM+HMR 不能直接跨镜头复用全局坐标。HumanMM 关注的是**时间上交替出现的不同镜头**，不是同时同步的多移动相机融合。论文假定多镜头相机内参保持一致（原文第 1 节脚注）；这对变焦、不同设备和鱼眼/ERP 仍是约束。

## 核心方法

1. **Shot transition detector**：融合场景变化、人框跟踪、2D pose tracking；分镜后处理每个连续片段。
2. **相机与人体初始化**：用 Masked LEAP-VO（长期 point tracking 与人体动态遮挡排除）估计 camera R/T，用 GVHMR 恢复片段内 SMPL/world motion。
3. **跨镜头对齐**：用镜头边界的相机相对旋转与人体 view-relative orientation 对齐 world heading，并使用 ms-HMR 的时序编码修复边界姿态突变。
4. **Motion integrator**：双向 LSTM 的 global velocity/trajectory predictor 加 refiner、contact-aware foot sliding correction，平滑世界坐标轨迹。它主要修正人体输出，不等同于让人体 residual 显式回传优化每帧 camera trajectory。
5. **输入/输出**：RGB 多镜头视频（ViTPose 2D joints/bboxes、ViT feature）；输出 shot-aware 的相机参数与连续 SMPL 人体世界姿态/朝向/root translation。

## 数据集与评价指标

- **ms-Motion**：由公开 AIST++ 与 Human3.6M 的多机位录制合成不同镜头切换，含 ms-AIST、ms-H3.6M；600 条视频，30 fps，平均 23.7 s，覆盖 2/3/4 次镜头切换；论文报告总时长 237 min。
- **参考 GT**：原始数据集的 multi-view human/camera GT；另外以 EMDB-1（17 段，13.5 min）、EMDB-2（25 段，24.0 min）评价/消融移动相机估计。
- **指标**：PA-MPJPE、WA-MPJPE（mm）、Root Translation Error RTE（m）、Root Orientation Error ROE（度）、Jitter、Foot Sliding（cm）、camera ATE（m）、RPE translation/rotation，以及镜头切换检测 F1。
- **统计核验提醒**：论文第 5.1 节写 42.7K frames，但 600×23.7s×30fps ≈ 426.6K，与 237 分钟也应约 426.6K，存在约十倍的计数不一致；不将 42.7K 当作确定帧数。

## 主要结果

- **ms-AIST，2 次镜头切换/2-shot setting（原表称 2-Shot）**：HumanMM PA-MPJPE **36.82 mm**、WA-MPJPE **121.35 mm**、RTE **2.56 m**、ROE **69.23°**、Foot Sliding **2.66 cm**；GVHMR 分别为 **60.72 mm**、**231.36 mm**、**6.20 m**、**96.58°**、**7.65 cm**（Table 1）。
- **ms-H3.6M，相同设置**：HumanMM PA-MPJPE **40.52 mm**、WA-MPJPE **132.13 mm**，对照 GVHMR **64.63 / 254.30 mm**（Table 1）。
- **镜头边界检测**：结合 scene+bbox+pose，F1 **0.92**，scene-only **0.70**（Table 4）。
- **相机精度的反例**：EMDB-1 上 Masked LEAP-VO 的 RPE translation 从 DPVO 的 **1.85** 降至 **0.92**，但 camera ATE **0.51 m**，反而略差于 DPVO 的 **0.48 m**；因此不能简单宣称 camera 所有指标均提高（Table 5）。
- 随镜头数增加，ms-AIST 的 WA-MPJPE 从 121.35 增至 141.38 / 161.52 mm，说明切换累积仍有误差。

## 优点

- 将跨镜头 world consistency 作为独立问题，而非仅对每段单独重建。
- 公开具有 camera/human 参考的 ms-Motion 以及多种 global/temporal 指标，方便分离姿态、轨迹、朝向和接触效果。
- 对不同镜头数量和相机 ATE/RPE 给出评价，包含失败趋势。

## 局限

- ms-Motion 主要是现有受控多机位数据的**人工拼接**，真实直播转场中的设备内参、曝光、焦段、时序漂移和强动态背景未完整覆盖。
- 分镜切换和同时存在的双 360 物理视角不同；没有原生 ERP/spherical BA、跨物理相机时间同步研究。
- RTE/ROE 仍较大，摄像机 ATE 不一定提高；不能将 world HMR 改善等同于 camera-human mutual refinement。
- 数据帧数的正文/表格推算不一致；官方 GitHub 当前仅 README，**方法实现尚未可复现**。

## 个人评价

**推断：**HumanMM 的价值主要是“长序列中的世界坐标连续性测试设计”和跨观测片段对齐，不是双相机同步融合的新核心算法。其 Masked LEAP-VO 表明 camera RPE 改善也可能不改善 global ATE，实验应严格拆分不同相机和人体指标。可视为 TRAM/GVHMR 与连续重定位之间的重要对照。

## 与我的研究关联

**推断：**在双 360° 滑雪里，可故意把长视频切成不重叠、间歇可见的观测片段，注入虚拟镜头切换，复现相邻 clip heading alignment、contact-aware trajectory integration，与已有 per-axis reliability / cross-view attention 比较。推荐设计四层消融：`per-clip HMR → +shot/visibility segmentation → +spherical camera/heading alignment → +world root/contact refinement`。报告 camera ATE/RPE、world root RTE/ROE、W-MPJPE、foot sliding 和 jerk，分层研究连续丢失时间及不同相机内参的影响。

## 后续阅读

- 优先阅读论文 Sections 3.2–3.5、4–5，以及 Tables 1、4、5；尤其检查多镜头世界坐标定义和相机轨迹复位方式。
- 对照 [TRAM](2024-tram-global-trajectory-and-motion.md)、[GVHMR](2024-gvhmr.md)、[SmartMocap](2023-smartmocap-uncalibrated-moving-cameras.md) 与 [Multi-moving-camera sports HMR](2025-sports-mesh-multiple-dynamic-cameras.md)。
- 检查官方 HumanMM-code 是否真正发布可运行代码；确认 ms-Motion 统计中的帧数矛盾。
- **推断：**在球面图像上用 human silhouette / joint ray consistency 作为跨镜头重定位 cue；不要把不同时刻相机视图当作同步多视角几何约束。

## 核验来源（一手）

- https://arxiv.org/abs/2503.07597 （原文完整方法、Table 1–5）
- https://openaccess.thecvf.com/content/CVPR2025/html/Zhang_HumanMM_Global_Human_Motion_Recovery_from_Multi-shot_Videos_CVPR_2025_paper.html （CVPR 出版信息）
- https://zhangyuhong01.github.io/HumanMM/ （作者项目页）
- https://github.com/zhangyuhong01/HumanMM-code （官方代码仓库发布状态）
- https://huggingface.co/datasets/YuhongZhang/ms-Motion （作者发布的数据资源）
