# Paper Repository Taxonomy Design — 2026-08-26

## 目标

把仓库从“按早期宽泛主题堆放论文”升级为“按研究问题组织论文 + 用 tags 表示方法属性 + 用 collections 表示研究脉络”的知识库结构。

## 一级主题

1. `global-human-motion`：moving camera、world-coordinate HMR、global trajectory、human-scene-camera alignment。
2. `3d-human-pose`：camera-relative 3D pose / mesh recovery 与人体表示。
3. `multiview-geometry`：多视角、相机几何、calibration、SfM/SLAM、cross-view fusion。
4. `360-vision`：ERP、fisheye、omnidirectional vision。
5. `motion-understanding`：动作识别、时序/周期运动表征。
6. `medical-ai`：clinical gait、spine、medical video。
7. `sports-biomechanics`：体育动作、生物力学、physics/contact、coaching。

一级目录回答“这篇论文主要解决什么研究问题”；`tags` 回答“它用了什么方法/适用于什么场景”。因此不为 diffusion、VLM、SLAM 等单一方法建立一级目录。

## Collections

`collections/` 不复制单篇论文内容，而是把已有笔记组织为研究问题、方法演进、共识、研究空白与下一步实验。首批包括：

- moving-camera-world-hmr
- 360-selfie-human-reconstruction
- multiview-pose-fusion
- clinical-gait-analysis
- sports-motion-analysis

## 导航

- README：仓库入口，只列主题与 collections。
- PAPER_INDEX：唯一完整论文索引。
- Collections：面向研究问题的 curated reading routes。
- Paper notes：单篇事实、方法、结果和个人判断。

## 迁移原则

- 移动文件时复用现有 Git blob，不重写论文正文。
- 更新所有索引链接，避免 dangling path。
- 不删除论文笔记。
- 新增/移动论文时同步更新 PAPER_INDEX；若影响研究脉络，再更新对应 collection。
