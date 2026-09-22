---
title: "Encoder-Free Human Motion Understanding via Structured Motion Descriptions"
authors: "Yao Zhang, Zhuchenyang Liu, Thomas Ploetz, Yu Xiao"
venue: "arXiv:2604.21668"
year: 2026
reading_date: 2026-09-23
status: skimmed
tags:
  - motion-understanding
  - motion-language
  - llm
  - biomechanics
  - interpretable-learning
  - skeleton
  - clinical-gait
---

# Encoder-Free Human Motion Understanding via Structured Motion Descriptions

## 基本信息

- **作者：** Yao Zhang, Zhuchenyang Liu, Thomas Ploetz, Yu Xiao
- **会议/期刊：** arXiv:2604.21668
- **年份：** 2026
- **阅读日期：** 2026-09-23
- **阅读状态：** `skimmed`
- **标签：** `motion-understanding`, `motion-language`, `llm`, `biomechanics`, `interpretable-learning`, `skeleton`, `clinical-gait`
- **论文：** https://arxiv.org/abs/2604.21668
- **DOI：** https://doi.org/10.48550/arXiv.2604.21668
- **代码：** https://github.com/yaozhang182/motion-smd
- **数据集：** https://huggingface.co/datasets/zyyy12138/motion-smd-data
- **项目主页：** https://yaozhang182.github.io/motion-smd/

## 一句话总结

Structured Motion Description（SMD）用确定性的 biomechanical joint-angle、global trajectory 与 temporal-segment rules 将 3D skeleton motion 直接变成结构化文本，使通用 LLM 无需 learned motion encoder 或 cross-modal alignment 就能进行 motion QA 与 captioning，并提供天然的人可读解释接口。

## 研究问题与动机

现有 motion-language 模型通常先用 VQ-VAE、VAE 或其他 motion encoder 把 skeleton sequence 压缩成 latent tokens，再学习与 LLM embedding space 的对齐。这类 pipeline 训练阶段多、与特定 backbone 绑定，而且 latent representation 本身难以解释，也可能对不同 motion acquisition domain 敏感。

SMD 反过来利用一个事实：LLM 已经理解“左膝屈曲”“向前移动”“右肩抬高”这类身体部位、方向和动作语义。与其训练新的 motion tokenizer，不如先把人体运动转换成 LLM 原生可读的 biomechanical language，再只用轻量 LoRA 适配下游任务。

这一设计特别适合需要可解释 motion reasoning 的场景，因为输入本身就是定量 joint angle、时间区间和 global trajectory，而不是无法直接阅读的 latent code。

## 核心方法

输入为 `T × J × 3` joint positions；对于 SMPL representation，`J=22`。SMD 首先建立 pelvis-centered body-local coordinate frame，然后按照 biomechanical conventions 计算 **26 个 joint-angle channels**，覆盖 **13 个 body-part groups**，包括 pelvis、lumbar spine、neck，以及双侧 hip/knee/ankle/shoulder/elbow。

Global trajectory 使用 pelvis 的 forward/backward、lateral、height 与 body yaw。Angle 和 trajectory time series 先平滑，再通过 peak-valley segmentation 压缩成带时间范围的自然语言段，例如某关节从多少度增加到多少度、某段保持、或出现重复周期；joint angle 变化阈值默认 5°，trajectory translation/yaw threshold 分别为 0.03 m / 15°。

最终文本由 meta information、global trajectory、joint-angle sections 构成。All-26 版本在 HumanML3D 上约 4,000 tokens，Top-3 active-joint 版本约 1,000 tokens。任务端不训练 motion encoder，只冻结 LLM base weights并训练 LoRA；默认 backbone 为 Qwen2.5-7B-Instruct，约 40M trainable parameters。

## 数据集与评价指标

- **BABEL-QA：**1,109 motions、2,577 QA pairs；train/val/test 为 1,800/384/393 QA pairs。
- **HuMMan-QA：**925 motions、3,123 QA pairs；train/val/test 为 2,066/524/533 QA pairs。
- **HumanML3D：**14,616 motions、44,970 captions；用于 motion captioning。
- **官方 SMD 数据发布：**包含上述三类数据的 SMD texts、预处理 QA / HumanML3D subset，并提供多种 SMD variants。
- **输入：**3D skeleton joint sequence，经 deterministic SMD conversion 后作为 text prompt 输入 LLM。
- **输出：**motion QA answer 或 natural-language motion caption。
- **QA 指标：**exact-match accuracy。
- **Captioning 指标：**R-Precision、MM-Distance、BLEU、ROUGE-L、CIDEr、BERTScore。

论文将 BABEL-QA 与 HuMMan-QA 统一成固定 10-option protocol，并对多数 baselines 重新训练，以减少原 benchmark 不同 option count 带来的比较偏差。

## 主要结果

