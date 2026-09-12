---
title: "World-Grounded Human Motion Recovery via Gravity-View Coordinates"
authors: "Zehong Shen, Huaijin Pi, Yan Xia, Zhi Cen, Sida Peng, Zechen Hu, Hujun Bao, Ruizhen Hu, Xiaowei Zhou"
venue: "SIGGRAPH Asia 2024 Conference Papers"
year: 2024
reading_date: 2026-09-13
status: skimmed
tags:
  - global-human-motion
  - moving-camera
  - world-hmr
  - gravity-view
  - temporal-modeling
---

# World-Grounded Human Motion Recovery via Gravity-View Coordinates

## 基本信息

- **作者：** Zehong Shen, Huaijin Pi, Yan Xia, Zhi Cen, Sida Peng, Zechen Hu, Hujun Bao, Ruizhen Hu, Xiaowei Zhou
- **会议/期刊：** SIGGRAPH Asia 2024 Conference Papers
- **年份：** 2024
- **阅读日期：** 2026-09-13
- **阅读状态：** `skimmed`
- **标签：** `global-human-motion`, `moving-camera`, `world-hmr`, `gravity-view`, `temporal-modeling`
- **论文：** https://arxiv.org/abs/2409.06662
- **正式 DOI：** https://doi.org/10.1145/3680528.3687565
- **代码：** https://github.com/zju3dv/GVHMR
- **数据集：** 暂无
- **项目主页：** https://zju3dv.github.io/gvhmr/
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** A+（高）

## 一句话总结

GVHMR 用由世界重力方向与相机视向共同定义的 Gravity-View（GV）坐标系重写 world-grounded HMR，使每帧人体朝向可并行预测，再借助相对相机旋转恢复统一世界轨迹，从而显著减轻长序列自回归累积误差。

## 研究问题与动机

World-grounded human motion recovery 不仅需要恢复相机坐标下的人体姿态，还要在重力一致的世界坐标中得到连续朝向与根部轨迹。传统做法要么直接把 camera-space HMR 通过 SLAM 变换到世界坐标，要么像 WHAM 一样自回归预测相对全局运动；前者容易继承 camera drift 和尺度误差，后者在长序列上会累积方向与位移误差。

GVHMR 的核心观察是：完整 world coordinate 在水平面内存在任意 yaw 旋转自由度，但“重力 + 当前相机视向”可以为每一帧定义一个唯一且自然的局部参考系。作者因此不直接学习歧义较大的绝对 world orientation，而是在 GV 坐标中预测人体朝向，再只利用相邻帧相机相对旋转恢复各 GV frame 之间绕重力轴的一维相对旋转。

## 核心方法

### Gravity-View 坐标表示

GV 坐标的 y 轴与重力方向一致，x 轴由重力与 camera view direction 的叉积确定，z 轴按右手系得到。网络预测每帧人体在 GV 坐标下的 global orientation，以及 SMPL 坐标中的 root velocity。对于 moving camera，通过 DPVO 或 gyroscope 提供的相邻帧相对 camera rotation 计算相邻 GV frame 的相对 yaw，再累积到首帧的 GV 坐标，从而得到统一的 gravity-aware world orientation；root velocity 经世界朝向旋转后积分得到 global translation。

### Relative Transformer 与长序列建模

输入预处理包括人体 bbox、ViTPose 2D keypoints、图像特征以及相对 camera rotation。不同特征先映射并融合成逐帧 token，再进入带 RoPE 的 12-layer Relative Transformer。推理时使用受限 receptive-field attention mask，使模型可以在训练长度之外处理更长序列，而不依赖自回归或复杂 sliding window。

### Stationary-joint 后处理

模型额外预测手、脚趾和脚跟的 stationary probability，用静止关节约束修正 global translation，并通过 IK 调整局部姿态以减轻 foot sliding。该设计把全局位置精度与运动物理合理性同时纳入输出，而不是只优化逐帧 MPJPE。

## 数据集与评价指标

- **训练：** AMASS、BEDLAM、Human3.6M、3DPW；官方发布模型使用 2×RTX 4090 训练 420 epochs。
- **RICH：** world-grounded evaluation，191 个视频，约 59.1 min，主要为静态相机场景。
- **EMDB-2：** 25 个 sequence，约 24.0 min，重点评价 moving-camera 下的 world motion。
- **EMDB-1 / 3DPW：** 用于 camera-space HMR 评价；3DPW test 包含 37 个 sequence、约 22.3 min。
- **World-grounded 指标：** WA-MPJPE100、W-MPJPE100、RTE、Jitter、Foot-Sliding。
- **Camera-space 指标：** PA-MPJPE、MPJPE、PVE、Accel。

