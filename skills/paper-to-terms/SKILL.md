---
name: paper-to-terms
description: Convert papers from an Obsidian-adjacent literature folder or a read-only Zotero MCP collection into structured Obsidian literature notes and update the user's terminology library. Use when the user asks to process a PDF, paper, article, Zotero collection, Zotero item, Zotero export, BibTeX/RIS entry, "文献库" item, "整理文献", "转成文献笔记", "提取术语", "加入术语库", or to turn papers into Markdown notes plus linked term notes.
---

# Paper to Terms

## Overview

Transform a user-supplied paper or Zotero collection into Obsidian literature notes and a small set of linked terminology notes. Treat messy PDF filenames as input hints only; derive consistent literature-note metadata and filenames from the paper metadata or paper text whenever possible.

## Defaults

- Default vault path: `/Users/stan/ai+med术语学习/AI医学学习`
- Input folder: `文献库/`
- Literature note folder: `文献笔记/`
- Topic folder: `专题/`
- Term folder: `术语库/`
- Deep-dive folder: `术语库/深入学习/`
- Overview note: `术语总览.md`
- Preferred language: Chinese study notes with paper titles and citation metadata retained in their original language. For terminology notes extracted from English papers, use the English term as the canonical filename, frontmatter `term`, and H1; put Chinese translation/explanation under `chinese_term` and `## 中文术语`.

If the vault uses English folders, use `Literature/`, `Literature Notes/`, `Topics/`, `Terms/`, and `Terms/Index.md` instead.

## Safety Boundary

Only read or modify the paper-learning system by default:

- `文献库/` or `Literature/`
- `文献笔记/` or `Literature Notes/`
- `专题/` or `Topics/`
- `术语库/` or `Terms/`
- `术语库/深入学习/` or `Terms/Deep Dives/`
- `Templates/`
- `术语总览.md` or `Terms/Index.md`

Do not modify Zotero internal databases, Zotero sync files, private notes, course notes, daily notes, or unrelated vault files unless the user explicitly asks.

## Input Handling

Accept these inputs:

- A PDF path in `文献库/`.
- A messy PDF filename, such as `download.pdf`, `paper (3).pdf`, or a DOI-like filename.
- A paper title, DOI, PMID, or author-year hint if the file can be found nearby.
- A Zotero collection name or key, such as `PhD P2 - PEDro`, when `zotero-readonly` MCP tools are available.
- A Zotero item key when `zotero-readonly` MCP tools are available.
- Optional companion metadata files: `.bib`, `.ris`, `.enw`, `.txt`, or `.md`.

Use this priority order for metadata:

1. Companion `.bib` or `.ris` metadata that clearly matches the PDF.
2. Zotero MCP metadata from `zotero-readonly`, when the user names a Zotero collection or item.
3. PDF first page and embedded metadata.
4. DOI/PMID/title visible in extracted PDF text.
5. User-provided title or citation text.
6. Original filename as a last resort.

If title, year, or first author remain uncertain, say so in the final response and mark the frontmatter field as `unknown` rather than inventing it.

## PDF Reading Ladder

Use a staged reading strategy instead of treating every PDF the same.

Stage 1: Zotero metadata

- Use Zotero MCP metadata first for title, authors, date, DOI, journal, abstract, collection, item key, and PDF attachment path.
- Prefer Zotero metadata over messy PDF filenames.
- If the abstract is rich and no PDF text is available, create an `abstract-only` note rather than pretending to have read the full paper.

Stage 2: Standard PDF text extraction

- Use bundled PDF tools such as `pdfplumber`, `pypdf`, PyMuPDF, or Poppler-assisted checks for ordinary searchable PDFs.
- This is the default path for clean articles with selectable text and simple structure.
- Use this level for quick literature notes when section structure is obvious enough.

Stage 3: Structured scholarly PDF parsing with GROBID

