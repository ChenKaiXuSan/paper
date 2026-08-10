---
title: "Superman: Unifying Skeleton and Vision for Human Motion Perception and Generation"
authors: "Xinshun Wang, Peiming Li, Ziyi Wang, Zhongbin Fang, Zhichao Deng, Songtao Wu, Jason Li, Mengyuan Liu"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-11
status: skimmed
tags:
  - 3d-human-pose
  - human-motion
  - multimodal-learning
  - motion-tokenization
  - motion-prediction
  - motion-inbetweening
---

# Superman: Unifying Skeleton and Vision for Human Motion Perception and Generation

## 基本信息

- **作者：** Xinshun Wang, Peiming Li, Ziyi Wang, Zhongbin Fang, Zhichao Deng, Songtao Wu, Jason Li, Mengyuan Liu
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **arXiv：** arXiv:2602.02401；v1 提交于 2026-02-02，v2 修订于 2026-07-07
- **阅读日期：** 2026-08-11
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `human-motion`, `multimodal-learning`, `motion-tokenization`, `motion-prediction`, `motion-inbetweening`
- **论文：** [arXiv](https://arxiv.org/abs/2602.02401) · [CVPR 2026 Open Access](https://openaccess.thecvf.com/content/CVPR2026/html/Wang_Superman_Unifying_Skeleton_and_Vision_for_Human_Motion_Perception_and_CVPR_2026_paper.html)
- **DOI：** [10.48550/arXiv.2602.02401](https://doi.org/10.48550/arXiv.2602.02401)
- **代码：** [BradleyWang0416/Superman-CVPR2026](https://github.com/BradleyWang0416/Superman-CVPR2026)
- **数据集：** 暂无（未发布专用数据集；实验使用 Human3.6M 与 3DPW）
- **项目主页：** 暂无

## 一句话总结

Superman 针对 3D pose estimation、motion prediction 和 motion in-betweening 长期由不同模型分别处理、视觉感知模型难以直接生成结构化运动而 motion generation 模型又缺少 raw-video grounding 的问题，提出 Vision-Guided Motion Tokenizer 将视觉与 3D skeleton 联合量化为统一 motion vocabulary，再用单一 Qwen2.5-VL-based MLLM 完成感知与生成任务，在 Human3.6M 上取得更低的 3D pose error，并在完全未见的 3DPW 上显著改善 motion prediction/in-betweening。

## 研究问题与动机

人体运动研究通常被拆成两类：一类从图像/视频“读出”姿态或动作，另一类从 text 或 skeleton condition “写出”新的动作序列。前者往往不能生成结构化 3D motion，后者通常无法直接理解 raw visual input；此外，现有 motion tokenizer 多由 skeleton 单模态训练，离实际视觉证据较远。

Superman 的目标是把 temporal 3D pose estimation、motion prediction 和 motion in-betweening 统一成 conditional sequence generation。作者把人体运动视为一种离散语言：如果视觉和骨架能够共享同一个 motion token vocabulary，单一 MLLM 就可以根据 video、skeleton sequence 或 text condition 读写同一种结构化人体运动表示。

## 核心方法

### 1. Vision-Guided Motion Tokenizer（VGMT）

VGMT 基于 VQ-VAE，把连续 3D skeleton motion 转换成离散 motion tokens。与只用 skeleton 的 tokenizer 不同，它采用双流表示：

- **Skeleton Encoder：** 编码 `F × N × 3` 的 3D joint geometry；
- **Visual stream：** 对 video frames 提取 feature map，以投影后的 2D joints 为 reference locations，通过 Visual-Skeleton Attention（VSA）采样并聚合 joint-centric visual features；
- **Hybrid codebook：** 每个 token 同时具有 visual prototype 与 geometric prototype，让量化过程同时受到外观和 3D 骨架几何约束。

VGMT 最终使用 skeleton decoder 重建 3D pose sequence。论文的 codebook size 为 **8192**。

### 2. 单一 MLLM 统一三个任务

作者以 **Qwen2.5-VL-7B** 为基础 MLLM，把 VGMT 产生的 motion token vocabulary 接入语言模型。通过改变条件输入和输出格式，同一个模型完成：

1. **PE（3D Pose Estimation）：** video → 3D skeleton sequence；
2. **MP（Motion Prediction）：** 历史 3D skeleton → 未来 skeleton sequence；
3. **MIB（Motion In-Betweening）：** start/end poses → 中间 motion sequence。

训练分两阶段：先独立训练并冻结 VGMT，再用所有任务混合训练 MLLM，统一使用 autoregressive cross-entropy objective。

### 3. Motion-Aware Fine-Tuning（MAFT）

为了进一步提高 video-based pose estimation，作者设计可选的 MAFT。它使用固定 2D pose estimator 提供 joint reference locations，再由 multi-scale deformable sampling 提取 pose-centric visual features，并使用 VSA cross-attention 把 skeleton-guided feature 注入 ViT visual tokens。

MAFT 的额外参数小于基础模型的 0.2%；论文报告 VSA 与 MAFT 等新模块合计只占总 computation 的不到 0.03%，但整体计算仍由 Qwen2.5-VL-7B 主导。

## 数据集与评价指标

### 数据集与划分

- **Human3.6M：** 唯一训练数据集，同时用于 in-domain 测试；
- **3DPW：** 完全排除在训练之外，用于 zero-shot generalization 测试；作者将 3DPW SMPL vertices 映射为 Human3.6M skeleton 格式后统一处理。
- 论文按照两个数据集的标准 train/test splitting protocol 进行实验，但主实验段落没有重新给出总 frame/sequence 样本数，因此这里不额外推断具体样本量。
- temporal 模型的主要设置为 **T=16 frames**。

### 评价任务与指标

- **3D Pose Estimation（PE）：** `MPJPE` 与 `N-MPJPE`，单位 mm；
- **Motion Prediction（MP）：** 报告平均误差以及 80/160/320 ms 等未来时间点误差；
- **Motion In-Betweening（MIB）：** 报告平均、middle、last 等位置的 MPJPE；
- 3DPW zero-shot table 主要报告 MP 和 MIB，不把 3DPW 作为 PE 的完整外部验证表。

## 主要结果

### Human3.6M：三个任务统一建模

在 T=16 设置下：

| 方法 | PE N-MPJPE ↓ | PE MPJPE ↓ | MP Avg ↓ | MIB Avg ↓ |
| --- | ---: | ---: | ---: | ---: |
| MotionBERT | 47.07 | 56.70 | 29.94 | 42.37 |
| Skeleton-in-Context | 45.26 | 55.57 | 33.42 | 31.27 |
| Human-in-Context | 44.77 | 53.86 | 26.66 | 31.13 |
| **Superman + MAFT** | **39.41** | **51.61** | **26.13** | **30.61** |
| Superman（无 MAFT） | 44.90 | 61.39 | **26.13** | **30.61** |

MAFT 只影响有视觉输入的 PE；MP 与 MIB 输入均为 3D poses，因此两个 Superman 版本在这两项上相同。相较 Human-in-Context，Superman+MAFT 的 N-MPJPE 从 44.77 降到 **39.41 mm**。

### 3DPW：zero-shot motion generalization

所有方法均只在 Human3.6M 训练，3DPW 完全不参与训练：

| 方法 | MP Avg ↓ | MP 80ms ↓ | MP 320ms ↓ | MIB Avg ↓ | MIB mid ↓ |
| --- | ---: | ---: | ---: | ---: | ---: |
| MotionBERT | 164.96 | 137.89 | 200.14 | 123.05 | 148.51 |
| Skeleton-in-Context | 140.71 | 110.77 | 183.74 | 103.97 | 127.68 |
| Human-in-Context | 141.90 | 112.53 | 183.62 | 108.54 | 131.99 |
| MotionGPT3 | 228.17 | 190.83 | 259.49 | 180.10 | 204.71 |
| **Superman** | **62.05** | **34.75** | **97.37** | **60.68** | **63.35** |

3DPW MIB 平均误差为 **60.68 mm**，明显低于 Skeleton-in-Context 的 103.97 mm；MP 平均误差也由最佳传统 baseline 的 140.71 mm 降到 **62.05 mm**。

### 计算开销

作者报告模型在 2 张 NVIDIA H20 GPU 上训练。虽然新增 VSA/MAFT 的额外开销很低，但 Qwen2.5-VL-7B 仍占整体约 **99.7% GFLOPs 和 98.6% latency**，说明统一 MLLM 的主要成本来自基础大模型本身。

## 优点

- 把 3D pose perception 与 motion generation 放进同一个离散 motion language，而不是维护三个完全不同的任务网络。
- tokenizer 不只看 skeleton，而是利用视觉信息参与量化，使 motion code 更贴近真实图像证据。
- 同一模型同时覆盖 video→pose、pose→future motion 和 keyframe→intermediate motion，适合研究表示共享与多任务迁移。
- Human3.6M→3DPW zero-shot 的 MP/MIB 改善幅度大，表明 learned motion vocabulary 具有一定跨数据集泛化能力。
- MAFT 显式把 2D joint location 与 visual feature 通过 attention 融合，与 joint-guided/clinical-attention 设计有直接方法论联系。
- 官方代码仓库已公开。

## 局限

- **外部泛化范围有限：** 训练只使用 Human3.6M，3DPW 用于 zero-shot MP/MIB；论文没有提供医疗步态、疾病患者、体育动作或多视角场景的专门验证。
- **最佳 PE 仍依赖额外 2D pose cue：** Superman+MAFT 的最佳 PE 结果使用 video + fixed 2D pose estimator。无 MAFT 的直接 video→3D 版本 MPJPE 为 61.39 mm，而 MAFT 后为 51.61 mm，因此外部 2D keypoint 质量仍是一个实际影响因素。
- **基础模型计算成本高：** 虽然作者新增模块很轻量，但 Qwen2.5-VL-7B 占据几乎全部 GFLOPs 和 latency；若目标是实时临床 gait 或嵌入式体育分析，直接部署成本较高。
- **骨架格式统一带来信息约束：** 3DPW 被转换成 Human3.6M skeleton 格式，适合通用关节运动比较，但不能直接代表 SMPL surface、手部细节或临床解剖学精度。
- **推断：** 离散 codebook 是否能保留细微、低幅度的病理步态特征仍需要临床数据验证，论文现有实验不能证明这一点。

## 个人评价

这篇论文最值得关注的是“视觉—骨架共享 motion vocabulary”，而不是把 MLLM 本身作为所有人体运动任务的最终答案。对运动分析来说，tokenizer 提供了一种中间表示：既不像 raw RGB 那样难解释，也不像单纯 joint coordinates 那样完全脱离外观和遮挡证据。

另一个值得借鉴的点是 MAFT。它不是简单把 RGB feature 与 keypoints concatenation，而是以 2D joints 作为空间 reference，抽取局部视觉证据后再通过 cross-attention 写回 visual tokens。这种结构比普通 early/late fusion 更有明确的身体部位对应关系。

阅读优先级：**高**。

## 与我的研究关联

**相邻前沿。** 这篇工作与人体动作识别、时空表征学习、RGB+keypoint 多模态融合和临床运动分析都有方法层面的联系。

可迁移的方向包括：

- 将周期步态序列先离散成 motion tokens，再研究不同 gait phases、病理模式或 skill levels 是否在 token space 中形成稳定 cluster；
- 借鉴 MAFT 的 joint-guided attention，把 spine、pelvis、knee、ankle 等临床关键部位作为视觉采样与注意力锚点，而不是全局 RGB feature 平均融合；
- 比较 skeleton-only tokenizer 与 vision-guided tokenizer 在遮挡、motion blur、视角变化时的鲁棒性；
- 用多任务方式同时学习 `pose reconstruction + gait classification + future motion prediction`，测试辅助运动任务是否改善疾病分类泛化；
- 在多视角重建中研究是否可以先把不同 view 的 pose/motion 映射到统一 token vocabulary，再进行 cross-view fusion。

以上为**推断性研究建议**，论文并未在临床 gait、ASD 或多视角 360° 数据上验证这些结论。

## 后续阅读

- 深入阅读 Vision-Guided Motion Tokenizer 的 hybrid codebook 与 VSA 细节，特别关注 visual/geometric prototype 的量化策略。
- 对比 MotionBERT、Skeleton-in-Context、Human-in-Context 与 Superman 在 representation sharing 上的差异。
- 复现 MAFT 的 `video + 2D joint cue` 融合，并测试 causal、低延迟或更小 backbone 版本。
- 在临床步态数据上检查 motion token 的可解释性：不同疾病等级、步态相位和异常关节是否对应稳定 token patterns。
