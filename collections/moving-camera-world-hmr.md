# Moving-camera World HMR

## 研究问题

目标是在相机自身运动的情况下，将 camera-relative 人体姿态恢复为稳定、具有尺度意义的 world-coordinate 人体运动，并理解 camera、human 与 scene 三者如何互相约束。

## 方法脉络

1. **Camera estimation / scale**：先获得相机轨迹、内参和 metric scale。
2. **Camera-relative → world**：将 HMR/HPE 输出变换到统一世界坐标。
3. **Unified recurrent human-scene-camera reconstruction**：在一个 persistent state 中同时恢复多人物 SMPL-X、dense scene 与 camera trajectory，减少多阶段流水线的误差传播。
4. **Multi-moving-camera observation fusion**：利用多个动态相机之间互补、间歇且可靠性不同的 ego/exo 观测，提高世界坐标人体运动的覆盖与姿态精度。
5. **Global refinement / high-order dynamics**：利用时序、人体尺度、接触、motion prior，以及 velocity / acceleration / jerk 等高阶动力学约束修正漂移与不自然运动。
6. **Joint optimization / representation coupling**：让 human、camera、scene 不再单向串联，而是在尺度、特征或优化层互相校正。

## 关键论文

- [AnyCam](../papers/multiview-geometry/2025-anycam.md) — 从 casual videos 恢复 camera pose 与 intrinsics。
- [VGGT](../papers/multiview-geometry/2025-vggt.md) — 通用视觉几何基础模型，可提供 camera / depth / point-map prior。
- [MASt3R-SLAM](../papers/multiview-geometry/2025-mast3r-slam.md) — 将两视图 3D reconstruction prior 嵌入实时 dense SLAM，同时输出 camera trajectory 与 dense geometry，可作为不利用人体的强 camera/scene baseline。
- [Human3R](../papers/global-human-motion/2026-human3r.md) — unified all-at-once online recurrent baseline，在同一个 persistent state 中同时预测多人物 SMPL-X、dense scene 与 camera trajectory；约 15 FPS / 8 GB，不依赖预先 detector/SLAM/depth，但 human→camera 的帮助主要是 shared-state / joint-training 层面的隐式互益，而非显式 residual correction。
- [MoRe](../papers/multiview-geometry/2026-more-motion-aware-4d-reconstruction.md) — 显式区分动态物体与 camera motion。
- [Everybody Tracking Every Body](../papers/global-human-motion/2026-everybody-tracking-every-body.md) — 在 2–4 名佩戴者均携带移动 egocentric camera 的场景中，将 VIO 头部/相机世界轨迹与其他佩戴者提供的稀疏 exocentric 人体观测进行可靠性感知 diffusion fusion；证明多 moving-camera evidence 可协同提升 world body motion，但相机轨迹仍作为固定 conditioning 输入。
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

单纯把 SLAM/camera estimator 与 HMR 串联容易把尺度、漂移和动态前景误差传递到 world-space human motion。MASt3R-SLAM 说明 foundation-model 3D prior 已经可以在约 15 FPS 下形成同时输出 camera trajectory 与 dense scene geometry 的强 camera-only baseline；Human3R 则进一步说明 camera、scene 与 multi-person SMPL-X 可以直接放进同一个 online recurrent persistent state 中统一恢复，避免 detector/HMR/SLAM/depth 多阶段串联，并在 generic camera/depth evaluation 中观察到 human prompt tuning 对几何任务的轻微互益。这使“camera-only pipeline → unified shared-state reconstruction → explicit human-assisted camera feedback”成为更清晰的比较层次。

WHAC 说明 human motion 可以反向提供 camera metric scale，再由尺度化 camera trajectory 更新 world human motion；SHOW 进一步把这种互约束推进到 feature representation 和 joint training 层，让 human semantic/scale prior 改变 scene geometry，同时让 scene point map 与 camera intrinsics 反向约束 SMPL-X；JOSH 则代表更完整的 optimization-based human-scene-camera joint refinement。

Everybody Tracking Every Body 补充了一个此前较少被单独讨论的维度：**多个相机本身都在移动时，observer 的人体观测既是稀疏的，也具有不同可靠性。** 它说明持续的 VIO/head trajectory 可以提供全局位置锚点，而来自其他 moving cameras 的 exocentric pose 可补充局部人体姿态；通过 observer-wise temporal encoding、attentional pooling 与 observation dropout，learned fusion 可以比 hard replacement 或单一 source 更稳定。这意味着多移动相机系统不应只做固定权重 pose averaging，而应显式建模可见性、coverage 和 observation confidence。

HTD-Refine 又补充了另一个重要维度：world-HMR 的质量不能只看位置误差。即使 W-MPJPE 较低，人体仍可能存在 jitter、foot sliding、速度和加速度不自然的问题。显式利用 video-grounded velocity / acceleration 约束可以大幅改善这些动态指标，并在多数情况下同步改善 global position accuracy。因此 camera trajectory、人体位置和人体高阶运动动态应作为三个相关但不同的评价层分别报告。

## 研究空白