默认 Qwen2.5-7B backbone 下，SMD 在 BABEL-QA / HuMMan-QA 达到 **66.7% / 90.1% accuracy**；此前 IMoRe 为 **60.1% / 75.2%**。同样使用 Qwen2.5-7B、但采用 VAE motion tokens 的 controlled MotionGPT3-Qwen 只有 **50.1% / 22.0%**，尤其显示 learned motion representation 在 cross-source HuMMan data 上的脆弱性。

HumanML3D motion captioning 上，SMD 达到 **R@1 0.584、R@3 0.883、MM-Dist 2.35、CIDEr 53.16、BERTScore 45.58**；MotionGPT3 为 **0.573、0.864、2.43、40.6、35.2**。论文还验证 8 个 LLM、6 个 model families，说明同一 SMD text representation 可跨 backbone 复用。

Attention visualization 显示，在“walking in place”样例中模型会关注 trajectory 与 cyclic joint-angle segments；“waving with right hand”则主要关注 right shoulder / elbow descriptions，为 motion-language reasoning 提供了直接的人可读分析入口。

## 优点

- 不需要 learned motion encoder、VQ-VAE 或 motion-language alignment，结构简单且跨 LLM backbone。
- Joint angles 与 global trajectory 采用明确的 biomechanical semantics，对 gait / sports / clinical motion 较容易解释。
- 统一 backbone 的 controlled baseline 能较清楚地把收益归因到 motion representation，而不是单纯模型规模。
- 对 BABEL-QA 与 HuMMan-QA 的跨数据来源表现差异给出了有价值的 representation-robustness 证据。
- 官方代码、derived data 与多个 LoRA adapters 均已发布。

## 局限

- 论文明确指出 All-26 SMD 约 4,000 tokens，约为 VAE-based 256 motion tokens 的 15 倍，导致 inference latency 增加。
- 当前规则固定在 22-joint SMPL 上定义 26 个 biomechanical angles，hands/fingers 等更细粒度运动可能被遗漏。
- Fixed smoothing / segmentation thresholds 可能平滑掉极慢或爆发性动作，作者将 motion-adaptive thresholds 列为未来方向。
- **推断：**SMD 依赖上游 skeleton 的准确性；如果 monocular HMR、360° pose 或 clinical video 中 joint positions 本身存在 depth/occlusion error，结构化文本会把这些误差显式写进角度与轨迹描述。
- **推断：**attention over readable tokens 提高了可审查性，但不等于 clinical causality；用于疾病诊断时仍需要患者级、跨中心和混杂因素控制验证。

## 个人评价

这篇工作最有价值的地方不是“又把 skeleton 喂给 LLM”，而是重新定义了 motion-language interface：先用 biomechanical conventions 做一个确定、可解释、跨域的中间 representation，再利用 LLM 的现有语言与身体知识。它与 SkeletonLLM 的“把 skeleton 渲染成视觉 token”形成很好的对照，一个走视觉接口，一个走 biomechanical-text 接口。

对医疗或体育任务而言，SMD 可能比通用 action caption 更实用，因为 joint angle、time segment、周期和 global trajectory 本身就接近临床/教练语言。不过它目前主要证明了 QA/captioning，而不是诊断、预后或高精度 biomechanical measurement。

## 与我的研究关联

**推断：**临床 gait 可以把 3D gait reconstruction 转成 `trunk/pelvis/hip/knee/ankle angle + gait phase + asymmetry + cyclicity + global trajectory` 的结构化描述，再与 RGB/VLM、clinical text 或 diagnosis head 融合。这样可以比较：

`raw keypoints → learned motion encoder → SMD text → SMD + RGB/VLM → SMD + clinical priors`。

对 ASD gait，可进一步把 sagittal/coronal trunk compensation、pelvic rotation、左右 ROM asymmetry、步长/速度写成结构化 token；对滑雪可加入 turn phase、joint flexion、CoM/contact 相关描述。建议重点复现 §3.1 joint-angle construction、§4.2 same-backbone baseline、§4.5 attention analysis，并额外测试 noisy HMR / camera error 对 SMD stability 的影响。

## 后续阅读

- [Universal Skeleton Understanding via Differentiable Rendering and MLLMs](2026-skeletonllm.md) — 将 heterogeneous skeleton 渲染成视觉 token 的另一种 foundation-model 接口。
- [NextMotionQA: Benchmarking and Judging Human Motion Understanding with Vision-Language Models](2026-nextmotionqa.md) — motion understanding / reasoning benchmark。
- [BioHuman: Learning Biomechanical Human Representations from Video](../sports-biomechanics/2026-biohuman.md) — 从视觉进一步预测内部 biomechanics，可与 SMD 的可解释表示形成互补。
- [Clinical-Prior Guided Multi-Modal Learning with Latent Attention Pooling for Gait-Based Scoliosis Screening](../medical-ai/2026-clinical-prior-scoligait.md) — clinical-prior + gait representation 的医学应用方向。