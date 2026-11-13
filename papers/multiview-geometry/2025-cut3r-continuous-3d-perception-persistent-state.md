---
title: "Continuous 3D Perception Model with Persistent State"
authors: "Qianqian Wang, Yifei Zhang, Aleksander Holynski, Alexei A. Efros, Angjoo Kanazawa"
venue: "CVPR 2025 (Oral), pp. 10510–10522"
year: 2025
reading_date: 2026-11-13
status: skimmed
tags:
  - moving-camera
  - camera-geometry
  - online-reconstruction
  - stateful-recurrent
  - pointmap
  - cut3r
---

# Continuous 3D Perception Model with Persistent State

## 基本信息

- **作者：** Qianqian Wang, Yifei Zhang, Aleksander Holynski, Alexei A. Efros, Angjoo Kanazawa
- **会议/期刊：** CVPR 2025（Oral），pp. 10510–10522；arXiv 首次提交 2025-01-21
- **年份：** 2025
- **阅读日期：** 2026-11-13
- **阅读状态：** `skimmed`
- **标签：** `moving-camera`, `camera-geometry`, `online-reconstruction`, `stateful-recurrent`, `pointmap`, `cut3r`
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高（2025 年重要方法补读）
- **论文：** https://arxiv.org/abs/2501.12387
- **DOI：** https://doi.org/10.1109/CVPR52734.2025.00983
- **代码：** https://github.com/CUT3R/CUT3R
- **数据集：** 暂无（没有单独发布的 CUT3R 专属数据集；训练和评测使用已有数据集）
- **项目主页：** https://cut3r.github.io/
- **正式会议页面：** https://openaccess.thecvf.com/content/CVPR2025/html/Wang_Continuous_3D_Perception_Model_with_Persistent_State_CVPR_2025_paper.html
- **核验来源：** https://arxiv.org/html/2501.12387v1 （原文、方法及实验表格）；上述作者官方代码和项目页面
- **论文状态：** 正式发表会议论文

## 一句话总结

CUT3R 使用持续更新的 recurrent persistent state，把单张图像、视频与无序视图统一为在线的相机参数、世界系 dense pointmaps 和深度预测，是 moving-camera world HMR 可复用的 camera-only 几何前端，而非人体辅助相机优化方案。

## 研究问题与动机

SfM、SLAM 和多视图优化通常为当前场景单独求解，在稀疏观测、视角退化或动态场景中不稳定；离线全局匹配也不适合持续输入的长视频。CUT3R 试图将先验知识和观测记忆结合，在新帧到达时更新场景状态，不依赖提供已知的相机内外参。

## 核心方法

1. **持久状态：** 对流式 RGB 帧使用视觉 Transformer 编码，将新的视觉 token 与固定数量的 learned scene-state tokens 交互；recurrent state update 用于积累跨时刻信息。
2. **读取及预测：** state readout 分支输出 world-frame 与 camera-frame pointmaps、置信度，并由相机参数头预测内参与外参；各帧输出保持共享参考坐标。
3. **未观测区域查询：** 使用 virtual raymap 作为查询，从状态中预测未曾观测的视图几何及颜色（属于生成/补全，并非真实观测 GT）。
4. **监督与训练：** 利用 32 个真实/合成数据源，渐进增加图像分辨率与序列长度，最终训练最长 64 帧的序列；以可获取的几何与相机监督联合训练。

**边界：** 在线重建与相机估计共享状态，不等于把人体关键点、骨长、接触或人体 motion prior 显式反向作用于相机轨迹。方法也不原生采用 ERP 球面成像模型。

## 数据集与评价指标

