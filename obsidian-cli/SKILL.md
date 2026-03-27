---
name: obsidian-cli
description: Use Obsidian CLI for project-first agent memory, including project/task Kanban updates, task lists, and durable memory notes.
license: MIT
compatibility: opencode
metadata:
  audience: engineers
  tool: obsidian
---

## What I do

- Use Obsidian from the terminal for scripting and automation.
- Build correct commands with `parameter=value` syntax and boolean flags.
- Target the right vault and file before running note operations.
- Maintain a project-first structure with top-level and per-project Kanban boards.

## When to use me

- The user asks to manage notes from terminal.
- You need repeatable project/task tracking and long-running memory.
- You need quick command examples without opening the Obsidian UI.

## Recommended structure

```text
L-Space/
  Projects.md
  Projects-Kanban.md
  Projects-Archive.md
  Projects/
    Archive/
    <project-slug>/
      README.md
      Tasks/
      Kanban.md
      Memory.md
      Log.md
```

- `Projects.md` is the master project list.
- `Projects-Kanban.md` is the top-level project board.
- `Projects-Archive.md` records archived projects.
- Each project folder contains its own task board, task notes folder, and memory.

## Prerequisites

- Obsidian 1.12 or later.
- CLI enabled in Obsidian: `Settings -> General -> Command line interface`.
- Obsidian app is running (first command can launch it).

## Quick reference

```bash
# Get help
obsidian help

# Open interactive TUI
obsidian

# Create notes
obsidian create
obsidian create name=Note content="Hello world"
obsidian create name=Note content="# Title\n\nBody text" open overwrite

# Target a specific vault (vault must be first parameter)
obsidian vault=Notes daily
obsidian vault="My Vault" search query="test"

# Create a project note file (repeat for each file needed)
obsidian vault=Work create name="L-Space/Projects/New-Project/README" content="# New Project"
obsidian vault=Work create name="L-Space/Projects/New-Project/Kanban" content="# Kanban"
obsidian vault=Work create name="L-Space/Projects/New-Project/Memory" content="# Memory"
obsidian vault=Work create name="L-Space/Projects/New-Project/Log" content="# Log"
```

## Parameter rules

- Parameters use `key=value`.
- Quote values with spaces, for example `name="My Note"`.
- Flags are bare words with no value, for example `open` and `overwrite`.
- Use `\n` for newlines and `\t` for tabs in content.

## Targeting rules

- If terminal CWD is a vault folder, commands target that vault.
- Otherwise, commands target the currently active vault.
- Use `vault=<name>` or `vault=<id>` to force a target vault.
- Use `file=<name>` for wikilink-style resolution or `path=<vault/relative/path.md>` for exact file targeting.

## Common mistakes

- Putting `vault=` after the command instead of before it.
- Forgetting to quote parameter values that contain spaces.
- Treating flags like parameters (`open=true`) instead of bare flags (`open`).
- Using `file=` when an exact `path=` is required.

## Kanban and task sync rules

- Update project status in both `L-Space/Projects.md` and `L-Space/Projects-Kanban.md`.
- Update task status in `L-Space/Projects/<project-slug>/Kanban.md` and the linked `L-Space/Projects/<project-slug>/Tasks/<task-slug>.md` note.
- Keep statuses aligned to: `todo`, `in_progress`, `blocked`, `done`.
- Keep memory append-only in `L-Space/Projects/<project-slug>/Memory.md`.
- Keep each task note updated with what was actually implemented under an `Implemented` section.
- If CLI cannot perform a specific edit cleanly, update markdown directly while preserving structure.

## Archive rule (projects only)

- Apply archiving only to `L-Space/Projects-Kanban.md`.
- If `done` has more than 10 project cards, archive oldest done projects until 10 remain.
- For each archived project:
  - Move `L-Space/Projects/<project-slug>/` to `L-Space/Projects/Archive/<project-slug>/`.
  - Remove it from `L-Space/Projects.md` active table.
  - Remove its card from `L-Space/Projects-Kanban.md`.
  - Add an entry to `L-Space/Projects-Archive.md`.
- Do not apply this rule to per-project task boards (`L-Space/Projects/<project-slug>/Kanban.md`).
