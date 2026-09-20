---
title: "H-Flow: Self-supervised Human Scene Flow via Physics-inspired Joint Multi-modal Learning"
authors: "Zhanbo Huang, Xiaoming Liu, Yu Kong"
venue: "arXiv preprint arXiv:2605.22629 (Under Review)"
year: 2026
reading_date: 2026-09-20
status: skimmed
tags:
  - human-motion
  - scene-flow
  - self-supervised-learning
  - physics-informed
  - biomechanics
  - multimodal-learning
  - dense-motion
---

# H-Flow: Self-supervised Human Scene Flow via Physics-inspired Joint Multi-modal Learning

## 基本信息

- **作者：** Zhanbo Huang, Xiaoming Liu, Yu Kong
- **会议/期刊：** arXiv preprint arXiv:2605.22629；作者项目页标注 `Under Review`
- **年份：** 2026
- **阅读日期：** 2026-09-20
- **阅读状态：** `skimmed`
- **标签：** `human-motion`, `scene-flow`, `self-supervised-learning`, `physics-informed`, `biomechanics`, `multimodal-learning`, `dense-motion`
- **论文：** https://arxiv.org/abs/2605.22629
- **DOI：** https://doi.org/10.48550/arXiv.2605.22629
- **代码：** 暂无（作者说明将在正式发表后释放）
- **数据集：** 暂无公开下载（DynAct4D 已在论文中提出，作者说明将在正式发表后释放）
- **项目主页：** https://actionlab-cv.github.io/H-Flow/

## 一句话总结

H-Flow 将 dense human scene flow、3D pose、depth、human mask 与 camera 放入一个共享 Transformer，通过 silhouette、skeleton-surface、center-of-mass support 和 minimum-jerk 等物理先验进行无 flow GT 的跨模态自监督，试图把骨架运动与衣物/软组织的非刚性表面运动统一到同一 dense motion representation 中。

## 研究问题与动机

SMPL/SMPL-X 等 parametric human model 擅长表示骨骼驱动的全局姿态，但其 LBS 表面无法完整刻画宽松衣物、软组织和其他偏离 skeleton 的局部动态；通用 3D scene flow 虽能为每个点预测运动，却常依赖局部刚体假设，在高度 articulated 的人体上失效。更现实的困难是，真实 clothed human 几乎无法获得 dense per-pixel 3D flow ground truth。

H-Flow 因此把 pose、depth 与 dense flow 视为同一个物理系统的不同投影，不直接依赖 flow label，而是让这些模态通过人体几何、骨骼结构和生物力学规律互相监督。作者同时构建 DynAct4D 作为 test-only synthetic benchmark，为 clothed human 提供精确 dense 4D flow GT。

## 核心方法

网络以 monocular RGB video 为输入，在 frozen DINOv3 backbone 上建立共享 Transformer。pose query 与 camera query 参与同一 attention，DPT-style dense decoder 从共享 patch features 输出 depth、scene flow 与 human mask。每帧联合预测：

- dense depth `D`；
- forward 3D scene flow `F`；
- soft human mask `M`；
- 3D skeletal joints `P`；
- camera intrinsics / extrinsics `C`。

训练的关键不是 dense flow supervision，而是四类 differentiable physical constraints：

1. **Silhouette Edge Alignment**：让 depth/flow 的边缘响应与 human mask boundary 对齐。
2. **Skeletal-Surface Coupling**：将每个前景 3D surface point 的 flow 与最近 bone 诱导的 joint motion 绑定，并用随 skin-to-bone distance 增大的 tolerance 允许衣物独立变形。
3. **Center-of-Mass Support**：根据解剖质量比例计算 CoM，使其投影位于由接触端点组成的 support polygon 内，同时联动 pose 与 ground/depth。
4. **Minimum Effort Path**：通过 inverse/forward kinematics 构造 minimum-jerk 参考轨迹，约束中间姿态的多帧运动。

此外，作者使用 SAM 3D Body 与 Depth Pro 作为 margin-based distillation anchors，为本身 scale-invariant 的物理约束注入 metric anthropometric scale，并加入 single-camera consistency。

## 数据集与评价指标

- **Human3.6M**：使用全部 7 名受试者进行 pretraining；不在该数据集上测试。
- **Fit3D**：S3/S4/S5/S7/S8/S9 训练，S10/S11 测试；用于 in-domain dense scene-flow、pose 与 depth 评价。
- **3DPW**：使用 official split；仅用于 pose 与 depth，因为其 SMPL 标注不适合作为 clothed-surface dense flow GT。
- **DynAct4D**：10 个 MetaHuman characters × 10 套 garments × 10 类 actions 构成候选组合，采样 100 个 motion sequences；每个 motion 从 8 个同步视角、3 个环境渲染，最终得到 **800 个 test sequences、约 720K frames、1920×1080、60 fps**。该 benchmark 完全作为 zero-shot test set。
- 训练为 `4×10^5` iterations，使用 6×NVIDIA H100。
- 主要指标：scene-flow EPE、1-Cos、Acc.S/Acc.R；pose MPJPE/PA-MPJPE；depth MAE/SiLog。

## 主要结果

Dense human scene flow 上，H-Flow 在 Fit3D 的 **EPE 为 24.2 mm**，优于最近的 Depth Pro + H-MoRe 的 38.3 mm；在从未见过的 DynAct4D 上 zero-shot **EPE 为 28.2 mm**，而 Depth Pro + H-MoRe 为 42.2 mm。通用 ZeroMSF 与 Self-Mono-SF 在 Fit3D 仍为 59.9 / 50.2 mm，在 DynAct4D 为 78.5 / 68.0 mm。

