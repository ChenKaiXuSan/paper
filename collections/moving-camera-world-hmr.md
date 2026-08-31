# Moving-camera World HMR

## 研究问题

目标是在相机自身运动的情况下，将 camera-relative 人体姿态恢复为稳定、具有尺度意义的 world-coordinate 人体运动，并理解 camera、human 与 scene 三者如何互相约束。

## 方法脉络

1. **Camera estimation / scale**：先获得相机轨迹、内参和 metric scale。
2. **Camera-relative → world**：将 HMR/HPE 输出变换到统一世界坐标。
3. **Global refinement / high-order dynamics**：利用时序、人体尺度、接触、motion prior，以及 velocity / acceleration / jerk 等高阶动力学约束修正漂移与不自然运动。
4. **Joint optimization / representation coupling**：让 human、camera、scene 不再单向串联，而是在尺度、特征或优化层互相校正。

## 关键论文

- [AnyCam](../papers/multiview-geometry/2025-anycam.md) — 从 casual videos 恢复 camera pose 与 intrinsics。
- [VGGT](../papers/multiview-geometry/2025-vggt.md) — 通用视觉几何基础模型，可提供 camera / depth / point-map prior。
- [MASt3R-SLAM](../papers/multiview-geometry/2025-mast3r-slam.md) — 将两视图 3D reconstruction prior 嵌入实时 dense SLAM，同时输出 camera trajectory 与 dense geometry，可作为不利用人体的强 camera/scene baseline。
- [MoRe](../papers/multiview-geometry/2026-more-motion-aware-4d-reconstruction.md) — 显式区分动态物体与 camera motion。
- [WHAC](../papers/global-human-motion/2024-whac.md) — 利用 camera-frame 人体运动恢复 camera metric scale，再用尺度化相机轨迹反向更新 world human trajectory。
- [SHOW](../papers/global-human-motion/2026-show.md) — 将 human semantic / metric-scale priors 注入 geometry backbone，并用 scene geometry 反向约束 SMPL-X，在 shared metric space 中进行 feed-forward 双向 coupling。
- [Humans as Checkerboards](../papers/global-human-motion/2025-humans-as-checkerboards.md) — 用人体接触提供 metric scale。
- [PhysDynPose](../papers/global-human-motion/2025-physics-based-human-pose-moving-camera.md) — moving-camera world HMR + physics refinement。
- [OnlineHMR](../papers/global-human-motion/2026-onlinehmr.md) — 在线 world-grounded human motion。
- [DuoMo](../papers/global-human-motion/2026-duomo.md) — camera-space / world-space 双 motion prior。
- [HTD-Refine](../papers/global-human-motion/2026-htd-refine.md) — 从视频预测 2D keypoints、3D velocity 与 acceleration，以高阶 temporal dynamics 后优化现有 world-HMR；显著改善 jitter、MPJVE、MPJAE 与多数 global motion 指标，但保持 camera trajectory 为上游固定输入。
- [UniSH](../papers/global-human-motion/2026-unish.md) — feed-forward scene-camera-human metric alignment。
- [JOSH](../papers/global-human-motion/2026-josh-joint-optimization.md) — human-scene-camera joint optimization。
- [TROPHIES](../papers/multiview-geometry/2026-trophies.md) — 多视角 human-scene-camera temporal reconstruction。

## 当前共识

单纯把 SLAM/camera estimator 与 HMR 串联容易把尺度、漂移和动态前景误差传递到 world-space human motion。MASt3R-SLAM 说明 foundation-model 3D prior 已经可以在约 15 FPS 下形成同时输出 camera trajectory 与 dense scene geometry 的强 camera-only baseline；WHAC 说明 human motion 可以反向提供 camera metric scale，再由尺度化 camera trajectory 更新 world human motion；SHOW 进一步把这种互约束推进到 feature representation 和 joint training 层，让 human semantic/scale prior 改变 scene geometry，同时让 scene point map 与 camera intrinsics 反向约束 SMPL-X；JOSH 则代表更完整的 optimization-based human-scene-camera joint refinement。

HTD-Refine 又补充了另一个重要维度：world-HMR 的质量不能只看位置误差。即使 W-MPJPE 较低，人体仍可能存在 jitter、foot sliding、速度和加速度不自然的问题。显式利用 video-grounded velocity / acceleration 约束可以大幅改善这些动态指标，并在多数情况下同步改善 global position accuracy。因此 camera trajectory、人体位置和人体高阶运动动态应作为三个相关但不同的评价层分别报告。

## 研究空白

- **推断：**`camera ↔ human ↔ scene` 已有 WHAC 的 scale-level feedback、SHOW 的 feed-forward bidirectional representation coupling 和 JOSH 的 joint optimization 等先例，但针对 camera rotation / translation / scale 与 world human motion 的**长序列、在线、稳定闭环修正**仍较少。
- MASt3R-SLAM 等 camera-only systems 已经把 learned geometry prior 引入实时 trajectory + dense reconstruction，但动态人体通常仍不是用于反向修正 camera 的显式结构约束。
- HTD-Refine 说明 high-order dynamics 能修正 world human motion，但 camera extrinsics 仍作为固定输入；如何让 velocity / acceleration / contact consistency 反向参与 camera correction 仍未充分研究。
- 长距离、高速、低纹理和强旋转场景中的 camera drift 仍是核心问题；SHOW 也明确存在 synthetic-to-real gap，并在 extreme scale / very-small-human 情况下困难。
- 多透视/360 观测如何用于反向修正物理相机 trajectory 尚未形成成熟 benchmark。
- Feed-forward human-scene consistency 或 high-order dynamics 改善是否会同步降低 camera ATE/RPE，目前仍需要显式验证。
- MASt3R 的 prior 目前主要基于 pinhole training；高畸变 fisheye / ERP 输入下如何与 360-specific camera estimation 结合仍需单独评估。

## 与我的研究关系

该 collection 可直接支撑 moving-camera 3D human reconstruction 的 Related Work 与 baseline 设计，特别适合组织 `GT camera / camera-only SLAM+geometry / human-assisted camera scale / human-aware geometry / human-assisted camera pose / joint or recurrent refinement / high-order motion refinement` 的递进实验。MASt3R-SLAM 可作为 perspective / rectified-view 下的 camera+scene baseline，与 panorama-level ViPE / 360DVO 并列，再统一测试 human constraints 的增益；HTD-Refine 则适合作为 world-HMR 之后的 dynamics refinement baseline。

## 下一步阅读 / 实验

- 比较 MASt3R-SLAM、ViPE、360DVO、AnyCam 在同一长序列移动相机数据上的 ATE/RPE、scale drift、dense geometry 与 runtime。
- 同时报告 ATE/RPE、W-MPJPE/RTE 与 MPJVE/MPJAE/Jitter/FS，检查 camera、global pose 与 motion dynamics 是否同步改善。
- 测试人体 2D/3D reprojection、尺度、contact、mask/DensePose prompt、velocity / acceleration 与跨视角一致性对 camera correction 的独立贡献。
- 在长序列滑雪/360 场景中比较 camera-only MASt3R-SLAM、feed-forward SHOW-style coupling、recurrent/optimization camera correction，以及 HTD-style high-order dynamics refinement。
- 对 perspective crop、fisheye rectification 与 ERP 做 distortion sensitivity 分析，明确 pinhole-trained 3D prior 与 360-specific SLAM 的适用边界。
