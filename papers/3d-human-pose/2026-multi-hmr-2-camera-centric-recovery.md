---
title: "Multi-HMR 2: Multi-Person Camera-Centric Human Detection, Mesh Recovery and Tracking"
authors: "Guénolé Fiche, Philippe Weinzaepfel, Romain Brégier, Fabien Baradel"
venue: "arXiv:2606.14841 (preprint)"
year: 2026
reading_date: 2026-10-26
status: skimmed
tags:
  - 3d-human-pose
  - camera-intrinsics
  - metric-localization
  - multi-person-hmr
  - tracking
  - detection
---

# Multi-HMR 2: Multi-Person Camera-Centric Human Detection, Mesh Recovery and Tracking

## 基本信息

- **作者：** Guénolé Fiche, Philippe Weinzaepfel, Romain Brégier, Fabien Baradel
- **会议/期刊：** arXiv:2606.14841（2026 年预印本；尚未核验正式会议/期刊发表）
- **首次提交：** 2026-06-12；核验版本 arXiv v1
- **年份：** 2026
- **阅读日期：** 2026-10-26
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`、`camera-intrinsics`、`metric-localization`、`multi-person-hmr`、`tracking`
- **论文：** https://arxiv.org/abs/2606.14841
- **DOI：** https://doi.org/10.48550/arXiv.2606.14841
- **代码：** https://github.com/naver/multi-hmr2 （官方已发布推理代码与预训练 Multi-HMR 2.b 模型）
- **数据集：** 暂无（未提出可单独获取的新评测数据集；Anny-One 为相关团队已有训练数据）
- **项目主页：** 暂无（官方 GitHub 仓库承担项目说明）
- **论文状态：** arXiv 预印本；官方代码和 checkpoint 已发布
- **主要价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** A

## 一句话总结

利用 DETR 风格一阶段多人 HMR 同时预测人、相机视场角和相机坐标下的真实尺度人体位置，并通过 SAM2 特征蒸馏支持无需视频训练的身份关联，强调绝对位置评价不能被 root-centered MPJPE 替代。

## 研究问题与动机

过去多人 HMR 主要报告 pelvis-centered PA-MPJPE / MPJPE，忽略相机坐标系的绝对距离和人物间相对位置。固定焦距/FOV 或弱透视假设可能使局部姿态看起来准确，但整个人被放在错误的深度。多人遮挡和近距离交互进一步使单人裁剪或中心点检测不稳。作者希望单张 RGB 图像即可完成检测、身体表征、metric camera-space localization 和后续视频 ID 关联，且无需提供真实相机内参。

## 核心方法

1. **一阶段多人检测与人体重建：** 图像 encoder + DETR decoder，用固定查询预测 detection score、2D 位置、深度/3D translation、Anny parametric mesh 与 track embedding。Anny 支持儿童至成人等不同身体比例。
2. **相机内参：** 图像级预测垂直 FOV，利用透视模型结合人物像素位置和深度恢复相机坐标下的三维位置。这里的 camera prediction 主要指 intrinsics/FOV，**不是视频 SE(3) camera trajectory**。
3. **高效训练匹配：** 训练时基于 2D 人体位置执行 Hungarian matching，避免对全部查询先解码三维网格，并可利用仅有 2D 注释的图像。
4. **无需视频监督的 tracking：** 蒸馏 SAM2 memory encoder 的 per-person 特征；推理时再与位置和人体状态共同做关联。该方案本身不提供独立的时序 world-motion refinement。

## 数据集与评价指标

- 训练使用合成 Anny-One/BEDLAM、带伪真值的人体图像和部分仅有 2D 标注的图像；论文未给出可直接相加的统一独立训练受试者人数。
- 测试：3DPW、EMDB（野外人体重建），Hi4D、Harmony4D、CMU-Panoptic（近距离互动与相对位置），MSCOCO val2017（2D 重投影），PoseTrack21（tracking）。不同测试集样本量不应机械求和；论文未报告统一总测试人数。
- 报告 PA-MPJPE、MPJPE、PVE、TA-PVE、Abs-PVE（mm）、FOV 误差（度）、F1/Recall、IDF1/MOTA 等；3DPW、EMDB 的 TA-PVE/Abs-PVE 强调相机坐标定位误差。

## 主要结果

- **3DPW，预测相机：** Multi-HMR 2.b 的 TA-PVE **161.9 mm**，SAT-HMR（固定相机假设）**174.5 mm**；Multi-HMR 2.a 的 PA-MPJPE **40.5 mm**，AiOS **45.0 mm**。上述 baseline 的 camera 假设不完全相同，需在控制变量实验中比较。
- **EMDB，预测相机：** Multi-HMR 2.a 的 Abs-PVE **273.6 mm**，SAT-HMR **1093.0 mm**，AiOS **11962.1 mm**；在提供真实 intrinsics 时 Multi-HMR 2.a 进一步降至 **129.4 mm**，明确表明 camera prediction 仍有误差。
- **FOV 预测：** Multi-HMR 2.a 在 3DPW / EMDB 的平均角度误差为 **6.4° / 4.0°**（论文 Table 4）。
- **推理：** 作者报告 NVIDIA V100 上未经特殊优化约 4–5 FPS，PyTorch compile 后约 **20 FPS**；条件与正式对照实验需一致。
- 数字来自作者原文 Tables 1、4 及 Appendix C；并非 world-coordinate camera ATE/RPE。

## 优点

- 不再把 root-centered accuracy 作为完整评价，直接比较人体 camera-space metric localization。
- 预测 FOV 可在缺少 GT intrinsics 的场景中使用，官方提供预训练 checkpoint 与视频推理示例，适合建立现实基线。
- 包含多人近距离交互/遮挡与全龄身体形态，问题定义贴近日常视频。

## 局限

- 作者明确将相机畸变建模列为未来工作；极端自拍、fisheye/ERP 不在当前原生适用范围。
- 仅预测单张图像的 intrinsics，没有估计移动相机的世界 SE(3) 轨迹、尺度漂移或 human→camera trajectory feedback。
- **推断：** 如果对每帧独立预测 FOV，时序抖动可能转化为人为深度变化；需要与统一内参或随时间平滑的 camera calibration 对照。
- **推断：** Anny 输出与已有 SMPL/SMPL-X 评测管线存在表征兼容成本；儿童比例虽更合理，但病理形态需要外部验证。

## 个人评价

这篇的亮点是改变评价重心：同样的局部关节精度未必代表 camera-centric 3D 位置准确。测试中真实 intrinsics 与预测 intrinsics 的差异本身就是值得复现的实验。它不是 camera-human mutual BA，不能把“joint camera/body prediction”夸大成“human-to-camera SLAM refinement”。

## 与我的研究关联

- **推断：** 在双 360° 滑雪数据上，将同一模型分别应用于 perspective crops，比较 GT intrinsics、估计 FOV、固定 FOV 与畸变校正后的相机相对 root 深度/尺度偏差；单台 360 同中心裁剪与双台真实 baseline 分开评价。
- **推断：** 用 Multi-HMR 2 作为 human-only camera-centric pose 前端，再接 camera-only SLAM / panorama VO；加入已知物理双相机几何约束后，报告 camera ATE/RPE、scale drift、W-MPJPE、global translation error，并验证是否仍优于 SAM 3D Body/已有 HMR 前端。
- 对临床步态可考察不同身材、患者和拍摄焦距的 metric depth 与失败率，不将局部 MPJPE 等同临床运动学有效性。

## 后续阅读

- 重点读原文 **Sec. 3（Camera intrinsics prediction、matching、tracking）、Sec. 4.1–4.2、Table 1/4、Appendix C**。
- 复现 `predicted FOV vs GT FOV vs fixed FOV` 消融，并分析重投影、root depth 与 camera-space errors。
- 对照 CameraHMR、SAM 3D Body、AnyCam 和 HMR+SLAM 串联系统；对真实 ERP/fisheye 必须单独进行畸变兼容性实验。
- 一手事实来源：https://arxiv.org/html/2606.14841v1 ；https://github.com/naver/multi-hmr2
