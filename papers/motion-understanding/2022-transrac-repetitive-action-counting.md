---
title: "TransRAC: Encoding Multi-scale Temporal Correlation with Transformers for Repetitive Action Counting"
authors: "Huazhang Hu, Sixun Dong, Yiqun Zhao, Dongze Lian, Zhengxin Li, Shenghua Gao"
venue: "CVPR 2022 (Oral), pp. 19013–19022"
year: 2022
reading_date: 2026-11-14
status: skimmed
tags:
  - motion-understanding
  - periodic-motion
  - repetitive-action-counting
  - temporal-correlation
  - transformer
  - sports-biomechanics
  - clinical-gait
---

# TransRAC: Encoding Multi-scale Temporal Correlation with Transformers for Repetitive Action Counting

## 基本信息

- **作者：** Huazhang Hu, Sixun Dong, Yiqun Zhao, Dongze Lian, Zhengxin Li, Shenghua Gao
- **会议/期刊：** CVPR 2022 (Oral)，pp. 19013–19022（正式会议论文，经典方法补读）
- **首次提交：** 2022-04-03（arXiv）
- **年份：** 2022
- **阅读日期：** 2026-11-14
- **阅读状态：** `skimmed`
- **标签：** `motion-understanding`, `periodic-motion`, `repetitive-action-counting`, `temporal-correlation`, `transformer`
- **论文：** https://arxiv.org/abs/2204.01018
- **DOI（arXiv）：** https://doi.org/10.48550/arXiv.2204.01018
- **代码：** https://github.com/SvipRepetitionCounting/TransRAC
- **数据集：** https://svip-lab.github.io/dataset/RepCount_dataset.html （RepCount，作者官方页面和下载入口）
- **项目主页：** https://svip-lab.github.io/dataset/RepCount_dataset.html （以官方数据集页作为项目入口；独立模型项目页暂无）
- **价值类型：** Baseline / Method Module / Dataset / Related Work
- **阅读优先级：** 中高；2022 年重要经典补读，并非近期新论文

## 一句话总结

利用视频的多时间尺度自相关矩阵与 Transformer/密度图回归定位和计数重复动作周期，并以带逐周期起止标注的 RepCount 评估节奏变化与动作中断。

## 研究问题与动机

动作周期的速度、持续时间可能不均匀，还会被休息、说话或干扰动作中断。仅预测整段视频的总次数无法验证每个周期在哪里，也不利于真实长视频。作者构建 RepCount，提供逐周期边界，然后使用多尺度视频片段编码和周期密度图回归，兼顾短/长动作。与经典 RepNet 的直接周期/次数预测相比，额外关注时间位置与可解释性。

## 核心方法

1. **输入/输出**：输入 RGB 视频（实验中每段采样 64 帧），输出按时间分布的周期密度图 `D[t]`，其求和为动作次数；逐周期位置可由密度峰值解释。
2. **多尺度编码**：取 1、4、8 帧规模的滑动窗口，Video Swin Transformer Tiny 预训练于 Kinetics-400，提取空间/时间 embedding。
3. **Temporal correlation**：跨时间计算自注意力相关矩阵，将不同尺度的相关性融合。
4. **Density map regression**：利用逐周期起止标注构造目标密度，使用 Transformer-based predictor，而非直接离散次数分类；多个尺度提升变速周期适应性。
5. **训练假设**：Video Swin 编码器冻结，只训练其余部分；因此性能依赖上游 Kinetics 预训练与采样质量。

## 数据集与评价指标

- **RepCount**：共 1,451 段、19,280 个逐周期标注；Part-A 是 1,041 段 YouTube 运动/体育视频，Part-B 410 段学校志愿者模拟体能训练，仅作未见域测试。包括慢速/快速、动作中断和节奏变化。
- 原文 Table 1 提供视频平均时长 **29.359 s**，而正文第 3 节及官方数据集页写 **39.359 s**；因此平均时长存在不一致，不能擅自选一个当作无争议数值。
- 额外测试 **UCFRep 526 段**，对照 **RepNet、Video SwinT、X3D、TANet** 等；所有模型在 RepCount-A 上统一训练的比较见原文 Tables 2–3。
- **指标**：论文所称 MAE 实际为按真实计数归一化的平均绝对误差（越低越好），OBO 为预测次数偏差不超过 1 的比例（越高越好）；不能误认作原始次数的绝对 MAE。

