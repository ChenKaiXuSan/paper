---
title: "EgoMo3R: Joint Egocentric Motion and Scene Reconstruction"
authors: "Chaitanya Patel, Yusuke Kato, Rin Takahira, Kazuki Kozuka, Juan Carlos Niebles, Ehsan Adeli"
venue: "NeurIPS 2026（作者项目页标注）；arXiv:2611.01627"
year: 2026
reading_date: 2026-11-06
status: skimmed
tags:
  - global-human-motion
  - moving-camera
  - egocentric-motion
  - camera-human-mutual-refinement
  - scene-reconstruction
  - diffusion-prior
  - camera-trajectory
---

# EgoMo3R: Joint Egocentric Motion and Scene Reconstruction

## 基本信息

- **作者：** Chaitanya Patel, Yusuke Kato, Rin Takahira, Kazuki Kozuka, Juan Carlos Niebles, Ehsan Adeli
- **会议/期刊：** NeurIPS 2026（作者项目页 BibTeX 标注；正式会议卷页尚未独立核验）；arXiv 预印本
- **首次提交：** 2026-11-01（arXiv v1）
- **年份：** 2026
- **阅读日期：** 2026-11-06
- **阅读状态：** skimmed
- **标签：** global-human-motion, moving-camera, egocentric-motion, camera-human-mutual-refinement, scene-reconstruction, diffusion-prior
- **论文：** https://arxiv.org/abs/2611.01627
- **DOI：** https://doi.org/10.48550/arXiv.2611.01627（arXiv DOI；不是会议 DOI）
- **代码：** 暂无（论文说明拟公开，目前未核验到官方代码仓库）
- **数据集：** https://ego-exo4d-data.org/ （EgoExo4D；另使用 Nymeria，官方可用性需按数据授权确认）
- **项目主页：** https://chaitanya100100.github.io/EgoMo3R/
- **主要价值类型：** Method Module / Baseline / Related Work
- **阅读优先级：** 高（最新工作；直接针对 motion→camera 反馈）

## 一句话总结

EgoMo3R 从单目头戴相机视频中交替优化 metric camera trajectory、逐帧深度尺度与 SMPL-X 全身运动，让 motion diffusion prior 通过头部轨迹及地面约束反向校正 camera/scene，而非仅把相机轨迹视为固定输入。

## 研究问题与动机

传统 egocentric body-motion 模型往往依赖头戴设备的 VIO/IMU 提供已知 metric、gravity-aligned 相机轨迹。实际普通 RGB 视频存在快速头部转动、motion blur、单目尺度不确定、深度或长程对应噪声等问题；直接把 SLAM 误差输入运动模型会产生 world-motion 漂移。方法希望从普通视频联合解决 head camera、scene 与 invisible-body motion 的互相依赖。

## 核心方法

1. **输入/输出：** 输入单目 egocentric RGB 视频；输出相机位姿 C、世界坐标 3D pointmaps（逐帧相对深度乘尺度 S）和 SMPL-X 全身运动 M，统一为 metric、gravity-aligned frame。
2. **Low-level geometric priors：** 相对单目深度作为几何基底，optical flow 提供短程匹配，CoTracker3 长程 point tracks 提供跨帧一致性；通过 flow-only warm start 和 MAD 外点筛除稳定相机估计。
3. **Camera/scene refinement：** 固定 M，利用 flow、track、depth、ground-plane scene consistency 优化 C 与 S。
4. **Motion/prior alignment：** 固定 C/S，由基于 SE(3)-invariant camera trajectory 条件的 motion diffusion prior、SMPL-X head-camera alignment 更新 M。
5. **交替优化：** 避免 C、S、M 一起直接梯度下降导致 motion collapse。关键损失包括 flow/track/depth、scene-ground、egocam/head consistency 与 diffusion motion prior；默认报告两轮迭代。详见 Section 3.2–3.5、Table 4–5。

## 数据集与评价指标

- **数据：** EgoExo4D、Nymeria 的 Project Aria egocentric 片段；所有模型输入相同的 rectified pinhole 512×512 视频（焦距 150 px），而非原始鱼眼/ERP。
- **样本量：** 主文说明随机片段与高运动片段混合测试，8 秒为主实验片段长度、附录还有 24 秒测试；主文未明确列出总测试 clip 数，不能自行推断。
- **GT 来源：** Aria MPS 多传感器离线 VIO/SLAM camera pose；Nymeria 自带 motion annotations；EgoExo4D motion reference 使用 UniEgoMotion 的拟合结果（非独立 marker mocap GT）。
- **评价：** camera ATE（metric-scale 或 scale-invariant alignment）、RPE rotation/translation；点云 Accuracy / Completion；global MPJPE（米）、PA-MPJPE、Head Rot/Trans、foot contact（FC）。
- **对比：** VGGT、CUT3R、ViPE、EgoMono4D、EgoEgo，以及 CUT3R/ViPE + EgoAllo / UniEgoMotion。

