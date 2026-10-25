---
title: "PACT: End-to-End Learning of Human Pose, Contacts, and Forces from Video"
authors: "Rikhat Akizhanov, Yangsong Zhang, Nikolai Kaliazin, Peter Wolf, Yoshihiko Nakamura, Pascal Fua, Fabio Pizzati, Ivan Laptev"
venue: "arXiv:2610.00451 (preprint, v3)"
year: 2026
reading_date: 2026-10-26
status: skimmed
tags:
  - sports-biomechanics
  - contact-estimation
  - force-estimation
  - monocular-video
  - world-motion
  - physics-supervision
---

# PACT: End-to-End Learning of Human Pose, Contacts, and Forces from Video

## 基本信息

- **作者：** Rikhat Akizhanov, Yangsong Zhang, Nikolai Kaliazin, Peter Wolf, Yoshihiko Nakamura, Pascal Fua, Fabio Pizzati, Ivan Laptev
- **会议/期刊：** arXiv:2610.00451（in submission，尚未核验正式录用）
- **首次提交：** 2026-09-30；**最新核验版本：** v3，2026-10-22
- **年份：** 2026
- **阅读日期：** 2026-10-26
- **阅读状态：** `skimmed`
- **标签：** `sports-biomechanics`、`contact-estimation`、`force-estimation`、`world-motion`、`physics-supervision`
- **论文：** https://arxiv.org/abs/2610.00451
- **DOI：** https://doi.org/10.48550/arXiv.2610.00451
- **代码：** https://github.com/rihat99/PACT （官方仓库已建立，但模型、权重、标注工具尚标为 Coming soon）
- **数据集：** 暂无（ForceWall 为论文提出的测力基准，目前尚无已核验可下载发布地址）
- **项目主页：** https://rihat99.github.io/PACT/
- **论文状态：** arXiv 预印本 v3；官网标注 in submission，实质性代码和数据待发布
- **主要价值类型：** Method Module / Dataset / Baseline / Related Work
- **阅读优先级：** A+

## 一句话总结

PACT 将单目视频到世界坐标人体、肢体接触和接触力的预测统一为端到端学习，并以 ForceWall 真实传感器数据验证手足分担的接触力；它提示人体视觉和物理交互不应永远采用互不反馈的串联流水线。

## 研究问题与动机

一般 HMR 只恢复人体姿态，后续再由接触检测或逆动力学推断接触力，分阶段误差累积，并可能丢失视频中的支撑几何信息。现有力监督稀缺，很多步态动力学算法还默认平地、双脚接触。攀岩等手足同时承担重量的动作更能暴露只用骨架推断受力的歧义。作者尝试从真实单目视觉证据中联合学习接触位置、力和人体轨迹，并借助刚体动力学约束训练。

## 核心方法

1. **基础视觉特征：** 使用 SAM 3 跟踪人体、冻结 SAM 3D Body backbone，重训练人体/相机输出头适配 SMPL-X，在 backbone 中加入 **35 个 contact-force tokens**。
2. **世界运动输入：** 使用 VGGT-Ω 估计相机轨迹，并由人体 metric scale 做全视频尺度对齐，将 camera-frame 人体映射到 world frame；组合视频视觉特征、人体相对关节运动及速度。
3. **时序与联合反馈：** temporal transformer 聚合接触/力的短期变化，pose correction、contact head、force head 进行迭代修正；接触点的预测控制哪些力可施加。
4. **物理监督：** 将作用在肢体上的力、重力、root wrench、joint torque 和 contact stillness 纳入学习目标，同时预测姿态、接触概率和接触力。
5. **伪标签与实测评价：** 几何重建 + 接触标注 + 物理优化自动产生大规模训练伪标签；ForceWall 则在攀岩板嵌入三轴测力单元，专门验证真实力，而非只对自己的伪标签评价。

## 数据集与评价指标

- **BEDLAM2：** 18,076 条序列、约 550 万帧、26,178 个人体（论文 Sec. 4.1 口径）。
- **ClimbingVideos：** 191 个视频的 1,528 个场景、约 4.5 小时；训练 1,419 场景/46 万帧，测试 109 场景/3.1 万帧，并对 14 个接触关节有人工作业。
- **ForceWall：** 2 名攀岩者、3 种墙角度、68 个 clip、约 29,600 个评价帧；4 个独立手足接触面板测量 3D force，是实测测力评估，不是大规模临床队列。
- 另评价 Parkour（28 段）、OpenCap（含 22 段 walking trials 的子集）、RICH；不同数据不可简单合并人数。
- 输入：单目 RGB 视频；输出：SMPL-X 世界空间人体轨迹、肢体接触概率、3D 接触力与重力相关量。指标包括 contact F1、Force MAE [N]、Pearson correlation、MPJPE [mm]、力/力矩残差以及 OpenCap 的分轴力误差。