Pose 作为 companion task，在 3DPW 达到 **91.5 mm MPJPE / 48.7 mm PA-MPJPE**，Fit3D 为 **112.8 / 55.4 mm**；绝对 MPJPE 优于 SAM 3D Body、CameraHMR 与 PromptHMR，但 PA-MPJPE 并非所有设置下最佳。Depth 在 Fit3D body region 的 MAE 为 **124.5 mm**，相比 Depth Pro 的 234.3 mm 降低约 47%。

消融进一步显示跨模态物理约束确实影响不同输出：去掉 skeletal-surface constraint 后 Fit3D/DynAct4D EPE 从 24.2/28.2 恶化到 **48.4/63.6 mm**；去掉 CoM support 后 3DPW MPJPE 从 91.5 变为 **104.6 mm**；去掉 minimum-effort path 后 Fit3D flow EPE 增至 **42.7 mm**。去掉 distillation anchor 后绝对 MPJPE 达 **183.3 mm**、Fit3D depth MAE 达 **276.8 mm**，说明 metric scale 仍高度依赖 teacher anchor。

## 优点

- 把传统 skeleton/HMR 与 dense scene flow 两条路线连接起来，显式建模 skeleton 不能表达的衣物与软组织动态。
- 物理先验不是后处理，而是让 pose、depth、flow 在共享表示与 loss 两个层面互相约束。
- Skeletal-surface、CoM support 和 minimum-jerk 三类约束具有明确的人体运动解释，消融也显示它们分别影响 flow、pose 与 temporal dynamics。
- DynAct4D 通过 renderer 直接跟踪高分辨率 clothed mesh vertices，提供现有真实数据难以获得的 dense 3D flow GT。
- 在 zero-shot DynAct4D 上仍保持较低 EPE，说明至少在 synthetic domain shift 下物理约束具有一定可迁移性。

## 局限

- 截至阅读日期，代码、模型和 DynAct4D 均尚未公开；作者仅说明将在正式发表后释放，复现暂时受限。
- 论文状态仍为 `Under Review` / arXiv preprint，尚无正式同行评审 venue。
- DynAct4D 是 Unreal Engine / MetaHuman synthetic benchmark，真实宽松服装、遮挡、运动模糊和户外高速动作仍存在明显 domain gap。
- 四个核心物理约束本身是 scale-invariant；绝对 metric scale 仍依赖 SAM 3D Body 与 Depth Pro 的 noisy teacher distillation。作者明确把 fully self-supervised anthropometric grounding 列为开放问题。
- 网络虽然输出 camera，但论文没有把 camera trajectory ATE/RPE 作为核心评价；因此不能从当前结果推断其已经解决 moving-camera world HMR 或 camera-human mutual refinement。
- 训练开销较高：论文报告使用 6×H100、约 400K iterations。

## 个人评价

H-Flow 的价值不在于替代 HMR，而在于提出一个比 joint trajectory 更密集的 **human motion representation**：同一人体同时拥有骨架运动和表面级 3D motion。对仅依赖 keypoints/SMPL 的动作理解而言，这可能补充衣物、软组织和局部高速运动信息；而对体育/临床分析，其 CoM support 与 minimum-jerk losses 也提供了可直接拆出来验证的物理模块。

**推断：**当前最值得关注的是“cross-modal physical constraint 是否真的优于把 pose/depth/flow 独立预测后再融合”。Table 4 已给出初步证据，但仍应在真实 clinical/sports 数据上用 gait phase、joint angle、contact、CoM 或外部 biomechanics GT 做 downstream validation，而不能仅凭 flow EPE 推断临床或运动学价值。

## 与我的研究关联

**推断：**在临床 gait、体育视频和周期运动研究中，可以把 H-Flow 作为 `RGB/optical flow/keypoints/SMPL` 之外的第五种 motion representation：使用 dense 3D human flow 描述局部表面运动，再与 gait phase、trunk/pelvis motion、joint angle、CoM、contact 与 clinical labels 融合。尤其值得测试 `skeleton-only → + optical flow → + dense 3D human flow → + biomechanics priors` 是否能在 ASD gait 或高速滑雪动作中带来可解释增益。

对 moving-camera / 360° 方向，H-Flow 的 pose-depth-flow-camera shared state 也有启发，但当前证据不足以证明 human flow 能反向降低 camera ATE/RPE。**推断：**可以把其 silhouette / skeletal-surface / CoM/contact constraints 改写为 spherical/fisheye geometry，并检查这些 human-centric residual 是否能真正修正 dual-360 camera state。建议重点阅读 Sec. 3.2、Table 2–4 与 distillation-scale ablation。

## 后续阅读

- [Natural Human Motion Recovery by Aligning High-Order Temporal Dynamics from Monocular Videos](../global-human-motion/2026-htd-refine.md) — 比较 velocity / acceleration 高阶动态与 dense human flow 的互补性。
- [Biomechanical 3D Body: Self-Supervised Distillation of Biomechanical Pose from a 3D Body Foundation Model](../medical-ai/2026-biomechanical-3d-body.md) — 对比 foundation body prior + biomechanics distillation 的训练范式。
- [Universal Skeleton Understanding via Differentiable Rendering and MLLMs](2026-skeletonllm.md) — 对比 skeleton representation 与 dense surface-motion representation 在通用 motion understanding 中的可迁移性。
