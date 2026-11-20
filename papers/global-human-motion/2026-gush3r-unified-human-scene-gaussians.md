---
title: "GUSH3R: Everyone Everywhere All at Once as Gaussians"
authors: "Keito Abe, Kaede Shiohara, Takashi Otonari, Toshihiko Yamasaki"
venue: "arXiv:2607.05243 (作者项目标注 WACV 2027，正式 proceedings 尚未核验)"
year: 2026
reading_date: 2026-11-21
status: skimmed
tags:
  - world-human-motion
  - dynamic-human-scene
  - gaussian-splatting
  - moving-camera
  - streaming-reconstruction
  - camera-geometry
  - human3r
---

# GUSH3R: Everyone Everywhere All at Once as Gaussians

## 基本信息

- **作者：** Keito Abe, Kaede Shiohara, Takashi Otonari, Toshihiko Yamasaki
- **会议/期刊：** arXiv:2607.05243；作者项目页标注 WACV 2027（截至阅读日尚不能作为已正式出版的 proceedings）
- **年份：** 2026
- **提交日期：** 2026-07-06（arXiv 初次提交）
- **阅读日期：** 2026-11-21
- **阅读状态：** `skimmed`
- **标签：** `world-human-motion`, `3d-gaussian-splatting`, `human-scene-camera`, `streaming`
- **论文：** https://arxiv.org/abs/2607.05243
- **DOI：** https://doi.org/10.48550/arXiv.2607.05243
- **代码：** https://github.com/abkeito/GUSH3R
- **数据集：** 暂无（本研究使用已有 NeuMan、EMDB、BEDLAM、MVHumanNet、DL3DV；未确认新数据集发布）
- **项目主页：** https://abkeito.github.io/gush3r-page/
- **论文状态：** 可信预印本，作者项目标注 WACV 2027；正式会议论文页面和出版日期暂无
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高

## 一句话总结

GUSH3R 将 Human3R 的流式 camera/scene/SMPL-X 几何先验解码成共享坐标系下的动态人体与静态场景 3D Gaussians，以可渲染的 4D 表示取代独立人体与场景重建再拼接的方案。

## 研究问题与动机

- 现有实时 feed-forward geometry 系统能够预测 pointmap、相机和人体，但输出未必支持高质量 novel-view synthesis；3DGS 方法在动态人体场景常需逐场景优化。
- 把静态背景 AnySplat 与独立人体 LHM 拼接容易造成遮挡边缘和几何对齐伪影。
- 研究问题是：仅凭移动单相机 RGB 视频，能否因果、逐帧恢复带可渲染外观的 human-scene 表示，并在统一坐标框架中处理动态人体？

## 核心方法

1. **Foundation backbone：** 对输入连续帧运行 Human3R，得到 image/human tokens、场景 pointmaps、camera pose 及 SMPL-X 人体网格。
2. **Scene Gaussian Decoder：** 由几何 pointmap 与 image token，经 Dense Prediction Transformer 预测场景 Gaussian 参数；背景 Gaussians 被持续积累/更新。
3. **Human Gaussian Decoder：** 利用 SMPL-X 顶点几何与 human/image tokens，通过 Human Gaussian Transformer 与 cross-attention 预测随人体运动变化的 Gaussians。
4. **统一渲染：** 将 scene/human Gaussians 放在同一坐标框架，可渲染静态场景与多名运动人体，避免后处理硬拼接。
5. **训练监督：** 场景支路用 BEDLAM+DL3DV 的 RGB/GT depth 和渲染误差；人体支路用 BEDLAM+MVHumanNet 的 silhouette、MSE、局部 LPIPS 等。
6. **推断输入/输出：** 单目 RGB 视频 → 在线场景/人体 Gaussian representation 与跨视角渲染；官方仓库提供 `infer.py` 和权重获取指导（SMPL/SMPL-X 另需许可）。这不是原生 ERP/360 模型。

## 数据集与评价指标

- **训练：** Scene Decoder 在 BEDLAM 与 DL3DV 上训练，Human Decoder 在 BEDLAM 与 MVHumanNet 上训练；每个训练样本抽取四帧。总受试人数/训练片段数在核验的原文相关段落没有统一汇总，不推测。
- **测试：** 单人动态场景 NeuMan、EMDB；多人合成场景 BEDLAM。原文未给出此处全部被测序列的总样本量，标记暂无。
- **评价：** novel-view synthesis 的 PSNR (dB)、SSIM、LPIPS；每帧重建时间；附录另有深度 AbsRel、阈值准确率及长序列 Gaussian 数量/内存。
- **Baselines：** optimization-based HSR；feed-forward AnySplat、AnySplat+LHM+Human3R、AnySplat+LHM+GT Pose。

## 主要结果

