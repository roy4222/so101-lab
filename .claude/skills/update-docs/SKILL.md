---
name: update-docs
description: >
  Identifies which docs files need updating when the project state changes.
  Use this skill whenever the user says "update docs", "更新文件", "sync docs",
  "文件要改", or when a phase is completed/started, a new discovery is made,
  hardware changes, a blocker is resolved, or any significant project event
  occurs. Also trigger when the user finishes a task and you notice the docs
  are now stale (e.g., roadmap status doesn't match reality). Proactively
  suggest using this skill if you detect drift between docs and actual state.
---

# Update Docs

When the project state changes, multiple docs files may need updates to stay
consistent. This skill maps change types to affected files so nothing gets
missed.

## How to Use

1. Identify the **change type** from the list below
2. Read the affected files (in parallel where possible)
3. Present the user with a checklist: which files need what changes
4. After user confirms, make the edits
5. If a journal entry is warranted, draft one

## Change Type → Affected Files

### Phase Status Change (starting, completing, or blocking a phase)

This is the most common trigger. When a phase moves to `in-progress`, `done`,
or `blocked`:

| File | What to update |
|------|---------------|
| `docs/roadmap.md` | Change the phase's `狀態` column value |
| `docs/README.md` | Update the **Current Focus** table (現在做什麼 / 下一步 / Blocker) |
| `docs/journal/_index.md` | Update **目前階段** to match new active phase |
| `docs/architecture/overview.md` | If phase is `done`, change module status from「未來規劃」to「已實作」|
| Previous phase file | If completing: fill in any remaining 踩坑紀錄 and 產出物 |
| New active phase file | If starting: review 前置條件 are met, update any stale 步驟 |

### New Discovery or Lesson Learned

When the user learns something important (a gotcha, a better approach, a
debugging insight):

| File | What to update |
|------|---------------|
| Active phase file | Add to **踩坑紀錄** section |
| `docs/references/*.md` | If a new resource was discovered, add to relevant reference file + update 最後確認日期 |
| Journal entry | Draft a new `docs/journal/YYYY-MM-DD-title.md` if significant enough |

### Hardware Change

When hardware is added, removed, or reconfigured:

| File | What to update |
|------|---------------|
| `docs/README.md` | Update **硬體清單** table |
| `docs/goals.md` | Update **硬體清單** table at bottom |
| `docs/references/hardware.md` | Add/update entries + update 最後確認日期 |
| `docs/architecture/overview.md` | Update hardware layer if architecture changes |
| Active phase file | Update 步驟 if commands/ports/config changed |

### Training or Experiment Results

When a training run completes or experiment results are in:

| File | What to update |
|------|---------------|
| Active phase file (likely 04-training) | Add results to 產出物, update 成功標準 checkboxes |
| `docs/references/algorithms.md` | If new insights about algorithms, add to 備註 |
| Journal entry | Draft entry with results, hyperparams, WandB link |

### Blocker Resolved or New Blocker Found

| File | What to update |
|------|---------------|
| `docs/roadmap.md` | Update status (`blocked` → `in-progress`, or `in-progress` → `blocked`) |
| `docs/README.md` | Update Blocker row in Current Focus |
| Active phase file | Update **風險 / Blockers** section |
| Journal entry | Document what blocked and how it was resolved (or what's needed) |

### New Community Project or Reference Discovered

| File | What to update |
|------|---------------|
| `docs/references/community-projects.md` | Add with 4 required fields: 連結, 說明, 你可以學什麼, 和本專案的關聯 |
| Or other `docs/references/*.md` | Add to appropriate category + update 最後確認日期 |

### Weekly Journal Entry

| File | What to update |
|------|---------------|
| `docs/journal/YYYY-MM-DD-title.md` | Create new entry using template from `_index.md` |
| `docs/journal/_index.md` | Add to 最近日誌 list (keep top 5), add to monthly section, update Current Phase if needed |

## Journal Entry Rules

- Filename format: `YYYY-MM-DD-kebab-case-title.md`
- Use 1-3 tags from the registered set (see `_index.md` for the 12 allowed tags)
- Keep content blog-ready: clear enough for someone outside the project to follow
- Always link back to the relevant phase

## Output Format

After analyzing the change, present a checklist like:

```
根據 [change description]，以下文件需要更新：

- [ ] `docs/roadmap.md` — Phase 01 狀態改為 `done`
- [ ] `docs/README.md` — Current Focus 更新為 Phase 02
- [ ] `docs/journal/_index.md` — 目前階段更新
- [ ] `docs/architecture/overview.md` — 硬體層標為「已實作」
- [ ] `docs/journal/2026-04-01-phase01-complete.md` — 新增日誌

要我執行這些更新嗎？
```

Wait for user confirmation before editing. The user may want to skip some
updates or add context you don't have.

## Consistency Checks

After making updates, verify:
- `roadmap.md` 的 status 和 `README.md` 的 Current Focus 一致
- `architecture/overview.md` 的模組狀態和 `roadmap.md` 對齊
- `journal/_index.md` 的 Current Phase 指向正確的 phase
- 所有 reference 檔案的「最後確認日期」是今天（如果有修改的話）
- Phase 文件的成功標準 checkbox 和實際進度一致
