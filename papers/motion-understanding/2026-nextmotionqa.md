---
title: "NextMotionQA: Benchmarking and Judging Human Motion Understanding with Vision-Language Models"
authors: "Yong Cao, Chuqiao Li, Xianghui Xie, Gerard Pons-Moll, Andreas Geiger"
venue: "arXiv"
year: 2026
reading_date: 2026-09-03
status: skimmed
tags:
  - motion-understanding
  - vision-language-model
  - benchmark
  - motion-captioning
  - human-motion
  - multimodal
---

# NextMotionQA: Benchmarking and Judging Human Motion Understanding with Vision-Language Models

## 基本信息

- **作者：** Yong Cao, Chuqiao Li, Xianghui Xie, Gerard Pons-Moll, Andreas Geiger
- **会议/期刊：** arXiv
- **年份：** 2026
- **阅读日期：** 2026-09-03
- **阅读状态：** `skimmed`
- **标签：** `motion-understanding`, `vision-language-model`, `benchmark`, `motion-captioning`, `human-motion`, `multimodal`
- **论文：** https://arxiv.org/abs/2606.04773
- **代码：** 暂无（官方仓库说明将在论文正式发表后发布代码与评测脚本）
- **数据集：** 暂无公开下载（官方仓库说明将在发表后发布）
- **项目主页：** https://github.com/yongcaoplus/NextMotionQA

## 一句话总结

NextMotionQA 用专家核验的多任务、多语义轴和分难度 benchmark 系统揭示：当前 VLM 在粗粒度动作理解上表现不错，但对身体部位、方向和细粒度动作错误的理解与判断仍显著不可靠。

## 研究问题与动机

人体运动理解正在从传统 action label 走向自然语言描述、问答和 VLM-based reasoning，但常见 benchmark 往往只评价单一任务、类别粒度较粗，而且容易出现答案歧义。因此即使模型在平均分上看起来很强，也很难知道它到底是否真正理解了“哪个身体部位在动、朝什么方向移动、动作语义是否正确”。

作者进一步关注另一个正在流行的用法：使用 VLM 作为 text-to-motion 或 motion generation 的自动 judge。若 VLM 本身在 fine-grained body motion 上就不可靠，那么把它作为评价器可能产生系统性偏差。

## 核心方法

NextMotionQA 构建为一个 `3 × 3 × 3` benchmark：

### 三种任务形式

1. **T1 Multiple-Choice QA**：针对视频回答运动问题；
2. **T2 Free-form Captioning**：生成细粒度 motion caption；
3. **T3 Caption Error Correction**：发现并修正已有 caption 中的运动错误。

### 三个语义轴

1. **Body-Part Involvement**：哪些身体部位参与；
2. **Translation Direction**：人体/身体部位移动方向；
3. **Action Semantics**：动作本身的语义。

### 三个难度等级

Easy / Medium / Hard，用于区分短、简单动作和更长、更复杂的组合动作。

数据构建利用 VLM 半自动生成候选，再进行专家核验，从而减少纯自动标注中的歧义和幻觉。作者同时评价 12 个代表性 VLM，并额外研究 VLM-as-a-Judge 在不同运动复杂度下与专家评分的一致性。

## 数据集与评价指标

Benchmark 共 **1,307 个专家核验实例**，对应 **992 个 unique videos**：

- T1 Multiple-Choice QA：511 examples / 483 videos；
- T2 Captioning：396 examples / 396 videos；
- T3 Caption Error Correction：400 examples / 400 videos。

Motion clips 来自 **16 个 AMASS 子数据集**中的 SMPL-H sequence，30 FPS，并结合 BABEL 与 HumanML3D metadata。

不同难度下平均视频长度大约从 7.6–8.2 秒（Easy）增加到 12.4–15.3 秒（Hard）。

作者评价 12 个 VLM。除各任务/语义轴的 benchmark score 外，VLM-as-a-Judge 部分还使用 instance-level correlation、Cohen's κ 和 system-level correlation 衡量与人工专家的一致性。

## 主要结果

官方结果中，**Gemini-3.1-Flash 总分 58.44**，**Qwen3.6-Plus 54.85**，位于整体 leaderboard 前列。