## 主要结果

- **EgoExo4D：** metric camera ATE 0.226，对比 CUT3R 0.415、ViPE 0.426；metric RPE translation 0.065，对比 ViPE 0.137。**注意**：scale-invariant ATE 0.161，仍差于 ViPE 0.112；不能泛称所有 camera 指标最优（Table 2）。
- **Nymeria：** metric ATE 0.325，对比 CUT3R 0.471；但 ViPE 更低，为 0.276。RPE rotation 为 2.142°、translation 为 0.060（Table 2）。
- **Human motion：** EgoExo4D global MPJPE 0.271 m，对比 ViPE + UEM 0.337 m；Nymeria 0.319 m，对比 ViPE + EgoAllo 0.341 m。PA-MPJPE 并非所有设置最好（Table 3）。
- **双向收益消融：** EgoExo4D locomotion 子集从第 1 轮 camera/scene 优化的 ATE 0.353，经 motion→scene 反馈降低至 0.318；direct joint optimization 的 MPJPE 0.456，交替方案为 0.320（Tables 4–5）。
- **结论边界：** 不仅报告人体 MPJPE，也分别验证 camera ATE、pointmap 与局部姿态；camera accuracy 与 motion accuracy 并不天然同步。

## 优点

- 显式把 human-motion prior 作为 camera/scene 优化反馈，而非简单上游固定 camera 输入。
- 提供与单向 pipeline、直接联合下降和 smooth-camera prior 的清晰消融，展示双向收益。
- 同时评价 metric vs scale-invariant 的 camera 与 3D geometry 指标，避免只报告人体局部误差。

## 局限

- 上游 monocular depth、flow 和 point tracks 仍会在低纹理、动态物体、严重模糊时失效；地面检测失败会削弱 gravity/ground alignment。
- 优化式 inference 较慢，不适合直接当实时 feedforward baseline。
- 研究重点是 egocentric locomotion，手部操控未显式优化；对 third-person 人体可见、双移动 360°/ERP 设定没有直接评价。
- EgoExo4D 的 motion reference 为拟合结果而非独立高精度 GT；MPS camera reference 也依赖多传感器离线估计。
- 官方代码截至本次核验未确认公开；NeurIPS 2026 信息来自作者项目，会议出版 DOI/页码尚待核查。

## 个人评价

**推断：**此研究比单纯 camera-first reconstruction 更接近所需的 human→camera mutual refinement。尤其应保留 Table 2 中 Nymeria 上 ViPE 的 metric ATE 更好，以及 EgoExo4D 上 ViPE 的 scale-invariant ATE 更好这两个反例，防止过度解释。alternating optimization 和 differentiable head-ground constraints 具有模块迁移潜力，但对体外双机位视频，头部轨迹不再等于 camera trajectory，需要把 egocam consistency 换为 2D/3D human reprojection 与跨相机几何约束。

## 与我的研究关联

- **moving-camera world HMR：** 适合作为 camera-human feedback 的近期对照，设计 camera-only → +pose prior → +human→camera → alternating refinement 的消融。
- **双 360° skiing：** **推断：**将 head-camera constraint 改为人体关节球面投影 residual、joint visibility/reliability 和多相机 pose consistency。必须先做 ERP 投影与 gauge/scale 处理，不能直接套用 512×512 pinhole 设置。
- **多视角融合：** 在 camera R/t/scale 与 3D MPJPE 外，增加 RPE、pointmap completeness、ground-contact consistency、velocity/jerk，判断 correction 是真实几何改善还是被运动先验过度平滑。
- **潜在风险：** skiing 含平滑滑行、急转和姿态变化，通用 locomotion prior 可能过度约束高速动作。建议把 motion prior 关闭/开启做对照。

## 后续阅读

- 复查 Section 3.4–3.5、Tables 2–5、Appendix D；尝试复现实验的 flow-only warm start、MAD 与 alternating ablation。
- 将 [PACE](2024-pace.md)、[SynCHMR](2024-synchmr.md)、[WHAC](2024-whac.md) 和 [TRAM](2024-tram-global-trajectory-and-motion.md) 作为不同 camera-human coupling 强度对照。
- 跟踪作者代码是否发布，正式 NeurIPS proceedings / DOI 是否可用。
