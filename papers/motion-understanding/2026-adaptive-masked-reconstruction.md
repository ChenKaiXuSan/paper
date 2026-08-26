---
title: "Exploring Adaptive Masked Reconstruction for Self-Supervised Skeleton-Based Action Recognition"
authors: "Shengkai Sun, Zhiyong Cheng, Zefan Zhang, Jianfeng Dong, Zhihui Li, Meng Wang"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-12
status: skimmed
tags:
  - action-recognition
  - skeleton
  - self-supervised-learning
  - masked-reconstruction
  - motion-representation
---

# Exploring Adaptive Masked Reconstruction for Self-Supervised Skeleton-Based Action Recognition

## 基本信息

- **作者：** Shengkai Sun, Zhiyong Cheng, Zefan Zhang, Jianfeng Dong, Zhihui Li, Meng Wang
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **提交日期：** arXiv v1，2026-06-09
- **阅读日期：** 2026-08-12
- **阅读状态：** `skimmed`
- **标签：** `action-recognition`, `skeleton`, `self-supervised-learning`, `masked-reconstruction`, `motion-representation`
- **论文：** [CVPR 2026 Open Access](https://openaccess.thecvf.com/content/CVPR2026/html/Sun_Exploring_Adaptive_Masked_Reconstruction_for_Self-Supervised_Skeleton-Based_Action_Recognition_CVPR_2026_paper.html) · [arXiv](https://arxiv.org/abs/2606.11450)
- **DOI：** [10.48550/arXiv.2606.11450](https://doi.org/10.48550/arXiv.2606.11450)
- **代码：** [AshenOne1005/AMR](https://github.com/AshenOne1005/AMR)（截至阅读日期，官方仓库只有 README 与 LICENSE，尚未看到完整实现）
- **数据集：** 暂无专用数据集；实验使用 NTU RGB+D 60、NTU RGB+D 120、PKU-MMD II
- **项目主页：** 暂无

## 一句话总结

AMR 针对 skeleton masked reconstruction 需要预测大量时空 patches、训练成本高且对所有区域一视同仁的问题，通过将 decoder 与 encoder 解耦以减少重建目标数量，再利用局部 motion energy 对关键关节和时间段进行 focal reconstruction，在更低计算成本下学习更有判别力的动作表示。

## 研究问题与动机

Skeleton-based action recognition 具有较强的背景和外观不变性，因此适合把人体动作直接建模为关节轨迹。近年来 masked reconstruction 被广泛用于无标签 skeleton pre-training，但标准方案通常需要对大量细粒度时空 patches 进行重建；decoder 中 mask tokens 之间的 self-attention 会进一步增加训练开销。

另一个问题是，标准 MSE reconstruction 会等权处理所有关节与所有时间段。然而对动作语义而言，不同身体部位的重要性并不相同：例如抬手、转头、起身等动作主要由少数关节和局部时间区间决定。作者因此提出两个目标：一是减少无必要的 reconstruction computation，二是让预训练更聚焦于动作语义更强的运动区域。

## 核心方法

### 1. Decoder Decoupling

标准 masked skeleton reconstruction 的 decoder 通常同时执行 mask tokens 之间的 self-attention，以及 mask tokens 与 encoder visible features 之间的 cross-attention。作者认为随机初始化的 mask tokens 本身没有语义，token-token self-attention 对表征学习贡献有限，却会随着 target sequence length 增加显著提高计算量。

AMR 将 decoder 简化为：

`learnable queries → multi-head cross-attention(visible features) → FFN`

也就是说，query 只从 encoder 输出的 visible patch features 中读取信息，不再做 mask-token self-attention。这样 encoder 与 decoder 的 target granularity 被解耦：如果要预测更大的时空 patch，只需要减少 query 数量，而不需要压缩或改变 encoder 输出。

### 2. Larger Patch Reconstruction

作者通过增大 reconstruction target patch 的时间跨度来减少预测目标数量。与传统 750 个 target patches 的配置相比，AMR 的主设置只需要预测 **125 个 patches**。

单纯增大 patch 会使单个 target 内包含更多复杂运动，可能降低重建质量。因此 AMR 不只是做 target reduction，而是进一步加入 focal reconstruction 来找出 patch 内真正重要的区域。

### 3. Focal Reconstruction

AMR 根据局部关节的 motion energy 估计不同区域的重要性。作者在多个 temporal windows 上计算关节运动变化，再融合成多尺度 motion weights，使持续、有判别力的运动获得较高权重，同时减少偶发噪声导致的短暂高权重。

最终 reconstruction loss 是 weighted MSE：运动信息更丰富的 joints/time regions 对 loss 贡献更大，从而引导 encoder 优先学习与 action semantics 相关的运动结构。

这一设计与 object detection 中 focal-style weighting 的思想类似，但这里的权重不是根据分类难度，而是根据 skeleton motion informativeness 动态产生。

## 数据集与评价指标

### NTU RGB+D 60

- **56,880 sequences**；
- **60 类动作**；
- **40 subjects**；
- 多视角采集；
- Cross-subject：20 subjects train / 20 subjects test；
- Cross-view：Views 2–3 train，View 1 test。

### NTU RGB+D 120

- **114,480 sequences**；
- **120 类动作**；
- **106 subjects**；
- **32 setups**；
- 使用 Cross-subject 与 Cross-setup protocol。

### PKU-MMD II

- **6,952 sequences**；
- **51 类动作**；
- **31 camera viewpoints**；
- cross-subject split：5,339 train / 1,613 test。

### 评价设置

论文统一使用 **Top-1 accuracy**，并从四个角度评估预训练表征：

1. linear evaluation；
2. semi-supervised evaluation（1% / 10% labels）；
3. transfer learning；
4. training efficiency，包括 FLOPs、reconstruction patch 数量和 pre-training hours。

## 主要结果

### 1. Linear evaluation 与训练效率

在同一张 **L20 GPU** 上预训练时：

| 方法 | Reconstruction patches | FLOPs | 训练时间 | NTU-60 x-sub | NTU-60 x-view | PKU-II |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| SkeletonMAE | 750 | 13.7G | 29.9 h | 74.8 | 77.7 | 36.1 |
| GFP | 251 | 4.1G | 4.9 h | 85.9 | 92.0 | 56.2 |
| **AMR** | **125** | **3.8G** | **3.7 h** | **87.4** | **92.3** | **60.3** |

相对于 SkeletonMAE/MAMP 采用的 750-target setup，AMR 将 target 数降到 125；论文以 SkeletonMAE 的 29.9 h 为基准报告 **8.1× training speedup**，同时 NTU-60 与 PKU-II 的 linear probing accuracy 更高。

在 NTU-120 linear evaluation 上，AMR 达到 **81.1% x-sub / 81.9% x-setup**；GFP 为 79.1/80.3，S-JEPA 为 79.6/79.9。

### 2. 低标签 semi-supervised learning

NTU-60 只使用部分 labels 时：

| 标签比例 | x-sub | x-view |
| --- | ---: | ---: |
| 1% | **72.2** | **74.4** |
| 10% | **89.0** | **92.7** |

作为对比，GFP 在相同 1% / 10% 设置下为 x-sub 71.8/88.7、x-view 72.9/92.1。AMR 的优势在极低标签设置下仍然存在。

### 3. Transfer learning

从 NTU-60 预训练后迁移到 PKU-II，AMR 为 **70.9%**；从 NTU-120 预训练后为 **73.0%**。这与 MAMP/NAT 的最佳结果接近，但并非所有 transfer setting 都绝对领先，因此更合理的结论是 AMR 在迁移性能与训练效率之间取得了很强的平衡。

### 4. 核心消融

在 125 targets 的统一设置中：

| 配置 | NTU-60 | NTU-120 |
| --- | ---: | ---: |
| Decoder masking baseline | 84.8 | 76.3 |
| Downsampling baseline | 78.9 | 70.2 |
| + Decoupled Decoder | 86.0 | 80.1 |
| + Decoupled Decoder + Focal Reconstruction | **87.4** | **81.1** |

结果说明，大 patch 本身并不足以保证高质量表征；decoder decoupling 负责降低计算并避免信息压缩，而 motion-energy focal weighting 进一步恢复并提高判别能力。

## 优点

- 在 self-supervised skeleton pre-training 中同时考虑**效率**与**表征质量**，并使用同一 GPU 条件报告训练时间，比较较清楚。
- Focal reconstruction 不是依赖 action labels，而是直接从 joint motion energy 得到权重，仍保持 self-supervised 属性。
- 对 1%/10% labels 的实验显示其表征适合低标注场景。
- 同时覆盖 NTU-60、NTU-120 与 PKU-II，并做 cross-dataset transfer，不只报告单一 benchmark。
- motion importance 可以可视化到具体 joint/time region，为后续人体动作解释提供了自然入口。

## 局限

- **适用边界：** 实验全部是通用 skeleton action recognition benchmark，没有临床步态、病理动作或细微功能障碍验证，因此不能直接把通用动作分类增益等同于医疗任务增益。
- **推断：** motion-energy weighting 天然偏向幅度较大的运动。对临床步态中幅度很小但有诊断意义的变化，例如轻微躯干倾斜、左右不对称或关节活动度下降，需要验证该 weighting 是否会错误降低其重要性。
- 输入依赖 skeleton sequence，因此上游 2D/3D pose estimation 的系统性误差可能传入 representation learning；论文的标准 benchmark 无法完全覆盖真实临床视频中的遮挡与关键点漂移。
- 虽然作者提供了官方 GitHub 仓库，但截至 2026-08-12，仓库当前只有 README 与 LICENSE，完整训练/评估代码尚未实际公开，因此目前的可复现性仍有限。

## 个人评价

这篇论文最值得关注的部分不是单纯把 NTU accuracy 再提高一点，而是提出了一个很容易迁移到其他周期运动任务的原则：**在自监督重建时，不必平均学习所有时空区域，而可以根据运动结构动态分配学习预算。**

它同时表明 skeleton self-supervised pretraining 在 1% labels 下仍然具有较强效果，这对小样本临床动作数据比纯 SOTA accuracy 更有实际价值。另一方面，临床异常常常恰恰表现为“动作幅度变小”或“局部差异很微弱”，因此不能直接照搬 motion-energy 权重，最好把 clinical prior、joint importance 或 phase-aware information 一并考虑。

阅读优先级：**很高**。

## 与我的研究关联

AMR 与人体动作识别、周期运动建模和临床运动分析都有直接方法联系。可以借鉴的方向包括：

- 对 gait cycle 做 masked reconstruction 预训练，再用少量疾病 labels fine-tune；
- 把原始 motion-energy focal weight 改成 **joint × gait-phase** 的二维权重，让 stance/swing 中不同关节拥有不同重要性；
- 将 spine、pelvis、knee、ankle 等临床关注区域加入先验权重，与纯数据驱动 motion energy 比较；
- 把当前 RGB / optical flow / keypoint 多模态模型中的 keypoint branch 先进行 AMR-style self-supervised pretraining；
- 检查 focal weights 是否与模型分类 attention、临床 gait features 或医生关注区域一致，从而增强可解释性。

以上迁移设计属于基于论文方法的**推断性建议**，并非 AMR 已在临床数据上验证的结论。

## 后续阅读

- 重点阅读 Sec. 3.2 `Decoder Decoupling` 与 Sec. 3.3 `Focal Reconstruction`，确认 motion weight 的具体计算方式。
- 复现 Table 1 的训练效率比较，并在自己的 gait skeleton 数据上记录 accuracy–GPU hours trade-off。
- 对比纯 motion-energy、learnable attention、clinical-prior weighting 与 phase-aware weighting。
- 在病理步态上专门检查“低幅度但有诊断意义”的关节变化是否会被 focal reconstruction 忽视。
