---
name: term-vault
description: Maintain a lightweight Obsidian-based personal terminology library for AI, medicine, statistics, research methods, and related study topics. Use when the user asks to explain a term and save, update, search, or connect it in an Obsidian vault; when the user says "记到术语库", "写进 Obsidian", "用 term-vault", "用术语库 skill"; or when a term note should be created or updated as Markdown.
---

# Term Vault

## Overview

Maintain the user's Obsidian vault as a Markdown terminology library. Explain terms clearly, then create or update structured term notes while preserving the vault's privacy boundaries.

## Defaults

- Default vault path: `/Users/stan/ai+med术语学习/AI医学学习`
- Default term folder: `术语库/`
- Default template path: `Templates/term-template.md`
- Default overview note: `术语总览.md`
- Preferred note language: Chinese explanations, with the canonical term retained in the source language when relevant. For English papers, use the English term as the note filename, frontmatter `term`, and H1. Put `## 英文术语` and `## 中文术语` immediately after the H1 so either language is easy to read and search.
- Preferred directory style: Chinese names unless the vault already uses `Terms/` and `Terms/Index.md`.

If the current workspace is not the vault root, locate the vault before writing. A directory containing `.obsidian/` is the Obsidian vault root. If only a subfolder is available, continue only inside the accessible term library and tell the user that full-vault linking is limited.

## Safety Boundary

Only create, read for matching, or modify files directly related to the terminology system by default:

- `术语库/` or `Terms/`
- `Templates/term-template.md`
- `术语总览.md` or `Terms/Index.md`

Do not proactively open, summarize, modify, or reorganize private notes, course notes, literature notes, daily notes, or unrelated vault files unless the user explicitly asks. To discover related terms, search filenames and term-library content first.

## Workflow

1. Explain the term first in the conversation. Include a plain-language meaning, the domain context, and one short example.
2. Locate the vault and term library. Prefer the defaults above, then fall back to a nearby `.obsidian/` root and existing `术语库/` or `Terms/`.
3. Search before writing. Check existing filenames, frontmatter `term`, `aliases`, headings, and obvious spelling variants. Avoid duplicate notes.
4. If no note exists, create one from the term note structure below. Use a stable, readable filename based on the canonical term. For terms from English sources, prefer the English canonical term and filename, such as `Structured output.md`, while keeping Chinese in `chinese_term`, aliases, and `## 中文术语`.
5. If a note exists, update it conservatively. Preserve useful existing content, update `updated`, add new aliases/examples/context, and append dated learning notes when appropriate.
6. Use Obsidian wikilinks for important related concepts, for example `[[few-shot]]`, `[[sensitivity]]`, `[[specificity]]`.
7. Update the overview note with a link to the term if it is missing. Keep the overview simple and readable.
8. Summarize what changed for the user, including the note path.

## Term Note Structure

Use this structure for new notes and as the target shape when improving old notes:

```markdown
---
type: term
term: canonical-term
english_term: English term when available
chinese_term: 中文术语或中文译名
aliases:
  - common alias
tags:
  - term/domain
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# canonical-term

## 英文术语

English term when available

## 中文术语

中文术语或中文译名

## 一句话解释

## 核心理解

## 使用场景

## 例子

## 容易混淆

## 相关术语

## 我的理解

## 学习记录

- YYYY-MM-DD: Initial note.
```

## Classification

Choose tags sparingly. Prefer broad, stable tags:

- `term/ai`
- `term/medicine`
- `term/statistics`
- `term/research-methods`
- `ai/ml`
- `medicine/diagnosis`
- `statistics/epidemiology`
- `research/evidence-based-medicine`

If unsure, use one broad `term/...` tag and improve later.

## Update Rules

- Keep YAML dates as `YYYY-MM-DD`.
- Do not overwrite the user's own wording in `我的理解`; append or lightly reorganize only when clearly helpful.
- Prefer appending a dated bullet under `学习记录` when the user gives new context, a source excerpt, or a personal understanding.
- When adding related terms that do not have notes yet, still use wikilinks so Obsidian can show uncreated linked notes.
- If the user asks only for an explanation and does not ask to save/update, explain without writing unless they invoked the skill specifically for saving.
- Avoid compound term notes when the components are independently reusable concepts. For example, create separate notes for `Structured output` and `Constrained decoding`, or for `Precision`, `Recall`, and `F1 score`, then cross-link them.
- For term notes based on English source material, use the English term as the canonical note title and put Chinese in `chinese_term`, aliases, and the `## 中文术语` section. This preserves English paper terminology while keeping Chinese search and review friendly.
