---
name: project-onboard
description: >
  Fast project onboarding for AI agents entering this SO-101 robotics project.
  Use this skill at the START of every new conversation, before doing any work.
  Also use when: the user says "onboard", "catch me up", "what's the status",
  "where are we", "專案狀況", "目前進度", or any question about overall project
  state. This skill ensures you never waste time re-discovering what's already
  documented. If you're unsure whether to use this skill, use it — the cost of
  reading a few files is far less than the cost of giving outdated advice.
---

# Project Onboard

This is a robotics learning project building toward a dual-arm mobile robot.
The docs are well-structured — read them in the right order and you'll have
full context in under 30 seconds of tool calls.

## Onboarding Sequence

Read these files in this exact order. Each one is short and gives you a
different layer of understanding. Parallelize the reads where marked.

### Step 1 — Big Picture (read in parallel)

```
docs/README.md          → Current Focus (what's happening NOW), hardware list, nav links
docs/goals.md           → Vision, 8-phase milestones, success criteria, reproducibility requirements
docs/roadmap.md         → Phase table with priorities (P0-P3), status, dependencies, Mermaid graph
```

These three files tell you: what, why, and when.

### Step 2 — Architecture

```
docs/architecture/overview.md → Module diagram, data flow, which modules are implemented vs planned
```

This tells you: how the pieces fit together, and what exists today vs what's future.

### Step 3 — Current Phase (read only what's active)

Check `roadmap.md` for which phases have status `in-progress`. Then read only
those phase files:

```
docs/phases/01-hardware-setup.md    (if Phase 01 is active)
docs/phases/02-teleoperation.md     (if Phase 02 is active)
...etc
```

Each phase file has 10 sections: 目標, 前置條件, 輸入/輸出, 步驟, 驗證方式,
關鍵連結, 成功標準, 產出物, 踩坑紀錄, 風險/Blockers.

**Do NOT read all 8 phase files.** Only read active ones + the one immediately
before (for context on what was already done) and the one immediately after
(to understand what's coming next).

### Step 4 — Latest Journal (if exists)

```
docs/journal/_index.md → Current Phase pointer + recent entries
```

If there are journal entries, read the most recent 1-2 for ground-level context
(what happened last week, current blockers, discoveries).

### Step 5 — References (only on demand)

Do NOT read reference files during onboarding. They're for when the user asks
about specific topics:

```
docs/references/hardware.md           → BOM, datasheets, wiring
docs/references/algorithms.md         → ACT, Diffusion, VLA papers
docs/references/software.md           → LeRobot, uv, accelerate, tools
docs/references/simulation.md         → Isaac Sim, MuJoCo
docs/references/community-projects.md → Hackathon projects, XLeRobot
```

## After Onboarding

Once you've read Steps 1-4, respond with a brief status summary:

```
目前狀態：Phase [N] — [name] ([status])
Current Focus：[from README.md]
下一步：[next phase]
Blockers：[any, or 無]
```

Keep it to 3-4 lines. The user already knows their project — they just need
confirmation that you're up to speed.

## Key Project Facts (stable, rarely change)

- **Hardware**: 2x SO-101 arms (Feetech STS3215), Intel RealSense D435, Jetson Orin Nano 8GB, 5x RTX 8000 training workstation
- **Framework**: HuggingFace LeRobot (Python, installed via `uv` from source)
- **Ultimate goal**: XLeRobot dual-arm mobile robot
- **Language**: 繁體中文為主，專有名詞保留英文
- **Owner**: 盧柏宇 (roy4222), experienced with Python/ROS2/Jetson/ML from PawAI project (Unitree Go2)