一个很明显的结论是 **free-form captioning 是所有模型的共同瓶颈**：官方统计指出，没有开源模型在 T2 上超过 **35.57**。相比选择题或纠错，直接准确描述细粒度人体运动仍然困难。

`Translation Direction` 也是所有模型家族的普遍弱项。官方分析指出，closed/open model 在方向语义轴上的差距缩小到约 **8.2 points**，而 Body-Part 轴差距约 **25.0 points**，说明方向理解不是简单依靠更大模型就能完全解决的问题。

VLM-as-a-Judge 的结果更值得注意：

- 单动作粗粒度评价：instance `r=0.774`，Cohen's `κ=0.701`，system `r=0.966`；
- 组合动作：instance `r=0.495`，`κ=0.346`；
- 细粒度 part-level 判断：instance `r=0.116`，`κ=0.104`，system `r=-0.146`。

也就是说，模型作为 judge 在粗粒度上可以和人类相当一致，但一旦要求判断细粒度身体部位运动，其可靠性几乎崩溃。

## 优点

- 不只做单一 action QA，而是同时覆盖问答、caption 和 error correction。
- 明确区分 body part、direction、action semantics，可诊断模型具体失败在哪里。
- 使用 Easy/Medium/Hard 分层，不把所有动作复杂度混在一个平均指标里。
- 由专家核验 1,307 个实例，减少自动 benchmark 中常见的答案歧义。
- 对 VLM-as-a-Judge 给出非常直接的可靠性边界，对使用大模型自动评价人体运动特别重要。

## 局限

- 当前数据主要来自 AMASS / SMPL-H motion，而不是复杂真实视频中的遮挡、背景、视角和 camera motion，因此更接近“motion rendering / motion-centric video understanding”而非完整 in-the-wild perception。
- 规模只有 1,307 个专家核验实例，优势在质量与结构，而不是大规模训练。
- 官方代码、数据和 evaluation scripts 截至本次阅读仍标注为待正式发表后释放，当前复现条件不完整。
- benchmark 关注语言层面的 motion understanding，并不直接验证临床诊断、生物力学量化或 3D reconstruction accuracy。

## 个人评价

这篇最重要的价值不是提供一个新的 VLM backbone，而是提醒研究者：**VLM 对人体动作的“看起来会说”并不等于真的理解细粒度运动。** 特别是身体部位和方向错误，在普通 caption demo 中很容易被忽略。

对于需要可解释医学动作分析或体育 coaching 的系统，如果让 VLM 直接生成“左膝内翻、躯干向右倾、重心向前”之类的结论，就必须有结构化运动证据约束，而不能只靠视频语言模型自由生成。

## 与我的研究关联

对 clinical gait、滑雪技术分析和 driver behavior 都有直接启发。可以把现有 pose / biomechanics 特征转成结构化 evidence，再让语言模型做解释，而不是让 VLM 直接从 RGB 猜运动语义。

**推断：**一个很值得做的实验是比较：

`RGB-only VLM → RGB + 2D pose → RGB + 3D pose → RGB + 3D pose + biomechanics tokens`，

并把评价拆成 NextMotionQA 风格的：

- body-part correctness；
- direction correctness；
- action/pathology semantics；
- hallucination / correction accuracy。

对于临床 gait，可以进一步把“trunk lean、pelvis rotation、step asymmetry、knee flexion”等可量化指标转成 evidence tokens，检查 VLM 是否真的减少 part-level 和 direction-level hallucination。

对于滑雪，可构建 turn phase、edge change、COM lateral motion、upper/lower-body separation 等细粒度问题，而不是只让模型输出泛化的“动作好/不好”。

## 后续阅读

- KPM-Bench：细粒度 kinematic parsing 和 motion hallucination evaluation。
- MotionVLA：视觉、语言与人体 motion token 的统一建模。
- BioGait-VLM / biomechanics-grounded VLM：结构化生物力学信息如何约束语言解释。
- 未来可考虑建立面向 clinical gait / skiing 的小规模 expert-verified MotionQA benchmark。
