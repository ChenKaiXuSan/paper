---
title: "Toward Universal Skeleton-Based Action Recognition across Heterogeneous Skeletons and Open Vocabularies"
authors: "Jidong Kuang, Hongsong Wang, Jie Gui, Yuan Yan Tang, James Tin-Yau Kwok"
venue: "arXiv"
year: 2026
reading_date: 2026-08-30
status: skimmed
tags:
  - motion-understanding
  - skeleton-action-recognition
  - heterogeneous-skeletons
  - open-vocabulary
  - motion-text-alignment
---

# Toward Universal Skeleton-Based Action Recognition across Heterogeneous Skeletons and Open Vocabularies

## 基本信息

- **作者：** Jidong Kuang, Hongsong Wang, Jie Gui, Yuan Yan Tang, James Tin-Yau Kwok
- **会议/期刊：** arXiv:2604.17013（v2: 2026-08-18）
- **年份：** 2026
- **DOI：** 10.48550/arXiv.2604.17013
- **阅读日期：** 2026-08-30
- **阅读状态：** `skimmed`
- **标签：** `motion-understanding`, `skeleton-action-recognition`, `heterogeneous-skeletons`, `open-vocabulary`, `motion-text-alignment`
- **价值类型：** Baseline / Method Module / Dataset / Related Work
- **阅读优先级：** A（高）
- **论文：** https://arxiv.org/abs/2604.17013
- **代码：** https://github.com/jidongkuang/Universal-Skeleton
- **数据集：** 原始 NTU RGB+D、NW-UCLA、HumanML3D 需从各自官方来源获取；官方仓库提供 HumanML3D 400-class annotation / split / label map 等预处理文件
- **项目主页：** https://jidongkuang.github.io/projects/Universal-Skeleton/

## 一句话总结

该工作把不同来源的 2D/3D skeleton topology 映射到统一 31-joint 表示，并通过多粒度 motion-text alignment 学习一个可跨 Kinect、COCO、SMPL 等骨架格式和开放动作词汇迁移的通用 skeleton action representation。

## 研究问题与动机

现有 skeleton-based action recognition 通常假定训练和测试使用相同的 joint topology、坐标维度和封闭动作类别。例如 Kinect v1 / v2、COCO-17、SMPL-22 的关节数量和语义并不一致，导致一个模型很难直接复用到不同 pose estimator、临床数据集或体育数据集。另一方面，传统 closed-set classifier 也无法自然处理训练中没有出现的动作语义。

作者希望同时解决两个问题：heterogeneous skeleton formats 与 open-vocabulary action recognition，使不同骨架来源能够进入共享 motion encoder，并借助语言语义实现跨格式和未见动作类别迁移。

## 核心方法

首先建立统一的 31-joint canonical skeleton，将 Kinect v1 20-joint、Kinect v2 25-joint、COCO-17 2D 和 SMPL-22 3D 等表示通过 joint mapping 与 kinematic imputation 转换到统一拓扑。输入同时构造 joint、bone 和 motion 三类 skeleton streams，以减少单一坐标表示对具体格式的依赖。

模型使用两级 Transformer 对空间关节关系与时间运动进行编码，并引入 CLIP text embedding 作为动作语义空间。训练包含 global motion-text alignment，以及 stream-specific / temporal / body-part 的 fine-grained alignment，使 joint、bone、motion 与不同时间尺度和身体区域都能与动作语言对齐。

作者进一步构建 HOV（Heterogeneous Skeleton and Open Vocabulary）评估设置，将 NTU RGB+D、NW-UCLA 和 HumanML3D 统一到同一研究框架中，测试同格式识别、跨格式迁移以及 zero-shot / generalized zero-shot recognition。

## 数据集与评价指标

NTU RGB+D 60 包含 56,880 个样本和 60 个动作类别；NTU RGB+D 120 包含 114,480 个样本和 120 类；NW-UCLA 包含 1,494 个动作样本。HumanML3D 包含 14,616 条 motion 和 44,970 条自然语言描述。

为了进行 skeleton open-vocabulary evaluation，作者从 HumanML3D 文本中抽取动作词并聚类：初始约 8,801 个 action tokens 被整理为 400 个 action categories，并采用约 70/30 的分层划分。类别还按照频率划分为 head / medium / tail，用于观察 long-tail recognition。

