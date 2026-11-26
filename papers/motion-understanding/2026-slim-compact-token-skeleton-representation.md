---
title: "Less is More: Compact-Token Masked Feature Prediction for Skeleton Representation Learning"
authors: "Jeonghyeok Do, Yun Chen, Geunhyuk Youk, Munchurl Kim"
venue: "NeurIPS 2026 (accepted, author-confirmed)"
year: 2026
reading_date: 2026-11-27
status: skimmed
tags:
  - skeleton
  - self-supervised
  - masked-feature-prediction
  - contrastive-learning
  - missing-joints
---

# Less is More: Compact-Token Masked Feature Prediction for Skeleton Representation Learning

## 基本信息

- **作者：** Jeonghyeok Do, Yun Chen, Geunhyuk Youk, Munchurl Kim
- **会议/期刊：** NeurIPS 2026（arXiv v4 与作者项目确认；正式 proceedings 暂无核验）
- **首次提交：** 2026-03-11；v4：2026-11-05
- **年份：** 2026
- **阅读日期：** 2026-11-27
- **阅读状态：** `skimmed`
- **标签：** skeleton、self-supervised、masked-feature-prediction、contrastive-learning、missing-joints
- **论文：** https://arxiv.org/abs/2603.10648
- **DOI：** https://doi.org/10.48550/arXiv.2603.10648
- **代码：** https://github.com/KAIST-VICLab/SLiM
- **数据集：** NTU RGB+D 60/120：https://rose1.ntu.edu.sg/dataset/actionRecognition/；PKU-MMD II：https://struct002.github.io/PKUMMD/ （既有数据）
- **项目主页：** https://kaist-viclab.github.io/SLiM_site/

## 一句话总结

以紧凑骨架 token、无重建 decoder 的教师原型预测和全局—局部对比学习获得高效且具有缺失关节鲁棒性的运动表示。

## 研究问题与动机

骨架对比学习可能忽略局部动态，坐标重建式 MAE 则依赖密集 token 和训练用 decoder；相邻关节的相关性还会使遮挡重建退化成插值。

## 核心方法

- 将 64 帧按 8 帧时间 patch 编码为 8×25 的 token 网格；共享 encoder 结合 masked feature prediction (MFP) 与 global-local contrastive learning (GLCL)。
- EMA teacher 提供原型分布，student 预测被遮挡 token 的原型分配，使用交叉熵而非单纯重建 XYZ；不使用坐标重建 decoder。
- Semantic Tube Masking 在时间上遮挡相连身体区域，随机比例 50–90%；Skeleton-Aware Augmentations 包括绕垂直轴旋转、镜像和骨长缩放。
- 输出骨架运动 embedding，用冻结 encoder 线性评价、少标注分类、检索及跨数据集迁移。

## 数据集与评价指标

NTU-60（60 类/56,880 样本）、NTU-120（120 类/114,480 样本；包含 NTU-60）及 PKU-MMD Phase II（13 人、41 类、1,009 段短视频为原始数据规模）；使用 Kinect 25 关节 XYZ 单流。比较 SkeletonMAE、MAMP、S-JEPA、GFP、AMR 等；Top-1 accuracy（X-Sub/X-View/X-Set）、1%/10% 标注、k-NN retrieval、transfer、GFLOPs 和人工缺失/噪声测试。

## 主要结果

- Table 1：NTU-60 线性 X-Sub/X-View 87.9%/93.2%，AMR 87.4%/92.3%；NTU-120 81.2%/83.6%。PKU-II 59.7%，低于 AMR 60.3%。
- Table 2：NTU-60 仅 1% 标签 X-Sub/X-View 为 72.1%/75.8%；k-NN retrieval 为 72.5%/89.9%。
- Tables 7/8：8×25 token 下推理 encoder 3.59 GFLOPs，常规 30×25 为 28.32；同为 8×25 时 SLiM 平均 accuracy 90.55%，MAMP 80.90%。不是总训练加速比。
- Table 10：坐标重建 86.96%，直接 teacher feature MSE 86.92%，teacher prototype CE 90.17%（消融平均 accuracy），说明仅删除 decoder 不能解释增益。
- Table 11：显式 mask 8/25 个关节时 X-Sub/X-View 仍为 87.23%/92.14%；用父关节填充且不声明缺失时 X-Sub 66.51%。

## 优点

- 同 token budget 的对比、原型目标消融、缺失关节表示方式对比，能分辨性能来源。
- 官方项目及 PyTorch 代码可访问，结果包含少标签与跨数据集协议。

## 局限

- 推断：Kinect 室内动作分类结果不能证明单目 HMR、滑雪或老年临床步态泛化；高比例遮挡和镜像可能删除关键病理或左右方向信息。
- 官方 Table 8 说明 GFLOPs 是下游 encoder 或单视图模块成本，不是多视图教师学生训练总成本。Table 11 的随机损坏重复不是预训练随机种子方差。
- OpenReview 入口遇浏览器验证，arXiv HTML 无法访问；会议状态及数值由 arXiv v4、作者项目和代码替代核验，未核验 proceedings。

## 个人评价

推断：阅读优先级高；价值类型 Method Module / Baseline / Related Work。值得重点复现显式 missing mask 与 teacher prototype prediction，而非只增加 MAE。

## 与我的研究关联

推断：CanonFuse3D 可以对双视角低置信关节显式 mask，用可信视角或 EMA teacher 的运动原型训练共享时序 encoder；对齐坐标后再比较 XYZ reconstruction、teacher feature MSE 与 prototype CE。ClinicalGated 可做 clinical ROI tube masking，但不应把 Non-ASD 当健康，也不应无条件左右翻转；评估 patient-disjoint 分类、ROI 一致性及置信度分层。周期运动保留相位信息，比较 8 与更密集 token，检查压缩是否损失短时关节峰值。

## 后续阅读

- 官方项目 Tables 1–2、5、7–11；尤其检查 declared missingness 与 unflagged zero/parent filling。
- 对照 [MotionBERT](2023-motionbert-human-motion-representations.md)、[AMR](2026-adaptive-masked-reconstruction.md)。
- 事实来源：https://arxiv.org/abs/2603.10648 、https://kaist-viclab.github.io/SLiM_site/ 、https://github.com/KAIST-VICLab/SLiM 及上述数据集官方页面。