- Use GROBID when the paper has complex structure, important references/citation contexts, messy metadata, many section boundaries, or when a reliable header/body/reference split matters.
- Prefer GROBID output as TEI XML as the high-fidelity source. Markdown or JSON projections can be used for LLM input, but treat TEI as the source of truth.
- If the `grobid-readonly` MCP tools are available, prefer them over raw HTTP calls:
  - `grobid_status` to confirm the local service is alive.
  - `grobid_process_header` when metadata/header is uncertain.
  - `grobid_process_fulltext` when section structure matters.
  - `grobid_process_references` when bibliography or citation context matters.
- If no MCP tool is available but a local GROBID service is running, call `/api/processFulltextDocument` for full-text structure, `/api/processHeaderDocument` for metadata/header, and `/api/processReferences` for references when needed.
- Record in the literature note when GROBID was used, for example `pdf_extraction_method: grobid`.

Stage 4: OCR fallback

- If the PDF is scanned/image-only or extraction returns too little text, mark `processing_status: needs-ocr`.
- Do not create a confident full-paper note from unreadable PDF text.
- After OCR, run Stage 2 or Stage 3 again depending on quality.

Default decision:

- Start with Stage 1 + Stage 2.
- Escalate to Stage 3 when structure or references matter.
- Escalate to Stage 4 only when text is not readable.
- Do not use GROBID for every paper by default; it is an on-demand upgrade path.

## Filename Normalization

Normalize every literature note filename, independent of the PDF filename:

```text
FirstAuthor Year - Short Title.md
```

Examples:

```text
Smith 2024 - Local LLMs in Clinical Settings.md
Wang 2023 - Retrieval Augmented Generation for Clinical QA.md
UnknownYear - Untitled Paper.md
```

Rules:

- Use first author's family name when available.
- Use `UnknownAuthor` if no author is known.
- Use four-digit year when available; otherwise use `UnknownYear`.
- Keep the short title meaningful and under about 80 characters.
- Remove filesystem-problematic characters: `/`, `:`, `?`, `*`, `"`, `<`, `>`, `|`.
- If a note with the same normalized filename already exists, update it instead of creating a duplicate.

## Topic Organization

When the paper comes from a Zotero collection, use the collection name as the Obsidian topic by default.

Use this layout:

```text
文献笔记/<topic>/
  FirstAuthor Year - Short Title.md

专题/
  <topic>.md
```

Rules:

- If the user explicitly provides a topic, use it.
- If the source path is `文献库/<topic>/<paper.pdf>`, infer `<topic>` from the folder.
- If the source is a Zotero collection, infer `<topic>` from the collection name.
- Keep the global `术语库/` shared across topics; do not duplicate term notes per topic.
- Create or update `专题/<topic>.md` with links to processed literature notes and shared key terms.

## Processing Status and Idempotency

Avoid re-processing the same Zotero item unless the user explicitly asks to update, refresh, redo, or force reprocess it.

Use this identity order:

1. `zotero_key` from Zotero MCP metadata.
2. DOI, if present and unique.
3. PMID, if present and unique.
4. Normalized literature-note filename.

Before processing a Zotero collection:

1. Build a processed-item map by searching `文献笔记/**/*.md` for frontmatter fields such as `zotero_key`, `doi`, `pmid`, and `processing_status`.
2. For each Zotero item in the collection, check whether its `zotero_key` already appears in an existing literature note.
3. If it exists and the user did not request an update, skip full processing and keep the existing note.
4. If the Zotero item version is newer than `zotero_item_version` in the note, mention that an update is available; do not rewrite by default unless the user asked for updating.
5. If no existing note is found, create the literature note and mark it as processed.

Supported `processing_status` values:

- `processed`: full literature note was created from readable PDF or sufficiently rich metadata.
- `abstract-only`: only abstract/metadata were available.
- `needs-ocr`: PDF exists but text extraction was not readable.
- `metadata-only`: only bibliographic metadata were available.
- `needs-review`: generated note should be manually checked.