主要评价包括 closed-set top-1 accuracy、cross-format transfer accuracy，以及 zero-shot / generalized zero-shot recognition 指标。

## 主要结果

完整 J+B+M 模型结合 semantic alignment 后，在 NTU-60 3D、NTU-60 2D 和 HumanML3D 上的 top-1 accuracy 分别达到约 87.01%、90.10% 和 61.23%。在 HumanML3D 上，已有 PURLS baseline 的 overall accuracy 为 55.22%，而该方法提高到 61.23%；few-shot / tail 类别也有明显提升。

跨 skeleton format 时，NTU-60 3D 训练后迁移到 NW-UCLA 可达到 91.59%；HumanML3D 训练后直接迁移到 NTU-60 3D / 2D 分别达到约 74.81% / 74.91%，说明统一 topology 与语言语义确实提供了跨来源迁移能力。

在 HumanML3D open-vocabulary 设置中，zero-shot accuracy 约为 43.26%；generalized zero-shot 的 harmonic mean 约为 33.24%。这些结果表明 skeleton-language alignment 能处理一定程度的未见动作类别，但与 fully supervised closed-set recognition 仍存在明显差距。

## 优点

- 直接解决 skeleton joint topology 不一致这一实际问题，对不同 pose estimator、数据集和传感器之间的模型复用具有现实价值。
- 同时考虑 2D / 3D、joint / bone / motion 与语言语义，而不是只做一个固定格式的 action classifier。
- 提供跨格式、long-tail、zero-shot 和 generalized zero-shot 多种评价，而不仅仅报告单一数据集 accuracy。
- 官方代码与 HumanML3D open-vocabulary annotation / splits 已公开，便于复现实验协议。

## 局限

- 作者明确指出，直接迁移到 unseen scenario 的表现仍弱于针对目标场景进行 fine-tuning；“universal”并不意味着完全无适配部署。
- 当前只验证 human skeleton，不包含 humanoid robot 等不同运动学结构。
- 原始大型数据集仍需遵循各数据集官方获取流程，官方仓库不会重新分发全部 raw data。
- **推断：**HumanML3D 的 open-vocabulary benchmark 最终仍被整理为一个 400-class candidate universe，而且 zero-shot 设置中的真正 unseen 类别数量有限，因此它与自由文本条件下完全开放的动作理解仍有距离。

## 个人评价

这篇论文最值得借鉴的部分不是最终 action classifier，而是“先消除 skeleton topology 差异，再学习共享时序表示”的设计。当前很多跨数据集实验容易把 COCO、OpenPose、SMPL、Kinect 等输出当作不同任务，实际其中一部分 domain gap 来自 joint definition 本身。统一 topology 可以让后续模型更专注于运动模式而不是输入格式。

**推断：**对于临床步态、体育动作和驾驶员行为，可以把不同 pose estimator / camera setup 的 skeleton 统一后，再训练共享 temporal backbone；语言对齐则可进一步提供语义辅助监督和可解释动作标签。相比直接把所有 skeleton zero-padding 到相同维度，这种显式 canonicalization 更适合作为跨数据集泛化 baseline。

## 与我的研究关联

可用于临床步态、体育动作和驾驶员姿态分析中的跨数据源 representation learning。例如，可以把 COCO-17 2D keypoints、SMPL-derived joints、不同 3D pose estimator 输出统一成 canonical skeleton，再比较：

1. source-specific skeleton encoder；
2. 简单 joint mapping / zero padding；
3. unified 31-joint canonicalization；
4. canonicalization + motion-text semantic alignment。

对于多视角 3D reconstruction，该方法也可以作为 downstream motion consistency module：先获得 per-frame 3D pose，再在统一 skeleton space 中判断动作时序是否合理，而不受前端 body model 的具体 joint definition 限制。

## 后续阅读

- 测试 canonical skeleton 对 missing joints、joint confidence 和 pose-estimation noise 的敏感性。
- 将 deterministic kinematic imputation 替换为带 uncertainty 的 joint completion，并观察跨格式迁移是否更稳定。
- 在临床 gait / sports 数据上进行 source-disjoint transfer，比较是否需要 target-domain fine-tuning。
- 研究 motion-text alignment 是否可以生成更细粒度、与生物力学变量对应的可解释动作语义。