## 主要结果

- **ForceWall：** PACT 接触力 **MAE 85.5 N**、与实测力的 Pearson correlation **0.725**；改造的 Li et al. (2019)++ 为 **141.4 N / 0.243**，PhysPT 为 **167.8 N / 0.169**。该比较建立在作者描述的各 baseline 适配协议上（Table 2）。
- 同一实验 PACT 的力方向 Angle 为 **14.4°**，优于 Li et al. (2019)++ 的 21.2°；ForceWall 只有 2 位受试者，不能外推为跨人群绝对泛化。
- **消融：** ClimbingVideos-only 模型中，加入 physics loss 后力 MAE 从 **96.0 降至 87.5 N**、力相关从 **0.711 提高至 0.729**（Table 4）；同时若只看重建 MPJPE，部分组件并非单调改善。
- **OpenCap：** 作者报告单目 PACT 在 OC-All 的多方向 GRF 误差上超过其他所列单目方法，但与两视角 OpenCap 的部分指标仍有差距（Table 3）。

## 优点

- 把接触力、人体与相机运动及视觉证据统一到可微学习框架，训练标签来自可扩充的伪标注管线。
- ForceWall 用真实测力传感器验证四肢受力，而不仅用视觉漂亮程度或模拟力作为验证。
- 从攀爬拓展到 Parkour/OpenCap 等不同动作，展示与传统分阶段方法的关系和适用差别。

## 局限

- **作者明确限制：** 场景接触物体必须刚性且静止；手持物体、柔性支撑（绳索、蹦床）、肌肉力及其个体差异均未建模（Appendix D）。
- **泛化边界：** ForceWall 仅 2 名攀岩者，训练伪力依赖估计相机、人体形体、接触几何和物理优化，可能系统性偏差。
- **代码状态：** GitHub 仓库存在但目前明确写明 `Coming soon`，无法按官方实现直接复现实验或下载 ForceWall。
- **推断：** 对滑雪而言，雪板持续滑动、非刚性雪面、杖/雪板外力不满足当前刚性静止手足接触的直接假设；不能将预测的 force 当作真实足底压力或肌肉激活。
- **推断：** 当前主要是 camera estimate → human/force 的单向输送，并未验证人体接触力反向降低 camera ATE/RPE。

## 个人评价

最有说服力的是实测 ForceWall 力相关改善和 physics/contact-token 消融；其价值是将“视觉姿态更准确”和“受力估计更可信”分开评价。单目预测真实肢体接触力仍强依赖假设，应重点关注不同动作、受试者与摄像机配置的外部验证。新预印本和代码待发布意味着短期适合先复现思想，而非声称可直接部署。

## 与我的研究关联

- **推断：** 对双 360° 滑雪可以建立 `pose-only → + temporal motion → + contact tokens → + physical consistency`，并与已有 axis-wise reliability fusion 联合；需要额外把雪板/雪面/雪杖和滑动摩擦建模，不能直接移植 ordinary foot-stillness loss。
- **推断：** 在多视角世界坐标重建后利用 RTK/IMU、boot/insole pressure 或接触传感器作 **独立**验证，分别报告 MPJPE、W-MPJPE、CoM、contact timing、pressure/force error，避免将低人体误差自动解释为正确动力学。
- **推断：** 对临床步态可以借鉴 contact + force 的结构化中间变量，但需要患者级、跨中心的关节运动学和力板验证；仅模拟训练力不满足临床有效性。
- **推断：** 若后续测试 human/force→camera refinement，应构造双向优化并比较相机 ATE/RPE，不能因模型有 world-space human 输入就宣称完成了 mutual refinement。

## 后续阅读

- 重点读 **Sec. 3.1–3.2、Sec. 4、Tables 2–4、Appendix D**，尤其复现物理损失、contact-force tokens 与摄像机/尺度误差敏感性。
- 跟踪官方 PACT 仓库是否正式发布代码、ForceWall、预训练模型和 annotation pipeline；不可将当前占位 README 误写成可复现发布。
- 对照 GRIP、BioHuman、BadmintonGRF、OpenCap Monocular 与 GRF/CoP 实测研究，分离 kinematics/metric localization/contact/force 的误差来源。
- 一手事实来源：https://arxiv.org/html/2610.00451v3 ；https://rihat99.github.io/PACT/ ；https://github.com/rihat99/PACT