- **推断：**`camera ↔ human ↔ scene` 已有 Human3R 的 unified recurrent shared-state prediction、WHAC 的 scale-level feedback、SHOW 的 feed-forward bidirectional representation coupling 和 JOSH 的 joint optimization 等先例，但针对 camera rotation / translation / scale 与 world human motion 的**长序列、在线、稳定且可解释的显式闭环修正**仍较少。
- Human3R 虽然将 human、scene 与 camera 放入同一 persistent state，并显示 human prompt tuning 对 camera/depth 有一定隐式互益，但缺少可单独量化的 `human residual → camera rotation/translation/scale correction` 模块；如何区分 shared representation 收益与真正的 human-to-camera feedback 仍值得研究。
- Everybody Tracking Every Body 已经展示多 moving-camera + reliability-aware human fusion，但其 VIO trajectories 仍是固定 conditioning；如何让 fused body evidence、骨长、contact 或跨 observer consistency **反向修正每个移动相机的 trajectory** 仍未解决。
- 现有多移动相机实验中 observation coverage 往往偏高；长时间完全不可见、observer 数量变化、低纹理导致 VIO drift 时，fusion 是否仍能稳定工作需要更有针对性的 benchmark。
- MASt3R-SLAM 等 camera-only systems 已经把 learned geometry prior 引入实时 trajectory + dense reconstruction，但动态人体通常仍不是用于反向修正 camera 的显式结构约束。
- HTD-Refine 说明 high-order dynamics 能修正 world human motion，但 camera extrinsics 仍作为固定输入；如何让 velocity / acceleration / contact consistency 反向参与 camera correction 仍未充分研究。
- 长距离、高速、低纹理和强旋转场景中的 camera drift 仍是核心问题；SHOW 也明确存在 synthetic-to-real gap，并在 extreme scale / very-small-human 情况下困难。
- 多透视/360 观测如何用于反向修正物理相机 trajectory 尚未形成成熟 benchmark。
- Feed-forward human-scene consistency、unified recurrent state、multi-observer fusion 或 high-order dynamics 改善是否会同步降低 camera ATE/RPE，目前仍需要显式验证。
- MASt3R 的 prior 目前主要基于 pinhole training；高畸变 fisheye / ERP 输入下如何与 360-specific camera estimation 结合仍需单独评估。

## 与我的研究关系

该 collection 可直接支撑 moving-camera 3D human reconstruction 的 Related Work 与 baseline 设计，特别适合组织 `GT camera / camera-only SLAM+geometry / independent camera+HMR / Human3R-style unified recurrent state / multi-moving-camera reliability fusion / human-assisted camera scale / human-aware geometry / explicit human-assisted camera pose / joint or recurrent refinement / high-order motion refinement` 的递进实验。MASt3R-SLAM 可作为 perspective / rectified-view 下的 camera+scene baseline，与 panorama-level ViPE / 360DVO 并列；Human3R 则作为不依赖多阶段 preprocessing 的 all-at-once unified baseline，用于检验后续显式 human constraints 是否不仅改善人体，还进一步降低 camera ATE/RPE；Everybody Tracking Every Body 可作为“多动态 observer 如何融合”的方法参考；HTD-Refine 则适合作为 world-HMR 之后的 dynamics refinement baseline。

**推断：**对于双 360° 跟拍，可以将不同物理相机、不同 perspective crops 或不同时间段的有效人体观测视为 reliability-varying observers，同时把 camera confidence 与 human confidence 共同输入 fusion。若 Human3R-style shared state 已能提供稳定初始化，再由人体 reprojection、骨长、contact、velocity 等显式 residual 进一步约束物理相机的 rotation / translation / scale，就可以把“隐式联合表征”与“真正 human→camera feedback”的贡献拆开验证。

## 下一步阅读 / 实验

- 比较 MASt3R-SLAM、ViPE、360DVO、AnyCam 与 Human3R 在同一长序列移动相机数据上的 ATE/RPE、scale drift、dense geometry、W-MPJPE/RTE 与 runtime。
- 增加 `independent camera+HMR → Human3R unified recurrent state → explicit human-assisted camera correction` 实验，检查 shared-state joint prediction 与显式 feedback 的增益是否可累加。
- 增加 `single observer → fixed-weight multi-view fusion → confidence-aware observer fusion → human-assisted camera correction` 递进实验，并按 observer 数量与 visibility/coverage 分层报告性能。
- 人为注入 VIO rotation/translation/scale drift，测试 Everybody Tracking Every Body 风格的 body fusion是否只是吸收 camera error，还是能够进一步用于 camera correction。
- 同时报告 ATE/RPE、W-MPJPE/RTE 与 MPJVE/MPJAE/Jitter/FS，检查 camera、global pose 与 motion dynamics 是否同步改善。
- 测试人体 2D/3D reprojection、尺度、contact、mask/DensePose prompt、velocity / acceleration 与跨视角一致性对 camera correction 的独立贡献。
- 在长序列滑雪/360 场景中比较 camera-only MASt3R-SLAM、Human3R-style unified recurrent reconstruction、reliability-aware multi-observer fusion、feed-forward SHOW-style coupling、recurrent/optimization camera correction，以及 HTD-style high-order dynamics refinement。
- 对 perspective crop、fisheye rectification 与 ERP 做 distortion sensitivity 分析，明确 pinhole-trained 3D prior 与 360-specific SLAM 的适用边界。