## 主要结果

- RepCount-A 测试：TransRAC **MAE 0.4431 / OBO 0.2913**；Video SwinT **0.5756 / 0.1324**，RepNet **0.9950 / 0.0134**（Table 2，同一训练划分）。
- 在 RepCount-A 训练后零微调到 RepCount-B：**0.7839 / 0.091**；到 UCFRep：**0.6401 / 0.324**；明显跨域退化（Table 3）。
- 消融：Self-attention correlation **0.4431 / 0.2913**，Temporal Self-similarity Matrix **0.5678 / 0.2251**（Table 4）；只用 4 帧尺度 **0.5434 / 0.2649**，多尺度 **0.4431 / 0.2913**（Table 5）。
- 作者公开了训练脚本、部分预训练权重和 RepCount-A 链接；2025-02-06 官方仓库另发布实验设置解释，指出 RepNet/TransRAC 评价协议易出现误解。

## 优点

- 逐周期起止标注比只给总次数更利于 phase、cycle localization、tempo variability 评价。
- 自相关与多尺度模块能自然刻画重复运动的时间结构，适合做可解释时间先验或辅助损失。
- 提供跨数据来源的 Part-B 测试，以及代码和模型/数据入口。

## 局限

- 本质是 RGB repetitive **action counting**，不是以真实 3D joints、解剖学 gait events 或物理时间码为输入；无法直接当作临床步态参数/滑雪转弯相位 GT。
- 实际 OBO **0.2913**，在跨域 Part-B 下降至 **0.091**，说明定位与跨场景泛化仍有限；多人物运动、64 帧抽样丢失前段周期会产生失败案例（原文 Figure 5）。
- 数据集包含公开视频及校内志愿者，存在隐私/偏差问题；Part-B 原始视频的公开获取受限制（官方仓库公告）。
- 论文平均时长统计有不一致；不能用不明确定义的数值外推到真实临床/滑雪场景。

## 个人评价

**推断：**与直接频域 FFT 或固定窗口周期检测相比，自相关图和密度 supervision 可以处理不严格等间距的动作；但对于临床细微病态步态，离散 64 帧采样可能掩盖 stride asymmetry 与极短相位。应在原始 FPS 下检查周期边界误差，而不仅看次数/相对 MAE。

## 与我的研究关联

**推断：**对双视角滑雪与临床步态，可把其自相关/density map 思路迁移至 `3D pelvis/wrist/foot canonical joint signals → multi-scale temporal correlation → cycle start/end density → normalized phase`，再与现有 wrist azimuth upward-crossing、固定 0.8 s 间隔与 64-phase 归一化比较。对 ASD gait 可加入左右步相位、stride time CV、周期可解释性；对滑雪则将转弯起止、左右交替和速度/jerk 稳定性作为评价。**不要**将 RepCount 动作次数直接理解为左右脚 gait-event 或滑雪 edge transition。

## 后续阅读

- 优先阅读 Sections 3–5，尤其 Fig. 3、Tables 2–5 和 Figure 5 的失败例。
- 与 [MotionBERT](2023-motionbert-human-motion-representations.md)、[PoseC3D](2022-posec3d-skeleton-action-recognition.md) 比较：`raw RGB correlation vs pose/3D canonical correlation`。
- 在自建数据上人工标 20–30 段相位 GT，评估 count MAE、cycle-boundary F1、phase RMSE、tempo drift 与不同相机 FPS/遮挡鲁棒性。
- 核验 RepCount 时长冲突；以作者 2025-02-06 补充说明为依据复现公平评估设置。

## 核验来源（一手）

- https://arxiv.org/abs/2204.01018 （原文、数据规模、方法与 Table 1–5）
- https://github.com/SvipRepetitionCounting/TransRAC （作者官方实现、权重与实验设置更新）
- https://svip-lab.github.io/dataset/RepCount_dataset.html （官方数据集与下载入口）
