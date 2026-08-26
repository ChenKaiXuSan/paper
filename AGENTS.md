# 论文笔记仓库维护规则

本文件定义当前仓库的有效维护规则。后续 agent、自动化任务或人工新增论文笔记时，应以本文件为最高优先级。

## 1. 笔记语言

- 从 2026-08-07 起，新论文笔记统一使用中文总结和分析。
- 论文原标题、作者姓名、venue、模型名、方法名、数据集名和指标名等专有名词保留原文。
- 历史双语笔记无需批量改写。

## 2. 一级主题目录

一级目录按**研究问题**组织，而不是按单一方法名组织：

- `papers/global-human-motion/`：moving camera、world-coordinate HMR、global trajectory、human-scene-camera alignment、global motion refinement。
- `papers/3d-human-pose/`：camera-relative 3D pose / mesh recovery、HPE/HMR backbone、body representation。
- `papers/multiview-geometry/`：multi-view learning、camera calibration、SfM/SLAM、camera pose、cross-view geometry、multi-camera fusion。
- `papers/360-vision/`：equirectangular、fisheye、omnidirectional、360° geometry / pose。
- `papers/motion-understanding/`：action recognition、motion representation、temporal learning、periodic motion。
- `papers/medical-ai/`：clinical gait、spine、medical video、clinical diagnosis / prognosis。
- `papers/sports-biomechanics/`：sports motion、biomechanics、physics-based motion、coaching、ground reaction / contact。

如果论文同时属于多个主题，只保留一个主目录，通过 `tags` 表示其他属性。不要按 `diffusion`、`VLM`、`SLAM` 等单一方法建立一级目录。

## 3. Collections

`collections/` 用于整理研究脉络，而不是复制论文笔记。

- 每个 collection 围绕一个研究问题组织论文，可包含问题定义、方法路线、关键论文、研究空白与下一步阅读。
- collection 中只链接已有论文笔记；不要复制整篇笔记正文。
- 当前主要 collections：moving-camera world HMR、360 selfie human reconstruction、multiview pose fusion、clinical gait analysis、sports motion analysis。
- 新论文如果明显改变某条研究脉络，应同步更新相关 collection。

## 4. 去重规则

新增笔记前必须检查：
1. 完整标题或核心标题关键词；
2. arXiv ID、DOI 或其他稳定标识；
3. paper/project/code 链接；
4. `PAPER_INDEX.md` 与 `papers/` 现有独立笔记。

仅在其他笔记正文被提及、但没有独立笔记的论文，不算已收录。

## 5. 元数据

每篇笔记保留 YAML front matter，至少包含 `title`、`authors`、`venue`、`year`、`reading_date`、`status`、`tags`。

`status` 只使用：
- `skimmed`
- `read`
- `deep-read`

不得因为笔记篇幅较长自动提高阅读状态。

## 6. 笔记结构

新笔记原则上包含：
1. `基本信息`
2. `一句话总结`
3. `研究问题与动机`
4. `核心方法`
5. `数据集与评价指标`
6. `主要结果`
7. `优点`
8. `局限`
9. `个人评价`
10. `与我的研究关联`
11. `后续阅读`

不要为了填模板而编造信息。

## 7. 来源与事实

- 优先使用论文原文、arXiv/出版方页面、作者项目主页、官方代码仓库和官方数据集页面。
- 数值、样本量、实验设置、单位、发布日期和 venue 等必须可核验。
- 作者结论与个人判断必须区分；推断需明确标注。
- 不确定的信息写“暂无”或保留未核验状态，不猜测。

## 8. PAPER_INDEX.md

- `PAPER_INDEX.md` 是唯一完整论文索引。
- 按上述 7 个一级主题分组，每个主题内部按 `reading_date` 从新到旧排列。
- 新增、移动、重命名或删除笔记时必须同步更新索引。
- README 不维护逐篇论文列表。

## 9. README.md

README 只承担仓库入口功能：仓库简介、7 个研究主题、collections、论文索引、阅读状态、模板和维护规则。

## 10. 模板

- 单篇论文使用 `templates/paper-note-template.md`。
- 新 collection 使用 `templates/collection-template.md`。

## 11. 修改安全

- 修改前记录远程 `main` HEAD。
- 一组相关维护尽量组成单一 commit。
- 在最终移动 `main` 前再次确认 HEAD 未发生并发变化。
- 禁止 force push，除非用户明确要求且风险已确认。
- 对批量重构，优先复用原 blob 做路径移动，避免无意义改写论文正文。