Maintain a lightweight status table in the topic note under `## Zotero处理状态`. This table is for human visibility; the primary machine check remains the per-note `zotero_key` frontmatter.

## Workflow

1. Locate the vault root. A directory containing `.obsidian/` is the Obsidian vault root.
2. Locate the requested paper in `文献库/`, the user-provided path, or Zotero via `zotero-readonly` MCP tools.
3. If the user names a Zotero collection, use `zotero_collection_snapshot` to get compact metadata and PDF attachment paths; process each requested item.
4. Build the processed-item map before writing. Skip already-processed Zotero items unless the user explicitly asks to update or redo them.
5. Extract readable text from each new or update-requested PDF when a local PDF path is available. If the PDF is scanned or unreadable, tell the user OCR is needed and avoid creating low-quality full-paper notes.
6. Identify metadata using the priority order above.
7. Create or update one structured literature note in `文献笔记/<topic>/` using the normalized filename.
8. Extract a focused set of key terms, usually 5-12 for the main `## 关键术语` list, then perform a readability coverage pass over the full literature note. Prefer terms that are necessary for understanding the paper, reusable across future reading, or likely to appear in AI/medicine/statistics/research-methods study. Use atomic, reusable terms rather than bundled compound labels; for example, split `Structured output and constrained decoding` into `Structured output` and `Constrained decoding`, and split `Precision, recall, and F1` into `Precision`, `Recall`, and `F1 score` unless the paper explicitly treats the phrase as a single named construct.
9. Readability coverage pass: every technical word, abbreviation, method, metric, study design, statistical quantity, tool name, or evaluation concept that a diligent reader may need explained to understand the note must either link to an existing term note or receive a new atomic term note. Do not create notes for generic language, author names, journal names, product names used only as labels, or obvious everyday words. If a term is mentioned in prose but is secondary, it may be linked in prose without being promoted to the short `## 关键术语` list.
10. For each key term and readability-coverage term, search `术语库/` before writing. Update existing notes; create missing notes using the term-card style from `$term-vault`.
11. Link the core terms under `## 关键术语`, and link secondary readability terms at their first meaningful mention in the relevant section.
12. Link each created or updated term note back to the literature note, usually under `## 学习记录` or a concise `## 文献语境` section.
13. Create or update `专题/<topic>.md` with literature-note links, shared key terms, and the Zotero processing status table.
14. Update `术语总览.md` for newly created term notes if it exists.
15. Summarize created, updated, skipped, and blocked items separately, distinguishing core `关键术语` from secondary readability terms when useful.

## Literature Note Structure

Use this structure for new literature notes:

```markdown
---
type: literature-note
title: "Full paper title"
authors:
  - "First Author"
year: YYYY
journal: "Journal or venue"
doi: "10.xxxx/xxxxx"
pmid: "PMID if available"
source_file: "文献库/original-file.pdf"
zotero_key: "item key if sourced from Zotero"
zotero_collection: "Collection name if sourced from Zotero"
zotero_collection_key: "Collection key if sourced from Zotero"
zotero_item_version: 123
zotero_pdf_attachment_key: "PDF attachment key if available"
topic: "Topic or Zotero collection name"
pdf_extraction_method: zotero-metadata | standard-pdf | grobid | ocr
grobid_used: false
processing_status: processed
processed_at: YYYY-MM-DD
tags:
  - literature
  - literature/ai-medicine
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# FirstAuthor Year - Short Title

## 基本信息

- 标题：
- 作者：
- 年份：
- 期刊/会议：
- DOI：
- PMID：
- 原文：
- Zotero key：
- Zotero collection：
- PDF extraction method：
- Processing status：
- 专题：[[专题/topic]]

## 一句话总结

## 研究问题

## 背景与动机

## 方法

## 关键发现

## 局限性

## 临床/科研意义

## 关键术语

- [[term-1]]：这篇文献中如何使用这个概念。
- [[term-2]]：这篇文献中如何使用这个概念。

## 对我的启发

## 待追问问题

## 学习记录

- YYYY-MM-DD: Created from source PDF.
```

