---
title: "ScaleCurve: From Constant Scale to Consistent Motion for World-Space Human Mesh Recovery"
authors: "Sanghyeon Lee, Jong Taek Lee"
venue: "arXiv"
year: 2026
reading_date: 2026-11-27
status: skimmed
tags:
  - world-hmr
  - camera-scale
  - temporal-calibration
  - human-ground
  - motion-refinement
---

# ScaleCurve: From Constant Scale to Consistent Motion for World-Space Human Mesh Recovery

## 基本信息

- **作者：** Sanghyeon Lee, Jong Taek Lee
- **会议/期刊：** arXiv:2611.34005（预印本，正式 venue 暂无核验）
- **首次提交：** 2026-11-25
- **年份：** 2026
- **阅读日期：** 2026-11-27
- **阅读状态：** `skimmed`
- **标签：** world-hmr、camera-scale、temporal-calibration、human-ground、motion-refinement
- **论文：** https://arxiv.org/abs/2611.34005
- **代码：** 暂无（作者称将公开）
- **数据集：** EMDB：https://eth-ait.github.io/emdb/；SLOPER4D：https://github.com/climbingdaily/SLOPER4D （既有评测数据）
- **项目主页：** 暂无

## 一句话总结

利用人体与局部地面的尺度观测估计时变相机平移尺度，再固定校准相机优化世界坐标人体运动。

## 研究问题与动机

单目 SLAM 的整体尺度校准不足以消除长轨迹中局部尺度失真；即使相机坐标 HMR 准确，世界人体轨迹仍可能漂移。作者用 GT-assisted oracle 区分恒定和时变尺度的影响；oracle 仅用于诊断。

## 核心方法

- 单目 RGB 经 DROID-SLAM 获得任意尺度相机轨迹及点云，经 GVHMR 等获得公制相机坐标人体。关键帧以局部地面平面和预测脚部射线交点距离的比值形成尺度观测。
- 每 1 秒设置一个 log-scale 节点，以 Student-t likelihood 抑制异常残差，random-walk prior 耦合相邻节点，通过 IRLS 求解。
- 对原始相机平移的逐帧增量施加插值后的正尺度并累积；保持相机旋转及位移方向不变，而非逐帧直接缩放绝对位置。
- 固定校准后的相机和体型，用 PVA-Net 的关键点、速度/加速度与 RAFT 表面光流约束人体；L-BFGS 优化姿态、朝向及局部 root translation，同时限制整体人体轨迹重新缩放。所有预训练网络冻结。

## 数据集与评价指标

EMDB-2 全部 25 条序列，SLOPER4D 6 条序列；BEDLAM2.0 training sequences 用于标定尺度噪声参数。EMDB 完整数据库为 10 人/81 序列；SLOPER4D 完整数据为 12 人/15 序列（并非本文测试人数）。指标包括 W-MPJPE、WA-MPJPE（mm）、RTE（%）、相机 ATE（m）、MPJVE（m/s）、MPJAE（m/s²）、jitter、foot sliding。对比 GVHMR、TRAM、HAC、DuoMo、HTD-Refine、HumanBA、JOSH 等；部分结果引用原论文，部分由作者复测。

## 主要结果

- Table 1：EMDB-2 W-MPJPE/WA-MPJPE/RTE 为 154.3 mm / 57.3 mm / 1.0%，DuoMo 为 171.6 / 67.6 / 1.2。
- Table 2：SLOPER4D W-MPJPE 446.8 mm，GVHMR 959.1 mm；JOSH 的 438.3 mm 略好。本文 WA-MPJPE 85.0 mm、RTE 1.5%、foot sliding 16.7 mm。
- Table 3（保持人体 refinement 一致）：恒定尺度 ATE 7.52 m、W-MPJPE 708.12 mm；完整时变校准 4.36 m、446.83 mm。改为高斯平滑时 W-MPJPE 为 503.77 mm。
- Table 4：固定校准相机，去除人体 refinement 时 MPJVE/MPJAE 为 0.95/26.01，完整优化为 0.43/7.12。Table 7 GT-assisted oracle：SLOPER4D 恒定→时变 ATE 7.18→1.89 m，W-MPJPE 745.09→360.32 mm（不是实际推理性能）。
- Table 5：三种人体 backbone（TRAM、Human3R、GVHMR）在共享初始相机条件下均获益。

## 优点

- 分离相机尺度校准与人体动力学 refinement，提供相机及人体的独立指标和受控消融。
- 不需要重新训练人体 backbone；oracle 对照支持时变尺度是独立问题。

## 局限

- 原文 Appendix H：依赖可用地面几何和可靠人体深度，不能修复相机旋转及位移方向误差；只评价单人。
- 推断：滑雪者脚部与雪面之间存在雪板/雪靴，且坡度、遮挡与滑行状态变化；应使用装备接触几何或独立尺度锚点，不能直接等同普通步行。
- 预印本，专用代码尚未核验公开；全序列优化不是已证明的实时方案。

## 个人评价

推断：阅读优先级高；价值类型 Method Module / Baseline / Related Work。时变尺度及其鲁棒优化是可独立比较的模块，但不是完整 camera R/t 双向闭环。

## 与我的研究关联

推断：双 360° 可分别估计两条相机轨迹的 log-scale 曲线，用雪杖长度、已知刚性安装关系、可靠跨视角骨架/场景对应或坡面几何约束尺度；比较恒定 scale、滑窗 scale、鲁棒随机游走 scale 与 full SE(3) refinement。分别报告无尺度对齐的 metric error、ATE/RPE、W-MPJPE 与动态误差；避免以静止脚假设处理滑雪。若采用 TTA，应单独比较仅 scale 更新与人体/相机联合更新。

## 后续阅读

- §3.2–3.3、Tables 1–5、Appendices C/H；先复现 IRLS 及平移增量积分。
- 对照 [Humans as Checkerboards](2025-humans-as-checkerboards.md)、[Fast-camera diagnosis](2026-fastcam-camera-human-scale-mismatch.md)、[HTD-Refine](2026-htd-refine.md)。
- 事实来源：https://arxiv.org/abs/2611.34005 （官方入口提供全文；独立 HTML 入口访问失败）。
