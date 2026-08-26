---
title: "Flex4DHuman: Flexible Multi-view Video Diffusion for 4D Human Reconstruction"
authors: "Jen-Hao Cheng, Yipeng Wang, Hao Zhang, Gengshan Yang, Jenq-Neng Hwang"
venue: "arXiv:2606.13655 [cs.CV]"
year: 2026
reading_date: 2026-08-08
status: skimmed
tags:
  - multiview
  - video-diffusion
  - camera-pose
  - 4d-reconstruction
  - gaussian-splatting
---

# Flex4DHuman: Flexible Multi-view Video Diffusion for 4D Human Reconstruction

## 基本信息

- **作者：** Jen-Hao Cheng, Yipeng Wang, Hao Zhang, Gengshan Yang, Jenq-Neng Hwang
- **会议/期刊：** arXiv:2606.13655 [cs.CV]
- **年份：** 2026
- **阅读日期：** 2026-08-08
- **阅读状态：** `skimmed`
- **标签：** `multiview`, `video-diffusion`, `camera-pose`, `4d-reconstruction`, `gaussian-splatting`
- **论文：** [arXiv](https://arxiv.org/abs/2606.13655)
- **代码：** [Andy-Cheng/Flex4DHuman](https://github.com/Andy-Cheng/Flex4DHuman)（官方仓库注明代码仍在准备发布）
- **数据集：** [Multi-view Caption Dataset](https://huggingface.co/datasets/andaba/multi-view_caption)
- **项目主页：** [Flex4DHuman Project Page](https://andy-cheng.github.io/Flex4DHuman/)

## 一句话总结

Flex4DHuman 把相对相机 SE(3) 几何直接写入视频 diffusion Transformer 的 positional encoding，仅依赖参考视频和相机位姿，就能从单目或稀疏多视角输入生成同步的密集多视角视频，并进一步用于 4D Gaussian Splatting 重建。

## 研究问题与动机

动态人体的自由视点生成通常需要目标视角的显式人体几何，例如 SMPL skeleton、depth、normal 或预渲染的 target-view geometry。这类方案虽然有效，但会继承上游几何估计误差，并且容易绑定到特定人体表示和固定相机设置。

Flex4DHuman 希望回答一个更一般的问题：是否可以只利用相对相机位姿，不依赖人体 skeleton、depth 或 normal，就把单目或稀疏多视角动态视频扩展为同步、密集的多视角视频？如果这些生成视图具有足够的跨视角和时间一致性，就可以再交给现成的 4D reconstruction 方法构建动态 3D/4D 表示。

## 核心方法

### 1. 基于 Wan 2.1 的多视角视频 diffusion

模型从 Wan 2.1 1.3B text-to-video DiT 初始化，尽量保留原有 backbone。输入由 noisy latent、clean conditioning latent 和 reference/target mask 组成，所有视角和时间 token 在同一个 Transformer 中联合交互。

### 2. 五轴 positional encoding

作者对原有 spatio-temporal RoPE 做的核心改动是加入 view 和 camera geometry 两个维度，将 attention head 的 positional bands 组织为：

`time + view + SE(3) + height + width`。

其中离散 view index 用于区分视角槽位，连续 SE(3) 部分基于 PRoPE 将相机变换直接作用到 query/key，使 attention 显式依赖 token 之间的相对相机变换。每个序列先以第一台相机为参考，并将平移归一化到单位距离。

这套设计没有额外增加可学习 camera embedding，也不要求固定参考相机数量或固定 rig 布局。

### 3. 三阶段训练 curriculum

- **Stage 1：** 单参考、单目标、单帧设置，用于适应新的 camera-aware positional encoding。
- **Stage 2：** 在总视角数固定的情况下随机改变参考视角数量，训练从可变 reference views 生成 target views；同时加入 background-drop augmentation。
- **Stage 3：** 加入动态时间窗口和 teacher-forced history conditioning，使模型可以通过 overlapping chunks 做长视频 temporal rollout。

### 4. Multi-view captions

作者对 DNA-Rendering、ActorsHQ 和 DFA/Artemis 的多视角序列生成外观描述，并将文本作为可选条件。对于人体，caption 更强调外观而非精细动作，因为作者在 pilot study 中发现不同相机角度容易让自动 motion caption 混淆左右或前后方向。

### 5. 4D Gaussian Splatting 下游

生成得到的同步多视角视频可以直接输入 FreeTimeGS 等动态 Gaussian Splatting 方法。论文展示了从单个静态相机视频出发，先生成密集 target views，再重建动态 4D Gaussian actor 的完整流程。

## 数据集与评价指标

### 数据集

- **DNA-Rendering：** 主要训练与 in-distribution 评估数据；作者使用 Diffuman4D 处理版本，包含 1038 条 human performance sequence、548 个 identity 和 48-camera rig。
- **ActorsHQ：** 不参与训练，用于 zero-shot cross-rig 评估；论文使用 14 条 sequence，每条评估 200 帧。
- **DFA / Artemis：** 用于验证相同模型形式在动物类别上的迁移能力。

### 指标

主要在 foreground 区域计算：

- **PSNR ↑**
- **SSIM ↑**
- **LPIPS ↓**

ActorsHQ 评估不是直接比较生成帧，而是先将生成多视角视频拟合成 FreeTimeGS，再从 ground-truth cameras 重渲染后计算指标。

## 主要结果

### DNA-Rendering：1 个 reference、47 个 target views

| 方法 | PSNR ↑ | SSIM ↑ | LPIPS ↓ |
| --- | ---: | ---: | ---: |
| MV-Performer | 17.44 | 0.7204 | 0.2697 |
| Diffuman4D-mono-skeleton | 16.12 | 0.8760 | 0.1580 |
| Diffuman4D-GT-skeleton | 24.23 | 0.9479 | 0.0744 |
| Flex4DHuman-unmatted | 25.27 | 0.9268 | 0.0977 |
| Flex4DHuman-fg | **25.44** | **0.9516** | **0.0617** |

论文报告 Flex4DHuman-fg 相对 Diffuman4D-GT-skeleton 提升 1.21 dB PSNR、0.0037 SSIM，并将 LPIPS 降低 0.0127；相对单目 skeleton 和 MV-Performer 的 PSNR 优势分别为 9.32 dB 和 8.00 dB。

### Reference-view scaling

在同一个 checkpoint 下，参考视角从 1 个增加到 2 个、4 个时，target-view 平均 PSNR 从 25.21 提升到 28.62 和 31.90 dB，说明模型能够直接吸收额外参考视角，而不需要重新训练。

### ActorsHQ zero-shot cross-rig

| 方法 | PSNR ↑ | SSIM ↑ | LPIPS ↓ |
| --- | ---: | ---: | ---: |
| Diffuman4D-mono-skeleton | 17.97 | 0.815 | 0.307 |
| Flex4DHuman-fg | **21.32** | **0.856** | **0.277** |

在未见过的 ActorsHQ camera rig 上，Flex4DHuman 相比单目 Diffuman4D 提升 3.35 dB PSNR、0.041 SSIM，并将 LPIPS 降低 0.030。

### Temporal rollout

在相同 42 帧窗口中，`T=4` 与 `T=16` 的 chunk rollout 分别达到 24.79 和 24.86 dB PSNR。作者据此认为 teacher-forced history conditioning 可以支持较稳定的长时间扩展，而短 chunk 可作为节省显存的方案。

## 优点

- 相机几何不是额外拼接的 metadata，而是直接进入 attention positional encoding，使跨视角关系成为 Transformer 的基础位置信号。
- 只使用 relative camera pose，不依赖 SMPL、skeleton、depth、normal 或 target-view geometry，减少对人体特定先验的绑定。
- 同一个 checkpoint 可以接受不同数量的参考视角，并外推到不同 target camera layouts。
- 对跨相机 rig 的 zero-shot 测试较有说服力，因为 ActorsHQ 不参与训练。
- 生成结果并非只做视觉展示，还通过 4D Gaussian Splatting 下游验证了跨视角一致性是否足以支持重建。

## 局限

- **论文明确指出：** 当前训练数据仍以静态 studio multi-camera rig 为主，因此对动态相机运动、in-the-wild 环境以及极端 tilt、top-down 等视角的泛化有限。
- **论文明确指出：** 虽然 PRoPE 支持任意逐帧 camera transform，但要提高真实动态相机鲁棒性，仍需要加入 dynamic multi-camera capture 数据。
- **论文明确指出：** teacher-forced history 可以支持 temporal rollout，但更长时间仍存在 drift，需要 self-forcing、diffusion forcing 或其他 long-horizon consistency 方法。
- **资源成本：** 论文报告训练使用 32 张 H100 GPU；因此它更适合作为前沿方法和表示设计参考，而不是低成本基线。
- **代码状态：** 官方 GitHub 仓库已建立，但截至阅读日期仍注明正式 code release 正在准备中。

## 个人评价

这篇论文最值得关注的部分不是“视频 diffusion 可以生成新视角”本身，而是作者如何把**相对相机 SE(3)** 当作 attention positional relation。相机旋转和平移不再只是网络外部的后处理参数，而是直接决定不同 view tokens 如何互相关注。

从研究方法上看，它提供了一条与传统 triangulation、feature fusion 和 pose fusion 不同的路线：如果几何关系能够足够自然地编码到 Transformer attention 中，模型可以学习在不同相机配置之间迁移，而不必固定 camera IDs 或使用预定义 rig embedding。

阅读优先级：**高**。

## 与我的研究关联

这是“相邻前沿”而不是可以直接替换当前 3D pose pipeline 的方法，但相机编码设计非常值得借鉴。

当前多视角人体融合网络如果已经有每个 view 的 3D pose feature，可以尝试把视角之间的相对旋转和平移从简单拼接 feature，升级为类似 PRoPE 的 attention geometry encoding。这样 cross-view attention 的权重就不仅由人体姿态 feature 决定，也显式受 camera-to-camera SE(3) 关系约束。

对于 360° 视频，多个 perspective crop 虽然来自同一个相机中心，但具有已知的虚拟相机旋转；这恰好构成一个相对几何非常明确的特殊场景。**推断性建议：** 可以先只编码 relative rotation，再在上下双 360 或真实多相机设置中加入 translation，比较 rotation-only、translation-only、SE(3) joint encoding 与无 camera feature 的差异。

另一个可探索方向是把 Flex4DHuman 看成“稀疏视角补全器”：当某些方向人体严重遮挡时，先生成新的 canonical target views，再交给人体估计器。不过这一方案的计算量、生成误差和 3D metric fidelity 都需要单独验证，目前不能假设生成视图一定优于直接多视角融合。

## 后续阅读

- 深入阅读 PRoPE，理解 SE(3) 如何作用于 Q/K positional rotation，并评估是否能缩小成适合 pose token 的轻量版本。
- 将固定 camera embedding、相对旋转/平移 feature concatenation、ray encoding 与 PRoPE-style attention encoding 做对比。
- 研究 360° perspective views 中“同中心、不同旋转”的特殊几何是否需要完整 SE(3)，还是 SO(3) 编码已经足够。
- 跟踪官方代码发布后实际测试不同 reference-view 数量和动态 camera trajectory 的鲁棒性。
