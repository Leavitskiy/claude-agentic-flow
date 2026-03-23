# Set Up This Project for Claude Code

## Step 1: Analyze the Project

Before creating any files, study the project:

1. **Stack & language** — find `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `pom.xml` or similar
2. **Directory structure** — first 2–3 levels of the file tree
3. **Code conventions** — open 3–5 key files, identify: naming style, patterns, import organization
4. **Commands** — find `Makefile`, `scripts/`, `scripts` section in `package.json`, `Dockerfile`
5. **Current state** — `git log --oneline -20`, open issues, TODOs in code

Use the analysis results when filling out the files below. Don't invent — extract from actual code.

---

## Step 2: Create Required Files

### 2.1 `CLAUDE.md` (project root)

The main context file — Claude Code reads it automatically on every launch.

```markdown
# {{Project Name}}

## About
{{What this is, stack, language. 2–3 sentences derived from analysis.}}

## Current Status
**Focus:** {{current task or phase — determine from git log / issues}}

## Key Rules
{{Numbered list of 3–7 CRITICAL rules. Only things that break the project if violated. Everything else goes in .claude/rules/}}

## Project Rules
Detailed rules in `.claude/rules/`:
- `conventions.md` — code conventions, naming, file organization
- `workflow.md` — assistant workflow, post-task checklist
- `lessons-learned.md` — error log, check before working on a previously problematic component

## Common Commands
{{Real project commands in copy-paste format:}}
- Install dependencies: `{{command}}`
- Run: `{{command}}`
- Test: `{{command}}`
- Build: `{{command}}`
- Lint: `{{command}}`

## Project Structure
{{File tree, first 2–3 levels with one-line comments}}

## After Completing a Task
Update if needed:
- [ ] `CLAUDE.md` — structure/focus/rules changed
- [ ] `CHANGELOG.md` — add entry about what was done
- [ ] `.claude/rules/conventions.md` — new convention discovered
- [ ] `.claude/rules/lessons-learned.md` — error occurred → log it
```

**Important:** CLAUDE.md is context for AI, not documentation for humans. Write densely, no fluff. Don't duplicate what's visible in code — reference files instead.

---

### 2.2 `.claude/rules/conventions.md`

Conventions extracted from real code. Don't invent — if the codebase has no clear convention, don't write one.

```markdown
# Code Conventions

## File Organization
{{What goes where — derive from project structure}}

## Naming
{{Style: camelCase / snake_case / kebab-case. For files, variables, functions, classes}}

## Patterns
{{Patterns in use: how components/modules/services are organized}}

## Imports
{{Import order and grouping, if there's a clear style}}

## Don'ts
{{What NOT to do — concrete examples, not abstractions}}
```

---

### 2.3 `.claude/rules/workflow.md`

The most important file in the system — defines assistant behavior after every task.

```markdown
# Workflow

## After EVERY Completed Task

### 1. CLAUDE.md
Update if:
- New files/directories appeared
- Current focus changed
- New architectural decision was made

### 2. CHANGELOG.md
Add entry at the top:
- Format: `## YYYY-MM-DD` → list of changes
- Brief: 1–2 lines per item, state the change without implementation details

### 3. .claude/rules/
Update the relevant file if:
- New code convention → `conventions.md`
- New domain knowledge → corresponding rules file

### 4. lessons-learned.md
If an error was made — log it IMMEDIATELY (format is in the file itself).

## Before Working on a Component
Check `lessons-learned.md` — are there records of previous issues with this component?

## Forbidden
{{List of concrete prohibitions for this project. Examples:}}
- Do not commit directly to main
- Do not modify {{critical file}} without explicit request
- Do not delete tests during refactoring
```

---

### 2.4 `.claude/rules/lessons-learned.md`

```markdown
# Lessons Learned

Updated AUTOMATICALLY on every error.
Before working on a component — check if there are records about it.

<!-- New entry format:

### [YYYY-MM-DD] Brief description
- **File:** file path
- **Error:** what went wrong
- **Cause:** why
- **Fix:** what was done
- **Rule:** how to prevent in the future

-->
```

---

### 2.5 `CHANGELOG.md` (project root)

```markdown
# Changelog

## {{today's date YYYY-MM-DD}}
- Set up Claude Code project files (CLAUDE.md, .claude/rules/, CHANGELOG.md)
```

---

## Step 3: Create Optional Files (if needed)

Only create these if the project is large enough or there's a clear need.

### `PROGRESS.md` (if milestones / roadmap exist)

```markdown
# Progress

> **Current focus:** {{synced with CLAUDE.md}}

## {{Milestone 1 — Name}} ({{status}})
- [x] {{Completed task}}
- [ ] **{{Next task}}**
- [ ] {{Future task}}
```

Rules: milestones are large (week+ of work), checkboxes `[x]`/`[ ]`, completed milestones are never deleted.

### `plans/` (if major work is ahead)

```
plans/
├── README.md     # Plan index with statuses
└── {{plan}}.md   # Each plan: goal, steps, completion criteria
```

---

## Step 4: Validate

Before finishing, verify:

1. `CLAUDE.md` contains **real** project information (no placeholders left)
2. `conventions.md` is based on **existing code** (not invented)
3. `workflow.md` has a concrete checklist and concrete prohibitions for this project
4. `lessons-learned.md` is an empty template with the entry format
5. `CHANGELOG.md` has the first entry about system creation
6. All links in `CLAUDE.md` to rules files are correct
7. Commands in `CLAUDE.md` actually work (verify by running them)

---

## Step 5: Delete This File

```bash
rm claude-code-project-setup.md
```

Setup complete. Further evolution of the files happens automatically via rules in `workflow.md`.
