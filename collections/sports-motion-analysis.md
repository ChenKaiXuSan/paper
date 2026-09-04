# Sports Motion Analysis

## 研究问题

研究如何从视频、3D pose、惯性/压力等传感器与生物力学约束中分析体育动作，并进一步生成可量化、可解释的技术评价或反馈。

## 方法脉络

1. 运动姿态与时序表征。
2. 高速动作、遮挡与移动相机下的 3D reconstruction。
3. **Metric body state / CoM estimation**：从 pose、depth、anthropometry 与 physical cues 恢复具有实际尺度的 root / center-of-mass 等中间生物力学量。
4. physics / contact / biomechanics constraints，以及由 pressure / GRF / CoP 等信号验证 kinetics。
5. 运动技术评价、coaching 与自然语言反馈。

## 关键论文

- [GRIP](../papers/sports-biomechanics/2026-grip.md) — sparse IMU + insole pressure + physics-based human motion capture。
- [BadmintonGRF](../papers/sports-biomechanics/2026-badmintongrf.md) — 多视角 RGB + mocap + force plates 的高速体育 GRF benchmark，强调 subject-disjoint 与 impact/load metrics。
- [MuyBridge](../papers/sports-biomechanics/2026-muybridge.md) — 将轻量 2D pose、低频 monocular metric depth、anthropometry、ground-contact 与 ballistic range cue 解析式融合，在手机端恢复运动员 metric segmental CoM；显示 vertical CoM 可较稳定，而 camera-to-athlete range/depth 是绝对 3D 的主要误差源。
- [Imitation Learning from Human Motion Alone Does Not Guarantee Biomechanically Plausible Gait Kinetics](../papers/sports-biomechanics/2026-kinetics-aware-gait-imitation.md) — 证明 motion imitation 的良好 kinematics 不保证真实 GRF/CoP/joint moments；显式 kinetics reward 可显著改善动力学一致性。
- [BioCoach](../papers/sports-biomechanics/2026-biocoach.md) — 3D pose 到 biomechanics-grounded coaching。
- [Action Motifs](../papers/motion-understanding/2026-action-motifs.md) — 可变长度层级运动表征。
- [EventGait](../papers/motion-understanding/2026-eventgait.md) — event streams 下的高速/低照度运动识别思路。
- [PhysDynPose](../papers/global-human-motion/2025-physics-based-human-pose-moving-camera.md) — moving camera + physics refinement。
- [EmbodMocap](../papers/multiview-geometry/2026-embodmocap.md) — 双移动相机 4D human-scene reconstruction。

## 当前共识

体育动作分析需要同时关注局部姿态精度、全局运动、接触与动力学合理性；单纯 PA-MPJPE 很难反映技术动作是否真实、稳定和可解释。进一步地，joint-angle / trajectory reproduction 本身也不能保证 ground reaction force、center of pressure 或 joint moments 合理：motion-only imitation 可以在 kinematics 上接近真人，却在 kinetics 上产生明显偏差。

MuyBridge 又补充了位于 pose 和 kinetics 之间的一个实用层次：**metric center-of-mass trajectory**。体育分析不一定需要先恢复完整 muscle/joint force 才能获得有意义的物理量；利用 pose、metric depth、人体测量学和 contact/range physics，可以先得到更直接用于 balance、translation 与动作阶段分析的 CoM。但其结果也表明，垂直 CoM 误差可以相对较小，而绝对 3D 仍被 camera-to-athlete range/depth 主导，因此 vertical/relative biomechanics 与 global metric localization 应分开评价。

因此，如果研究最终目标包含“负荷”“发力”“冲击”“关节力矩”等生物力学解释，应该把 kinematic fidelity 与 kinetic fidelity 分开评价。GRF、CoP、pressure、contact timing、joint moment 或其他可观测 load proxies 应尽可能作为独立 ground truth / supervision，而不是仅从姿态误差间接推断。同时对于 metric CoM / root trajectory，应额外报告 camera-to-athlete range、scale 或 depth error，避免把深度定位误差误解为人体动作本身的误差。

## 研究空白

- 高速户外体育缺少带 camera trajectory、world human motion、metric CoM 和接触/地形/kinetics GT 的统一 benchmark。
- **推断：**360° 伴随式拍摄可以提高观测连续性，但需要专门处理同中心多透视、动态 camera 和场景尺度。
- MuyBridge 已经证明手机单目可估计 metric CoM，但主要仍依赖一次性 scene calibration、ground/contact/range cue；在 moving-camera、长距离与 airborne sports 中，camera motion 与 athlete range 会进一步耦合，尚缺少直接验证。
- biomechanics 规则与 learned representation 如何结合、并跨运动项目泛化仍不明确。
- 实验室 force plate 很难直接迁移到雪场；boot/insole pressure、IMU、contact timing、ski deformation 等弱动力学信号能否替代或补充 GRF/CoP supervision 仍需验证。
- kinematic accuracy、metric localization / CoM accuracy 与 kinetic plausibility 之间可能存在 trade-off，方法评价需要避免用单一综合误差掩盖这种差异。
- 人体测量学通常使用 population-average segment proportions；个体差异、装备、厚衣服和运动姿态对 CoM 估计偏差的影响在户外体育中需要单独分析。

## 与我的研究关系

用于组织滑雪、体育视频 3D motion reconstruction、physics/contact evaluation 和动作技术解释相关文献。尤其适合把现有滑雪 pipeline 从 `3D pose / world trajectory` 扩展为 `metric root / CoM → contact-aware → pressure/IMU-aware → kinetics-aware`，并明确区分“重建动作准确”“世界尺度定位准确”和“动力学解释可信”三个层次。

**推断：**如果未来采集 boot pressure、insole pressure、IMU 或 RTK，可以把这些信号既作为 world-motion / CoM refinement 的辅助约束，也作为独立的 biomechanics validation，从而检验更低的 W-MPJPE / joint-angle error 是否真的对应更准确的 CoM、load / contact dynamics。双 360 或 moving-camera 提供的多视角 depth/geometry 还可以替代 MuyBridge 中较脆弱的单目 camera-to-athlete range cue。

## 下一步阅读 / 实验

- 增加 skiing / alpine sports 专门文献与数据集。
- 在现有滑雪序列上从 3D skeleton 计算 segmental CoM，报告 vertical CoM、3D CoM、root trajectory、camera-to-athlete range/scale 与 camera ATE/RPE，而不是只报告 MPJPE。
- 复现 MuyBridge 的 `pose-only → + depth → + anthropometry → + contact/range physics` 消融，并进一步加入 multi-view / 360 geometry、IMU 或 RTK 作为 metric cue。
- 报告 joint angle、root trajectory、velocity/acceleration、foot/contact 与 camera metrics，同时独立报告可获得的 pressure / GRF / CoP / load metrics。
- 做 `pose-only → CoM-aware → contact-aware → pressure/force-aware` 的递进 ablation，检查 kinematic、metric localization 与 kinetic 指标是否同步改善。
- 对比 OpenCap Monocular、MuyBridge、BadmintonGRF、GRIP 与 kinetics-aware imitation learning 中 metric/force/contact supervision 的来源、可部署性与误差定义。
- 探索从 3D pose / CoM / biomechanics variables 自动生成技术反馈，但避免在没有 kinetics validation 时直接把 pose-based proxy 解释为真实关节负荷。
