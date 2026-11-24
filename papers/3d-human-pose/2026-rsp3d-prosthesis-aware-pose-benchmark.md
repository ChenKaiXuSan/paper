---
title: "Prosthesis-Aware 3D Human Pose Estimation: A Dataset and Benchmark for RSP Users"
authors: "Yilin Wen, Kechuan Dong, Fumiya Suginaka, Ken Endo, Yusuke Sugano"
venue: "ECCV 2026"
year: 2026
reading_date: 2026-11-25
status: skimmed
tags:
  - 3d-human-pose
  - sports
  - clinical-gait
  - prosthesis
  - sam3d-body
  - pointmap
---

# Prosthesis-Aware 3D Human Pose Estimation: A Dataset and Benchmark for RSP Users

## 基本信息

- **作者：** Yilin Wen, Kechuan Dong, Fumiya Suginaka, Ken Endo, Yusuke Sugano
- **会议/期刊：** ECCV 2026
- **年份：** 2026
- **阅读日期：** 2026-11-25
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`、`sports`、`clinical-gait`、`prosthesis`、`sam3d-body`、`pointmap`
- **论文：** https://arxiv.org/abs/2609.18406
- **代码：** https://github.com/ut-vision/RSP3D
- **数据集：** https://ut-vision.github.io/RSP3D/download.html （申请制）
- **项目主页：** https://ut-vision.github.io/RSP3D/
- **首次正式在线发表：** 2026-09-09；arXiv 提交 2026-09-16。
- **DOI：** https://doi.org/10.1007/978-3-032-37095-2_24
- **正式出版：** https://link.springer.com/chapter/10.1007/978-3-032-37095-2_24
- **价值类型：** Dataset / Baseline / Method Module / Related Work
- **阅读优先级：** 高

## 一句话总结

RSP3D 为跑步专用假肢使用者同时标注自然关节及假肢几何，并展示人体模型与无模板点图的互补性。

## 研究问题与动机

传统 HMR 假设人体具有完整解剖拓扑，可能生成不存在的关节；通用点跟踪可表达非人体装备却缺少人体运动学先验。作者建立针对 running-specific prostheses (RSP) 使用者的三维任务与基准，以改善运动和康复分析。

## 核心方法

单目 RGB 视频输入，输出有效自然关节与 RSP 表面点。Hybrid alignment 采用 SAM3D Body 估计人体、SpatialTrackerV2-Pointmap 估计无模板点；根据 GT 截肢部位移除虚构关节。论文 §4.3 通过 XY bounding-box 尺度、pelvis 对齐、2D 投影一致性细化尺度；官方脚本也提供对齐及评估流程。2D detection 版本的 SAM2 假肢掩码从 GT 首帧框初始化，每 10 s 重启；STv2 以 60 帧片段推理。

## 数据集与评价指标

RSP3D：6 人（4 男 2 女），论文 Table 1 报告 5.6M frames（多视角记录，非独立样本）；16 台 GoPro 60 FPS、12 台 OptiTrack Prime 加一台 Prime Color；QR 视频同步及 ChArUco 标定。动作含走、跑、上楼、跨栏和跳跃等。自然关节为 H36M 17 点定义中的有效子集；反光标记用于独立三维参考。指标 root-aligned MPJPE、PA-MPJPE (mm)，假肢 Chamfer Distance、F@100/50 及局部中心对齐 CD-C/F-C。零样本评价；model-free 对照使用 GT 尺度。

## 主要结果

Table 3：2D detection 设置，SAM3D Body MPJPE/PA-MPJPE 78.35/50.89 mm；STv2-Pointmap 159.64/113.87 mm；Hybrid 保留 SAM3D 的 78.35/50.89 mm，假肢 F@50 为 19.92（STv2 18.82），但 CD 288.44 对 277.15，反而变差。Table 4：Hybrid 双侧截肢 MPJPE 91.91 mm、单侧 70.35 mm；双侧 CD 342.22，STv2 294.21。不能将关节改善等同为装备几何改善。

## 优点

实际采集非标准人体，并同时评价有效关节和装备表面；独立 mocap GT、相机同步标定细节以及代码/申请制数据有助于复现。2D detection vs GT、单侧 vs 双侧的对照揭示错误来源。

## 局限

作者指出深度误差与双侧缺少锚点导致全局对齐困难。仅 6 人、室内固定相机；使用 GT 截肢位置与 GT 初始框，尚非全自动未知人体结构重建。model-free baseline 使用 GT 尺度，需区分对照协议。**推断：**大量帧无法替代受试者多样性，尚不能外推至户外高速滑雪或真实 ASD 诊断。

## 个人评价

**推断：**这是检验 foundation HMR 人体先验偏差的好资源。Hybrid 的人体 MPJPE 改善直接来自沿用 SAM3D，而非互相优化；装备 CD 有恶化，适合进一步研究可靠性及深度/尺度联合校正。

## 与我的研究关联

**推断：**滑雪板、雪杖、滑雪靴可以视为人体之外的几何实体：用 SAM3D Body 重建人体，用点图/分割追踪装备，并在球面双视角约束下估计相对位置与尺度；独立评价装备姿态和人体关节，不能只报告 MPJPE。对 ClinicalGated，检查异常/代偿姿态是否被 HMR 健康先验抹除；先做有效关节 mask、人工 2D 审核和患者级 pose reliability 分层。其 16 相机 + OptiTrack + QR + ChArUco 流程也可作为实验室 GT 建设参考。

## 后续阅读

重点 §3.1–3.3、§4.2–4.3、Tables 3–4、§5.2；官方 scripts/alignment.py 与 scripts/eval.py。对照 ResiHMR、SAM3D Body、MotionBERT；先测试装备独立重建，再加身体几何约束与 reliability-aware fusion。