## 主要结果

在 RICH 上，使用 DPVO relative rotations 的 GVHMR 达到 **78.8 mm WA-MPJPE100、126.3 mm W-MPJPE100、2.4% RTE、12.8 m/s³ Jitter、3.0 mm Foot-Sliding**；对应 WHAM 为 109.9 / 184.6 mm、4.1%、19.7 m/s³、3.3 mm。

在 moving-camera 的 EMDB-2 上，GVHMR（DPVO）达到 **111.0 mm WA-MPJPE100、276.5 mm W-MPJPE100、2.0% RTE、16.7 m/s³ Jitter、3.5 mm Foot-Sliding**；WHAM（DPVO）为 135.6 / 354.8 mm、6.0%、22.5 m/s³、4.4 mm。将 DPVO 换成 GT gyroscope 后，GVHMR 的 W-MPJPE 只改善约 1.6 mm、RTE 约 0.1%，说明其 GV 表示对 camera rotation estimation error 较稳健。

Camera-space 结果中，3DPW 的 PA-MPJPE / MPJPE / PVE 为 **36.2 / 55.6 / 67.2 mm**，RICH 为 39.5 / 66.0 / 74.4 mm，EMDB 为 42.7 / 72.6 / 84.2 mm。官方项目页报告核心网络在 RTX 4090 上处理 1430 帧、约 45 秒视频仅需 **280 ms**，但该数字不包含检测、特征提取与 VO 等预处理。

## 优点

- 用坐标表示本身降低 world orientation 的学习歧义，而不是单纯堆叠更复杂的后优化。
- RoPE + limited attention 使长序列推理不依赖自回归，可明显减轻 error accumulation。
- 同时报告 global position、relative trajectory、jitter 与 foot sliding，比只看 MPJPE 更适合 world-motion 研究。
- 对 DPVO 与 GT gyroscope 的差距较小，说明方法对相机旋转噪声具有较好的工程鲁棒性。
- 官方代码、权重与复现实验配置已公开。

## 局限

- Camera relative rotation 仍然是上游输入；方法主要解决 `camera rotation → human world motion` 的稳健利用，并没有让人体 residual 反向更新 camera trajectory。
- Global translation 主要由预测的人体 root velocity 积分获得，不能替代独立的 metric camera trajectory / scene reconstruction。
- 没有针对 fisheye、ERP、360° projection、未知或变化 intrinsics 的系统实验。
- EMDB-2 虽包含 moving camera，但与长距离、高速户外滑雪中的弱纹理、强旋转和长期 scale drift 仍有明显 domain gap。

## 个人评价

这是当前 world-HMR 研究中非常值得补齐的经典强 baseline。它最重要的价值并不是“又一个更准的 HMR backbone”，而是说明坐标系设计可以显著改变 global-motion learning 的难度。对于后续 camera-human mutual refinement，GVHMR 可作为一个清晰的 `camera rotation 已给定，但 world human recovery 很强` 的参照点。

**推断：**若新方法使用人体证据反向修正 camera rotation / translation / scale，应在 GVHMR-style gravity-aware human representation 上继续比较，才能区分增益究竟来自更好的人体坐标表示，还是来自真正的 human→camera correction。

## 与我的研究关联

对 moving-camera / dual-360 skiing，最直接的迁移是把 Gravity-View 表示作为人体 world orientation 的中间层，并分别研究 camera orientation、translation 与 metric scale。可设计：

`camera-only 360 VO/VIO → GVHMR-style gravity-view world HMR → + dual-view human fusion → + human-assisted camera correction → joint refinement`。

**推断：**360 相机天然具有大 FoV，camera view direction 的定义与 pinhole 不完全一致，因此可以比较 ERP 主视向、当前人体所在 tangent-view 视向、以及 IMU gravity + physical-camera orientation 三种 GV 构造方式。评价应同时报告 W-MPJPE/RTE/Jitter/Foot-Sliding 与 camera ATE/RPE/scale drift。

## 后续阅读

- WHAM：GVHMR 的主要 world-HMR baseline，重点比较自回归 global motion 与 GV parallel prediction。
- WATCH：进一步把 camera local velocity / translation cue 引入 learned world-motion recovery。
- WHAC / SynCHMR / BodySLAM++：用于比较 human-assisted scale、human-aware camera/scene estimation 与显式 camera-human factor graph。
- 在 dual-360 数据上复现 GV representation，并测试 camera rotation noise、长期 yaw drift、IMU gravity 与 spherical projection 的影响。
