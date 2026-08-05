# Paper Reading Repository Design

## Objective

Create a public GitHub repository named `paper` for Kaixu Chen's bilingual paper-reading notes. The repository must remain easy to maintain as the number of papers grows, while making the notes useful to both Chinese- and English-speaking readers.

## Audience and Language

- The repository is public.
- Repository navigation, metadata labels, and section headings are bilingual or understandable in English without translation.
- Each paper note includes both an English and a Chinese one-sentence takeaway.
- Detailed analysis may use Chinese for efficiency, with English summaries preserving international readability.

## Repository Structure

```text
paper/
├── README.md
├── LICENSE
├── templates/
│   └── paper-note-template.md
└── papers/
    ├── 3d-human-pose/
    ├── 360-vision/
    ├── multiview/
    ├── medical-ai/
    └── others/
```

The repository uses one Markdown file per paper. Filenames follow `YYYY-short-paper-title.md`, use lowercase ASCII, and separate words with hyphens. A paper belongs to the most relevant primary topic folder; additional topics are represented by tags inside the note rather than duplicate files.

## README Design

`README.md` serves as the public landing page and contains:

1. A bilingual repository introduction.
2. A short explanation of the note format and reading-status labels.
3. Topic links for the five initial categories.
4. A "Recently Read / 最近阅读" table.
5. An "All Papers / 全部论文" index grouped by topic.
6. Instructions for creating a new note from the template.

The index is maintained manually in the first version. Automatic indexing, a documentation website, and GitHub API integration are intentionally excluded to keep maintenance simple.

## Paper Note Schema

Every note created from `templates/paper-note-template.md` contains:

- Title, authors, venue, publication year, and reading date.
- Reading status: `skimmed`, `read`, or `deep-read`.
- Topic tags.
- Links to the paper, code, dataset, and project page when available.
- One-sentence takeaway in English.
- 一句话中文总结。
- Research question and motivation.
- Core method.
- Datasets, evaluation metrics, and main results.
- Strengths.
- Limitations.
- Personal assessment.
- Relevance to current research.
- Follow-up questions or papers.

Unavailable external resources are written as `Not available / 暂无`, avoiding empty or misleading links.

## Licensing

The repository uses the Creative Commons Attribution 4.0 International license (CC BY 4.0), which is appropriate for publicly shared written notes. Notes must summarize papers in the author's own words and must not reproduce substantial copyrighted passages, figures, or tables without permission.

## Validation

The initial repository is accepted when:

- All required directories and files exist.
- README links to every topic directory and template using valid relative paths.
- The template contains every required field in the paper-note schema.
- English and Chinese takeaways are visibly separate.
- No placeholders such as `TODO` or broken example links remain in the README.
- Markdown renders cleanly on GitHub.

No automated workflow is required for the first version. Validation is performed with local file checks and a Markdown link scan before publication.

## Initial Scope

The first published version contains the repository structure, bilingual README, license, note template, and topic directories. It does not invent papers the owner has read. Real paper notes will be added only from user-provided titles, URLs, BibTeX, PDFs, or reading history.
