---
title: "Democratizing Ski Safety: Real-Time Turn Segmentation with Smartphone IMU and Causal LSTM Networks"
authors: "Michał Szymocha, Piotr Kacprzak, Jakub Robak, Wojciech Turek"
venue: "IJCAI 2026, Special Track on AI and Social Good, pp. 7446–7454"
year: 2026
reading_date: 2026-11-20
status: skimmed
tags:
  - sports-biomechanics
  - alpine-skiing
  - imu
  - temporal-segmentation
  - causal-lstm
  - subject-disjoint
  - sensor-fusion
---

# Democratizing Ski Safety: Real-Time Turn Segmentation with Smartphone IMU and Causal LSTM Networks

## 基本信息

- **作者：** Michał Szymocha, Piotr Kacprzak, Jakub Robak, Wojciech Turek
- **会议/期刊：** Proceedings of the Thirty-Fifth International Joint Conference on Artificial Intelligence (IJCAI-26)，AI and Social Good Special Track，pp. 7446–7454；2026 年正式会议论文；具体首次上线日暂无核验
- **年份：** 2026
- **阅读日期：** 2026-11-20
- **阅读状态：** skimmed
- **标签：** sports-biomechanics, alpine-skiing, imu, temporal-segmentation, causal-lstm, subject-disjoint, sensor-fusion
- **论文：** https://ijcai.org/proceedings/2026/0828.pdf
- **正式 DOI：** https://doi.org/10.24963/ijcai.2026/828
- **代码：** https://github.com/mszymocha/SkiC-LSTM （论文脚注指定的官方仓库；GitHub 仓库内容已核验）
- **数据集：** https://github.com/mszymocha/SkiC-LSTM/tree/main/data/turns_labeled （官方仓库包含转弯标注 CSV；已核验 Git tree）
- **项目主页：** https://github.com/mszymocha/SkiC-LSTM （官方统一项目入口，无其他独立主页）
- **论文状态：** IJCAI 2026 正式会议论文；2026 年体育/可穿戴动作分析方法补读
- **主要价值类型：** Method Module / Baseline / Dataset / Related Work
- **阅读优先级：** 中高
- **核验依据：** https://ijcai.org/proceedings/2026/828 与 https://ijcai.org/proceedings/2026/0828.pdf，GitHub 官方源码树

## 一句话总结

SkiC-LSTM 用小腿固定的普通手机 IMU，在 10 Hz 连续输入上以因果多尺度 CNN + 残差 LSTM 逐帧判定滑雪左/右转弯，并通过严格的跨受试者划分和时间 IoU 验证转弯分段的可泛化性。

## 研究问题与动机

- 自然雪场的左右转弯节奏、技能和姿态变化很大，规则式检测和离线处理难直接支撑即时动作反馈。
- 许多视觉/惯性动作识别方法采用双向时间上下文或随机切分，可能产生实时不可用、跨人的特征泄漏与性能虚高。
- 本文将滑雪转弯视为 **每个时间点左/右方向二分类**，而非 ACL 损伤概率预测。直滑或不转弯片段不是单独第三类；论文认为高可信度的转弯相位是后续技术及风险评估的基础。

## 核心方法

1. **输入预处理：** 手机上的 IMU（加速度计、陀螺仪与姿态通道）采样 10 Hz，设备固定在小腿；对角度做 phase unwrapping，使用严格 causal EMA 降噪，仅用当前训练 fold 统计量标准化。
2. **物理启发特征：** 增加加速度、角速度模长、jerk、yaw 一阶/二阶导数和 rolling 统计量；窗口长度 W=12、stride=1，预测窗口末帧。
3. **可学习对齐与多尺度卷积：** 1×1 sensor calibration Conv、四个并行 1D Conv kernel = 1/3/5/11，Squeeze-and-Excitation 通道重权重。
4. **残差因果时序编码：** 两层 128-hidden 单向 LSTM、残差连接、locked dropout 和全连接二分类；因果约束保证模型不读取未来的原始输入。
5. **训练卫生：** Leave-One-Subject-Out（LOSO）按人留出，在训练数据内部按整条 run 划分验证，数据增强仅作用于训练 fold；按单个已标注 turn 做 0.5/0.75/1.25/1.5 时间尺度 warping，并使用密集时间点交叉熵监督。
6. **输出平滑：** 末端 median filter 减少抖动，**会引入约 0.5 s 额外延迟**；因此完整系统是 near-real-time，不能宣传为零延迟逐帧即时结果。

## 数据集与评价指标