- **Table 2, NeuMan：** GUSH3R PSNR=18.2 dB、SSIM=0.57、LPIPS=0.29；独立组合 AnySplat+LHM+Human3R 为 13.9 / 0.32 / 0.46；HSR 的 PSNR 更高，为 20.6 dB。
- **Table 2, EMDB：** GUSH3R 18.0 dB / 0.57 / 0.40；独立组合为 14.7 / 0.48 / 0.49；HSR 的 SSIM 更高，为 0.68。
- **Table 2, 重建时间：** GUSH3R 为 0.54 s/帧；独立组合为 5.5 s/帧，HSR 超过一小时。此表的重建时间**未包含用于目标帧人体姿态估计的额外 Human3R pass**，不可与严格端到端运行时间混淆。
- **Table 3, BEDLAM 多人全图：** GUSH3R PSNR=17.0、SSIM=0.53、LPIPS=0.34；独立组合为 14.5 / 0.31 / 0.47。但仅人体区域 PSNR=14.2，低于独立组合 14.7。
- **Table 8：** 总 540.27 ms/帧，其中 Human3R 264.82 ms (49.0%)；约 1.85 FPS，不是 30 FPS 实时系统。
- **Appendix B.3：** 500 帧 streaming 中，背景 voxel size 0.01 m 在 234 帧处达到 1620 万 Gaussians 并内存耗尽；0.03/0.05 m 可完成 500 帧，反映长序列内存/细节权衡。
- 原文主要检验渲染质量，**没有**给出能证明 human residual 反向降低 camera ATE/RPE 的独立实验结论。

## 优点

- 可复用的 Human3R backbone 与官方公开推理代码/权重入口，便于作为 unified online human-scene rendering baseline。
- 在 NeuMan、EMDB 和 BEDLAM 提供跨单人/多人、感知质量和效率的可比证据；诚实地区分全图与人体区域结果。
- Appendix 提供较有价值的长序列 Gaussian 内存瓶颈分析。

## 局限

- 作者明确指出依赖上游 Human3R 的相机/点图/人体检测与 SMPL-X 质量；错误可能直接传播。
- 复杂遮挡、身份关联、快速运动的面手和衣物仍可出现伪影；每个 SMPL-X 顶点绑定一个 Gaussian 限制细节表达。
- 核心贡献是可渲染表示及在线效率，而非显式 camera-human mutual refinement；未证明改善 camera trajectory，且不处理原生 ERP 变形。
- 当前论文评价主轴是 novel-view rendering；没有足够的真实滑雪 camera ATE/RPE、W-MPJPE 或 contact/foot-sliding 指标。

## 个人评价

**推断：**这篇适合作为 Human3R 后接的 scene-human appearance/renderability 强基线，而不是直接作为双移动 360° camera correction 的完整解决方案。0.54 s/帧和长序列 Gaussian 增长提醒：系统创新点可能首先体现在内存与几何一致性，而非单帧 PSNR。

## 与我的研究关联

- **Dual-360 world HMR：** 可将 perspective crop 的 camera/SMPL-X/pointmap 送入 GUSH3R，再研究跨物理相机的统一坐标、人体身份和遮挡区域渲染。
- **Camera-human mutual refinement：** **推断：**在统一 Gaussian 表示上加入 human keypoint reprojection、静态背景光度残差、support/contact 后，明确对 camera R/t/scale 做迭代更新；验证 human→camera 反馈，不能只报告渲染提升。
- **CanonFuse3D：** 在运动学 MPJPE/jerk 之外增加 novel-view PSNR/LPIPS、foreground/background 分区误差、500–1000 帧长序列内存与漂移对照。
- **阅读/复现：** 精读 Section 3 的双 Gaussian Decoder、Section 4 的 NeuMan/EMDB/BEDLAM、Table 3 分区结果、Appendix B.3/B.4；先运行官方 `infer.py --max_frames 3`，逐步加入长序列数据。

## 后续阅读

- [Human3R 原有笔记](../global-human-motion/2026-human3r.md)：共同 backbone 与相机/人体表示边界。
- [UniCon3R 原有笔记](../global-human-motion/2026-unicon3r-contact-aware-human-scene.md)：对比显式接触引导的人体修正。
- [JOSH 原有笔记](../global-human-motion/2026-josh-joint-optimization.md)：区别 optimization-based joint refinement。
- 后续核验正式 WACV 2027 proceedings 是否发布，及评测是否提供 camera ATE/RPE、真实新视角 GT 和 long-term identity consistency。

## 事实来源

- 原始论文：https://arxiv.org/abs/2607.05243 （Section 3/4、Tables 2/3、Appendix B/C）
- 作者项目：https://abkeito.github.io/gush3r-page/
- 官方仓库：https://github.com/abkeito/GUSH3R
