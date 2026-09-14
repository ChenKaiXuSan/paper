---
title: "Universal Skeleton Understanding via Differentiable Rendering and MLLMs"
authors: "Ziyi Wang, Peiming Li, Xinshun Wang, Yang Tang, Kai-Kuang Ma, Mengyuan Liu"
venue: "ICML 2026"
year: 2026
reading_date: 2026-09-14
status: skimmed
tags:
  - action-recognition
  - motion-understanding
  - skeleton
  - multimodal
  - MLLM
  - VLM
  - differentiable-rendering
  - cross-format
  - explainable-learning
---

# Universal Skeleton Understanding via Differentiable Rendering and MLLMs

## 基本信息

- **作者：** Ziyi Wang, Peiming Li, Xinshun Wang, Yang Tang, Kai-Kuang Ma, Mengyuan Liu
- **会议/期刊：** ICML 2026（arXiv:2603.18003；v1: 2026-03-18，v5: 2026-05-21）
- **年份：** 2026
- **阅读日期：** 2026-09-14
- **阅读状态：** `skimmed`
- **标签：** `action-recognition`, `motion-understanding`, `skeleton`, `multimodal`, `MLLM`, `VLM`, `differentiable-rendering`, `cross-format`, `explainable-learning`
- **论文：** https://arxiv.org/abs/2603.18003
- **DOI：** https://doi.org/10.48550/arXiv.2603.18003
- **代码：** https://github.com/wangzy01/SkeletonLLM
- **数据集：** 暂无（论文使用 NTU-60/120、NW-UCLA、HumanML3D 等既有数据集，没有论文专属数据集页）
- **项目主页：** 暂无

## 一句话总结

SkeletonLLM 不为不同 skeleton topology 单独训练数值编码器，而是用可微分 DrAction renderer 把任意骨架序列转换成 MLLM 原生可理解的视觉 token，再结合判别式微调与因果推理蒸馏，实现 open-vocabulary recognition、cross-format transfer、captioning 与 motion QA 的统一建模。

## 研究问题与动机

Skeleton motion 具有隐私友好、外观无关和高压缩率等优点，但不同采集系统的骨架拓扑差异很大：Kinect v2 为 25 joints、MoCap/SMPL 常为 22 joints、2D pose 常为 17 joints。传统 GCN/Transformer 往往依赖固定 topology；motion tokenizer / VQ-VAE 又会把连续运动离散化，跨格式时通常需要 joint remapping 或重新训练。

与此同时，MLLM 已经具备强大的视觉-语言推理能力，但无法原生读取 structured skeleton sequence。SkeletonLLM 因此提出一个不同思路：不把骨架强行编码成语言 token，而是把它“翻译”为 MLLM 本来就擅长处理的视觉模态，并让任务梯度反向学习应该如何可视化运动。

## 核心方法

### 1. DrAction：可微分 skeleton renderer

DrAction 用 3D Gaussian Splatting 表示 joint / bone primitives，并用 Linear Blend Skinning 将 Gaussian 与 kinematic chain 绑定。这样既保持骨架结构和时间连续性，又不依赖固定 joint 数量或固定 topology。

Neural Feature Modulator 根据局部 kinematics、depth、velocity 等运动属性调节 Gaussian appearance；最终 renderer 把骨架序列变为紧凑 RGB pseudo-images。由于 renderer 全程可微，MLLM 的任务 loss 可以直接反向更新视觉表示，而不是使用固定 skeleton visualization。

### 2. MLLM integration

论文以 **InternVL3-8B** 为 backbone。每条 skeleton sequence 均匀采样 **12 frames**，渲染为 **448×448** 图像，经过 MLLM vision encoder、projector 与 language model 完成 recognition / captioning / QA。

### 3. 四阶段 cooperative training

作者针对“随机 renderer 一开始生成的图 MLLM 看不懂，而 renderer 又需要有意义的 MLLM gradient 才能学会”的 chicken-and-egg 问题，设计四阶段训练：

1. **Alignment Warm-up：** 冻结 MLLM，只训练 DrAction，以 multiple-choice recognition 建立可理解的视觉协议；
2. **Disc-FT：** 用容易混淆的动作对做 Yes/No hard-negative discrimination；
3. **CR-Distill：** 从 teacher MLLM 蒸馏逐步的 body-part causal reasoning；
4. **Recognition Refinement：** 固定 DrAction，进一步微调 projector 与 LLM LoRA。

默认实现采用 LoRA `rank=32, alpha=64`，在 **2× NVIDIA H20** 上训练。

## 数据集与评价指标

主要评价覆盖不同 skeleton format：

- **NTU RGB+D 60：** Kinect v2、25 joints、60 类，原始官方论文报告超过 **56,000 个视频样本、40 名受试者**；SkeletonLLM 使用 55/5、48/12、40/20、30/30 seen/unseen splits。
- **NTU RGB+D 120：** Kinect v2、25 joints、120 类，原始官方论文报告超过 **114,000 个视频样本、106 名受试者**；使用 110/10、96/24、80/40、60/60 splits。
- **NTU-60 (2D)：** 从 RGB 估计的 17-joint 2D pose，用于 MoCap→2D cross-format transfer。
- **NW-UCLA：** Kinect v1、20 joints，用于 cross-format action recognition。
- **HumanML3D：** SMPL 22-joint motion-language data，用于 captioning 与 cross-format transfer。

