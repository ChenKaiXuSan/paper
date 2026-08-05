# Paper Reading Repository Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish a public GitHub repository named `paper` containing a maintainable bilingual structure for paper-reading notes.

**Architecture:** Use a Markdown-only repository with one reusable note template and one directory per research topic. Keep the README as a manually maintained bilingual index; do not add a website generator, API integration, or automated indexing.

**Tech Stack:** Git, GitHub, Markdown, CC BY 4.0

## Global Constraints

- The GitHub repository name is exactly `paper` and its visibility is public.
- Repository navigation, metadata labels, and section headings are bilingual or understandable in English without translation.
- Each paper note includes separate English and Chinese one-sentence takeaways.
- Do not invent papers the owner has read.
- Use CC BY 4.0 for the written notes.

---

### Task 1: Create the bilingual repository content

**Files:**
- Create: `README.md`
- Create: `LICENSE`
- Create: `templates/paper-note-template.md`
- Create: `papers/3d-human-pose/.gitkeep`
- Create: `papers/360-vision/.gitkeep`
- Create: `papers/multiview/.gitkeep`
- Create: `papers/medical-ai/.gitkeep`
- Create: `papers/others/.gitkeep`

**Interfaces:**
- Consumes: the approved repository design in `docs/superpowers/specs/2026-08-05-paper-reading-repository-design.md`
- Produces: a complete Markdown repository whose README links to the template and all five topic directories

- [ ] **Step 1: Run the structural acceptance check before implementation**

```bash
test -f README.md && test -f LICENSE && test -f templates/paper-note-template.md
```

Expected: FAIL because the public repository files do not exist yet.

- [ ] **Step 2: Create the README**

Write a bilingual landing page with these exact sections: `About / 关于`, `Topics / 主题`, `Reading Status / 阅读状态`, `Recently Read / 最近阅读`, `All Papers / 全部论文`, `Note Template / 笔记模板`, and `License / 许可协议`. Link each topic to its relative directory and the template to `templates/paper-note-template.md`. State explicitly that no reading notes have been added yet.

- [ ] **Step 3: Create the note template**

Include metadata for title, authors, venue, year, reading date, reading status, and tags; optional links for paper, code, dataset, and project page; separate `Takeaway (English)` and `一句话总结（中文）` sections; and analysis sections for research question, method, datasets and metrics, results, strengths, limitations, personal assessment, research relevance, and follow-up reading.

- [ ] **Step 4: Add licensing and topic directories**

Use the unmodified Creative Commons Attribution 4.0 International legal text in `LICENSE`. Add one `.gitkeep` file in each of the five empty topic directories so Git tracks the initial structure.

- [ ] **Step 5: Run the structural and link checks**

```bash
test -f README.md
test -f LICENSE
test -f templates/paper-note-template.md
for path in papers/3d-human-pose papers/360-vision papers/multiview papers/medical-ai papers/others; do test -d "$path"; done
rg -n 'Takeaway \(English\)|一句话总结（中文）' templates/paper-note-template.md
```

Expected: all commands exit successfully and both takeaway headings are found.

- [ ] **Step 6: Commit the repository content**

```bash
git add README.md LICENSE templates papers docs
git commit -m "feat: initialize bilingual paper reading notes"
```

### Task 2: Publish and verify the public GitHub repository

**Files:**
- Modify: local Git remote configuration only

**Interfaces:**
- Consumes: the committed `main` branch from Task 1
- Produces: public repository `ChenKaiXuSan/paper` with the local `main` branch pushed to `origin`

- [ ] **Step 1: Confirm authentication and name availability**

```bash
gh auth status
gh repo view ChenKaiXuSan/paper
```

Expected: authentication succeeds; repository lookup reports not found before creation. If the repository already exists, stop rather than overwrite it.

- [ ] **Step 2: Create and push the public repository**

```bash
gh repo create ChenKaiXuSan/paper --public --source=. --remote=origin --push --description "Bilingual paper reading notes on computer vision, 3D human pose, multiview learning, and medical AI."
```

Expected: GitHub creates the public repository and pushes `main`.

- [ ] **Step 3: Verify the remote repository**

```bash
gh repo view ChenKaiXuSan/paper --json nameWithOwner,visibility,url,defaultBranchRef
git status --short
```

Expected: `nameWithOwner` is `ChenKaiXuSan/paper`, visibility is `PUBLIC`, the default branch is `main`, and the working tree is clean.
