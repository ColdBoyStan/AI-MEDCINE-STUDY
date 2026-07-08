---
name: term-deep-dive
description: Deepen existing Obsidian terminology notes by creating or updating linked long-form study notes. Use when the user asks to "深入学习", "深挖", "系统学习", "进阶学习", "扩展这个术语", "deep dive", "make this term deeper", or wants an existing term in the Obsidian term library expanded into a structured deeper learning note connected to the original term note.
---

# Term Deep Dive

## Overview

Turn a short Obsidian term card into a connected deeper study note without making the original card bulky. Keep the original note useful for quick review, and store detailed learning in a linked note under the term library.

## Defaults

- Default vault path: `/Users/stan/ai+med术语学习/AI医学学习`
- Base term folder: `术语库/`
- Deep-dive folder: `术语库/深入学习/`
- Base overview note: `术语总览.md`
- Deep-dive filename pattern: `<canonical term> - 深入学习.md`
- Preferred language: Chinese explanation with English terms retained when useful.
- If the vault uses English folders, use `Terms/Deep Dives/` and `Terms/Index.md` instead.

A directory containing `.obsidian/` is the Obsidian vault root. If the current workspace is only a subfolder, continue only inside the accessible term library and tell the user that full-vault linking is limited.

## Safety Boundary

Only read or modify terminology-system files by default:

- `术语库/` or `Terms/`
- `术语库/深入学习/` or `Terms/Deep Dives/`
- `Templates/term-template.md`
- `术语总览.md` or `Terms/Index.md`

Do not proactively open, summarize, modify, or reorganize private notes, course notes, literature notes, daily notes, or unrelated vault files unless the user explicitly asks.

## Workflow

1. Identify the requested term and locate its existing base term note in `术语库/` or `Terms/`.
2. If no base note exists, tell the user clearly. Do not create an unanchored deep-dive note unless the user explicitly asks; suggest using `$term-vault` to create the base term first.
3. Read the base term note and preserve its role as a short review card.
4. Search for an existing deep-dive note using the filename pattern and any existing `深入学习` links.
5. Create or update the deep-dive note in `术语库/深入学习/`.
6. Add a minimal link from the base term note to the deep-dive note. Prefer a `## 深入学习` section; if it already exists, add or update one bullet.
7. Add a backlink from the deep-dive note to the base term note.
8. Use Obsidian wikilinks for related concepts, including uncreated notes when they are useful future anchors.
9. Update YAML `updated` dates as `YYYY-MM-DD`.
10. Summarize the explanation and the file changes for the user.

## Base Note Update

Keep the original term note short. Add only this kind of link:

```markdown
## 深入学习

- [[术语库/深入学习/canonical-term - 深入学习|canonical-term - 深入学习]]
```

If the base note has `## 学习记录`, append a dated bullet such as:

```markdown
- YYYY-MM-DD: 创建或更新深入学习笔记：[[术语库/深入学习/canonical-term - 深入学习|深入学习]]。
```

Do not overwrite the user's own wording in `我的理解`.

## Deep-Dive Note Structure

Use this structure for new deep-dive notes:

```markdown
---
type: term-deep-dive
term: canonical-term
parent: "[[canonical-term]]"
tags:
  - term/deep-dive
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# canonical-term - 深入学习

> 对应术语：[[canonical-term]]

## 学习目标

## 为什么重要

## 背景与问题

## 深入解释

## 分层理解

### 初学者版本

### 实践者版本

### 研究者版本

## 例子与反例

## 常见误区

## 与相关术语的关系

## 如何应用或判断

## 自测问题

## 进一步阅读

## 我的理解

## 学习记录

- YYYY-MM-DD: Initial deep-dive note based on [[canonical-term]].
```

## Depth Guidelines

Adapt the deep-dive content to the domain:

- AI or machine learning: include mechanism, training/inference context, data assumptions, evaluation, limitations, and implementation implications.
- Medicine or clinical workflow: include clinical meaning, use cases, risks, privacy/compliance issues, evaluation, and what should not be inferred.
- Statistics or epidemiology: include intuition, assumptions, formula-level meaning when helpful, interpretation, common errors, and worked examples.
- Research methods: include study design context, bias, validity, evidence hierarchy, reporting implications, and practical checklist questions.

For any domain, prefer a clear conceptual model over a long encyclopedia entry. Include concrete examples, counterexamples, and "how to tell when this matters".

## Source Handling

Do not invent citations. If the user asks for sources, latest evidence, clinical guidelines, product details, legal/regulatory claims, or other time-sensitive facts, verify with browsing or reliable primary sources when available and include links in `## 进一步阅读`.

If no external sources were checked, write learning content from general knowledge and keep `## 进一步阅读` empty or note that no sources were added.

## Update Rules

- Preserve useful existing deep-dive content; update rather than rewrite unless the user asks for a rewrite.
- Add new material under the most relevant section.
- Add dated bullets under `## 学习记录` for major updates, source-based additions, or user-provided context.
- Keep the base term note concise; the deep-dive note is where detailed material belongs.
- Use stable Obsidian links. Prefer full path links for deep-dive notes if ambiguity is possible.