- **训练：** 32 个真实及合成数据源；实验论文没有统一的“人/场景/帧”总量，不能将其误记为单个标注数据集。
- **Camera pose：** Sintel、TUM-dynamic、ScanNet，使用轨迹对齐后的绝对轨迹误差 ATE、relative pose 的 translation/rotation RPE。
- **Dense reconstruction：** 7-Scenes、NRGBD，以点云/几何 Accuracy、Completeness 和相关距离评价。
- **Depth 与速度：** Sintel、Bonn、KITTI，采用 Abs Rel、δ 等深度指标；KITTI 512×144 输入下报告 FPS。
- **输入/输出：** 输入一帧接一帧的 RGB，输出相机 intrinsics/extrinsics、world/camera pointmaps、depth/confidence；未输出人体 SMPL、骨架或世界坐标人体轨迹。

## 主要结果

- **Table 3 Camera pose：** CUT3R 在线 Sintel ATE **0.213**，在线 Spann3R **0.329**；TUM-dynamic ATE **0.046** 对 Spann3R **0.056**；ScanNet ATE **0.099** 对 Spann3R **0.096**（后者略好）。离线 MonST3R-GA 在 Sintel 为 **0.111**，表明 CUT3R 不全面领先离线方法。
- **重建：** 7-Scenes 重建的 Accuracy 为 **0.126**（DUSt3R-GA **0.146**、Spann3R **0.298**）；该指标不是人体 MPJPE。
- **运行速度：** KITTI 视频深度基准 A100、512×144 配置约 **16.58 FPS**，与输入分辨率及硬件绑定；不是 360° 全分辨率测量。
- **限制性对照：** KITTI 深度某些指标并非全面优于 MASt3R；与较强 offline/global-alignment baseline 需区分在线/离线条件。

## 优点

- 同一逐帧状态同时预测相机与场景，适合持续视频并减少每段独立初始化的开销。
- 与 Spann3R 以及离线优化基线区分明确；提供相机 ATE/RPE、深度、场景几何和运行速度的多个评价层。
- 未知内参、动态内容与未观测区域推断提供了更广的实验方向，官方代码可查。

## 局限

- 作者指出长期视频可能积累漂移，缺少明确 global loop-closure / bundle-adjustment 纠偏机制。
- 虚拟未观测视角输出可能模糊；不能被当作已测量的可靠深度。
- 方法未在双移动 360° 物理相机、低纹理雪地或人体世界运动 GT 上系统评价；当前数值不能据此推广。
- 没有显式 human→camera feedback、接触/骨长或球面投影，不能直接作为 camera-human joint refinement 的实验结论。
- 训练多个数据域且含合成数据；性能受训练语料及 state 时间跨度影响（推断）。

## 个人评价

相对于直接将 HMR 接到经典 SLAM，CUT3R 提供了一个强的 foundation geometry camera-only baseline。需要特别注意：相机 ATE 经过轨迹对齐，不能和未经相同 alignment 处理的 metric 轨迹直接比较；在线方法相对于离线方法也不能只看某个数值的高低。

## 与我的研究关联

**推断：**对于双 360° 滑雪的 world-coordinate 3D human reconstruction，可以先基于 CUT3R 在固定 FoV 的 perspective crops 中获取相机/场景初始化，再设计 shared physical camera 的球面一致性；禁止把同一物理中心的多 crop 误当成立体基线。以 CUT3R 的输出构建 camera-only vs. `+human reprojection` vs. `+bone-length/temporal` vs. `+dual-view consistency`，验证 fused human 证据能否真正降低两台物理相机的 ATE/RPE 与 drift，而不仅降低 3D pose MPJPE。评估长视频 state window、低纹理雪地、相机快速转动与失配观测。

## 后续阅读

- **优先阅读：** Sections 3.1–3.4、4.2、4.4、Table 3、Figure 3、Limitations。
- 复现官方相机 pose 评测与 Sim(3) 对齐协议，分开汇报旋转、平移与 scale drift。
- 对比 [MASt3R-SLAM](../multiview-geometry/2025-mast3r-slam.md)、[ViPE](../multiview-geometry/2025-vipe.md)、[IM360](../360-vision/2025-im360-spherical-sfm-mapping.md) 和人体约束版本。
- 核实基于 crop 的内参回投与 fisheye/ERP 物理模型差异，测试 long-sequence truncation 或 loop closure。