Do not paste the full paper text into the note. The literature note should be a study artifact: concise, searchable, linkable, and useful for later review.

## Term Update Pattern

For an existing term note, preserve the user's wording and append paper-specific context. Prefer this lightweight addition:

```markdown
## 文献语境

- [[文献笔记/FirstAuthor Year - Short Title|FirstAuthor Year - Short Title]]：一句话说明这篇文献如何使用该术语。
```

If the note already has `## 学习记录`, append:

```markdown
- YYYY-MM-DD: 从 [[文献笔记/FirstAuthor Year - Short Title|FirstAuthor Year - Short Title]] 补充文献语境。
```

For a new term note, use the existing `$term-vault` style:

- YAML frontmatter with `type: term`, `term`, `english_term`, `chinese_term`, `aliases`, `tags`, `created`, `updated`. For English papers, `term` and the filename should normally be English; Chinese should be searchable through `chinese_term`, aliases, and `## 中文术语`.
- Sections: `英文术语`, `中文术语`, `一句话解释`, `核心理解`, `使用场景`, `例子`, `容易混淆`, `相关术语`, `我的理解`, `学习记录`.
- Include a paper-specific learning-record bullet linking back to the literature note.

## Topic Note Structure

Use this structure for new topic notes:

```markdown
---
type: topic
topic: "Topic name"
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Topic name

## 研究主题

## 核心问题

## 文献列表

- [[文献笔记/Topic name/FirstAuthor Year - Short Title]]

## Zotero处理状态

| Zotero key | 文献 | 状态 | 最近处理 |
|---|---|---|---|
| ITEMKEY | [[文献笔记/Topic name/FirstAuthor Year - Short Title]] | processed | YYYY-MM-DD |

## 共通术语

- [[term-1]]
- [[term-2]]

## 我的阶段性理解

## 待追问问题

## 学习记录

- YYYY-MM-DD: Created or updated from Zotero collection / literature processing.
```

## Term Selection Guidelines

Do not create a note for every noun phrase, and do not merge independent concepts into one note just because they appear together. Prefer:

- Concepts that the user is likely to meet again.
- Terms central to the paper's method, results, assumptions, or interpretation.
- Abbreviations that need expansion.
- Clinical, statistical, AI, ML, research-methods, or evaluation terms.
- Terms that connect multiple notes through Obsidian wikilinks.

Skip:

- Bundled labels made by joining multiple independent terms, unless the source treats the phrase as a named construct. Split them into separate linked notes instead.
- Generic academic words.
- One-off dataset names unless the paper centers on them.
- Very broad words already obvious to the user unless the paper uses them in a special technical sense.

If there are too many candidates for the short key-term list, keep only the top 8-12 under `## 关键术语`; however, do not leave note-critical technical terms unexplained. Link secondary terms in prose and create or update their term notes when they affect readability.

## Source and Accuracy Rules

- Do not invent citation metadata.
- Do not make clinical or regulatory claims stronger than the paper supports.
- Separate "paper says" from "my interpretation".
- If the user asks for latest guidelines, legal/regulatory status, citations beyond the paper, DOI lookup, or external verification, browse or use reliable primary sources when available.
- If only the abstract is readable, clearly label the note as abstract-based.

## Updating Existing Notes

When re-processing the same paper:

- Update the existing literature note rather than duplicating it.
- Preserve user-added `对我的启发`, `待追问问题`, and `我的理解` content.
- Add new dated bullets under `## 学习记录`.
- Avoid rewriting stable sections unless the previous extraction was clearly poor or the user asks for a rewrite.
- If the user processes a Zotero collection without specifying a paper, skip existing `zotero_key` notes by default and report them under "已跳过".

## User-Facing Summary

After processing, report:

- The literature note created or updated.
- The term notes created.
- The term notes updated.
- Any unreadable PDF/OCR issue.
- Any metadata uncertainty.