Open-vocabulary recognition 主要报告 **Accuracy**；cross-format captioning 使用 R-Precision、BERTScore 等；Motion QA 使用 accuracy。Cross-format experiments 明确要求目标 skeleton format **不做 finetuning**。

## 主要结果

- **NTU-60 open-vocabulary：** SkeletonLLM 在 55/5、48/12、40/20、30/30 splits 上分别为 **87.37%、64.72%、46.15%、37.84%**；TDSM 对应为 86.49%、56.03%、36.09%、25.88%。最困难 30/30 split 上提升 **11.96 个百分点**。
- **NTU-120：** 110/10、96/24、80/40、60/60 分别达到 **76.05%、67.20%、44.37%、34.94%**；60/60 上比 TDSM 的 27.21% 高 **7.73 个百分点**。
- 与同样基于 **InternVL3-8B + 固定 renderer** 的 baseline 相比，SkeletonLLM 在 NTU-60 48/12 与 30/30 上分别提高 **8.44 与 9.69 个百分点**，说明可微 renderer 本身带来明显收益。
- **Cross-format，不在目标域微调：** NTU-60→NW-UCLA 达 **60.38%**，TDSM 43.19%、MotionGPT 10.35%、SKI-LVLM 31.87%；HumanML3D→NW-UCLA 达 **56.73%**；HumanML3D→NTU-60 2D 达 **40.36%**。
- 同一个 NTU-60 55/5 checkpoint 不重新训练即可覆盖多任务：open-vocab recognition **87.37%**，NW-UCLA cross-format **60.38%**，HumanML3D captioning **37.28 BERTScore**，Skeleton-QA 为 **68/65% accuracy**。

## 优点

- 将不同 skeleton topology 统一成视觉接口，避免为每种 joint layout 设计专门 encoder / remapping。
- DrAction 是端到端可微的，renderer 不是手工固定图像，而是由下游 MLLM 任务本身决定哪些运动信息应被突出。
- 同一 checkpoint 同时支持 recognition、captioning、QA 和跨格式 transfer，验证了 representation 的通用性。
- Skeleton 作为输入天然减少外观、背景和身份信息，对于医疗、驾驶员和运动分析中的 privacy-preserving multimodal learning 很有吸引力。

## 局限

- 作者明确指出当前实验仍以**短时 atomic actions** 为主；分钟级、由多个 sub-actions 组成的真实行为需要 hierarchical temporal modeling 和 memory。
- 视觉表示的 token efficiency 较低：骨架只占图像很小区域，大量像素是黑背景；分辨率从 448×448 降到 224×224 会带来约 **5%** 性能下降，说明当前 dense visual encoding 成本较高。
- 主模型依赖 InternVL3-8B，并在 2×H20 上训练；CR-Distill 还依赖 teacher-generated reasoning，计算和数据生成成本显著高于轻量 skeleton encoder。
- **推断：**通过 renderer 把结构数据变成图像虽然解决 topology mismatch，却可能丢失原始 3D 数值精度；对于临床 joint angle、毫米级位移等任务，应验证 rendered representation 是否保留精细 biomechanical quantity。

## 个人评价

**价值类型：** Method Module / Baseline / Related Work  
**阅读优先级：** A

这篇论文的价值不是单纯把 MLLM 用到 skeleton action recognition，而是提出了一种很通用的“structured signal → differentiable visual language → foundation model reasoning”接口。对于已有 RGB / optical flow / keypoints / VLM 研究，它提供了一个比直接把关键点坐标转文本更自然的多模态路线。

我尤其关注其 cross-format transfer：如果同一个模型确实能从 Kinect 25-joint、SMPL 22-joint迁移到 20-joint / 17-joint skeleton，而不依赖手工 joint mapping，这对跨医院、跨 pose estimator、跨运动数据集的统一表示非常实用。

## 与我的研究关联

**推断：** 可以将该思路迁移到三个现有方向：

1. **临床步态：** 将 3D skeleton、trunk/pelvis angle、gait phase、左右 asymmetry 或 uncertainty 映射成 task-aware visual tokens，再让 MLLM 解释“哪些身体部位、哪个 phase 支持 ASD / severity 判断”；
2. **体育 / 生物力学：** 把 skeleton 与 velocity、CoM、contact、joint-angle deviation 一起编码为 differentiable visual representation，和 biomechanics-grounded VLM/coaching 比较；
3. **驾驶员行为：** 以 head pose + upper-body skeleton 作为隐私友好的 motion channel，再与 RGB road/context branch 融合。

建议比较 `raw coordinates / GCN → fixed skeleton rendering + VLM → SkeletonLLM-style differentiable rendering → + biomechanics channels → + RGB/context fusion`。对于临床任务，应额外测试跨 pose estimator / skeleton topology / center zero-shot generalization，而不能只看同数据集 classification accuracy。

## 后续阅读

- 重点细读 Sec. 3.2 DrAction、Sec. 3.4 cooperative training、Sec. 4.3 cross-format transfer，以及 Appendix G 的 long-horizon / token-efficiency 局限。
- 复现官方 NTU-60 pipeline 后，优先尝试将现有 gait skeleton 转成 22/25/17-joint 混合格式，检查无需 joint remapping 的迁移能力。
- 与 NextMotionQA、MotionVLA、BioGait-VLM 以及传统 ST-GCN / temporal transformer 进行统一的准确率、解释质量、算力与跨格式泛化比较。