- 完整公开数据记录为 **11 名滑雪者、105 条下坡 run、1,781 次转弯**，涵盖初学到专业教练；视频由教练参照标注边界，**推理只输入手机 IMU，不输入视频**。
- LOSO 的最终评估要求受试者至少完成 5 次下坡，实际 **9 名测试受试者、99 条 run、1,656 次转弯**；不能误把完整 11 人/105 runs 等同于模型测试规模。
- 以 10 Hz 采样进行 timestamp-level 的左/右标签；无显式 no-turn 类。报告每滑雪者 Accuracy、每 run Accuracy 与匹配连续转弯段的 per-run IoU（%），并按 carving、quick、skidded、snowplow 与技能等级分层。
- 对比基线：ResNet1D、standard unidirectional LSTM、Random Forest、XGBoost，同样的数据划分与后处理。

## 主要结果

- **Table 2：** SkiC-LSTM 的 per-skier Accuracy **89.77%**、per-run Accuracy **89.85%**、per-run IoU **79.45%**。最接近的 XGBoost 依次为 **88.50% / 88.66% / 78.51%**；Standard LSTM 为 **86.84% / 87.30% / 75.64%**。
- **Table 3：** SkiC-LSTM 的 carving **92.10%**、quick **87.60%**、skidded **89.70%**、snowplow **88.24%**；quick 类的 XGBoost **88.18%**，说明并非所有子条件均最优。
- **Table 4：** 按 Beginner / Intermediate / Advanced / Expert 分组为 **87.07% / 90.99% / 91.53% / 88.11%**。
- **运行效率：** 单核约 3 GHz CPU 的模型前向推理约 **0.1 ms/帧**，但 median filter 带来约 **0.5 秒** 延迟；两者是不同时间指标。

## 优点

- 在自然雪场采集真实参与者而非封闭室内演示，教练依据同步视频给出相位真值。
- 严格 LOSO + run-disjoint 验证和 train-only 增强，减少强相关滑雪片段造成的数据泄漏。
- 同时报告帧分类和连续事件重合度，比只看整体 classification accuracy 更能展示边界漂移。
- 公开的 GitHub tree 可核实时间序列数据与标注文件，利于建立低成本、不同受试者可复现 baseline。

## 局限

- 全部仅 **11 人**，实际 LOSO 统计只纳入 **9 人**；仅两名 beginner、snowplow 仅 10 次下坡且多来自一人，群体/风格泛化证据有限。
- 手机固定小腿；口袋/胸前等不同位置的方向和信号形态可能发生领域偏移。10 Hz 对高速边界和冲击精度有限。
- 因果网络虽然不看未来数据，输出后处理仍延迟 0.5 秒；报告的 0.1 ms 推理时间不能当成最终系统端到端 latency。
- **未评价 ACL 损伤发生率、真正的膝部负荷、压力/GRF 或技术纠错对伤病的因果效果**；“Ski Safety”是潜在应用，非已完成的临床安全验证。
- **推断：** 二分类没有独立直滑/静止状态，长时间直滑或不同运动项目迁移时可能产生错误的连续转弯段。

## 个人评价

相比追求复杂新 backbone，此工作最值得学习的是运动事件/相位问题定义、专家视频对齐和 subject-disjoint protocol。IoU 对临界时间错位敏感，适合与 3D 身体轨迹的转弯相位比较。模型的增益在 Table 2 中相对 XGBoost 幅度不大，应避免将微小百分比优势解读为医疗伤害风险降低。完整论文在时序分类与真伤病预测之间保持了重要的任务区分。

## 与我的研究关联

- **推断（CanonFuse3D）：** 可使用小腿 IMU 作为视觉 3D 运动转弯相位的独立参考标签，测试 side-view / 360 moving-camera 重建的 yaw、重心侧移、骨盆方向和转弯分段时间是否一致，避免以 3D pose 自身推算的标签既训练又评价。
- **推断（滑雪视频）：** 比较当前 wrist-azimuth 固定阈值周期检测、3D skeleton autocorrelation、多尺度 CNN-LSTM 与 IMU 标注/同帧同步对齐；报告 turn boundary MAE、phase F1、temporal IoU 和跨受试者/跨风格差异。
- **推断（ClinicalGated）：** 在临床步态中迁移 causal online event segmentation、LOSO 与 run-level 切分的设计，但临床动作事件应新增合理的非周期/站立/异常类别，不能沿用滑雪的左/右二分类。

## 后续阅读

- 优先阅读 §3（真实雪场数据与教练标签）、§4–5（特征和因果网络）、§6–7 与 Tables 2–5（LOSO、指标、风格泛化）。
- 复现原论文的 XGBoost / plain LSTM / SkiC-LSTM，同样做 10 Hz、W=12、subject-disjoint、0.5 s 后处理设置；另做实时部署去掉 median filter 的分段抖动和延迟对照。
- 对同一滑雪动作视频/IMU 时间轴增加 turn-boundary temporal offset、类别混淆矩阵与实际 injury/force 的独立评价，避免越过论文实际证据。
