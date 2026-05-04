# Claude Agentic Flow

A collection of specialized agents, skills, and flows for Claude Code that implement an efficient workflow for software development. Agents plan, implement, and review code changes; skills automate quality checks on demand; flows document the methodology behind them.

## Why This Workflow?

Instead of asking Claude to do everything at once, this workflow separates concerns:

1. **Planning agents** analyze the codebase and create detailed implementation plans
2. **Implementation agents** execute the plan with deep domain expertise
3. **Review agents** validate the output before it reaches human review
4. **Skills** automate specific quality checks the user invokes on demand (e.g. mutation testing after writing tests)

This separation leads to better results because each agent focuses on what it does best. It also **reduces context usage** in your main Claude window — agents run in their own context, so you can complete more tasks before hitting conversation compacting.

## Repository Overview

```
├── backend/
│   ├── backend-feature-designer.md   # Plans backend features
│   └── python-backend-engineer.md    # Implements Python backend code
├── frontend/
│   ├── frontend-feature-designer.md  # Plans frontend features
│   ├── react-native-engineer.md      # Implements React Native code
│   └── ui-engineer.md                # Implements frontend code
├── shared/
│   ├── agent-code-reviewer.md        # Reviews AI-generated code
│   └── refactoring-planner.md        # Plans refactoring tasks
├── skills/
│   └── verify-tests.md               # Validates tests via mutation testing
└── flows/
    └── test-quality-loop.md          # Methodology behind verify-tests
```

The repo holds three kinds of artifact: **agents** (subagents the main Claude invokes via the Task tool), **skills** (slash-commands the user invokes directly), and **flows** (methodology playbooks — the *why* behind the skills).

### Planning Agents

- **backend-feature-designer** — Analyzes existing backend patterns and creates comprehensive implementation specifications with API contracts, data models, and phased roadmaps
- **frontend-feature-designer** — Analyzes existing frontend patterns and creates component hierarchies, state models, and implementation plans with focus on reusability and accessibility

### Implementation Agents

- **python-backend-engineer** — Senior Python engineer specializing in FastAPI, Django, SQLAlchemy, async programming, and modern tooling like uv
- **ui-engineer** — Expert frontend engineer for React, TypeScript, modern CSS, state management, and accessibility

### Shared Agents

- **agent-code-reviewer** — Quality gate that validates AI-generated code for hallucinations, context blindness, and project consistency
- **refactoring-planner** — Analyzes code problems and creates actionable refactoring plans with metrics and risk assessment

## Skills

User-invoked slash commands that automate specific quality checks. Skills run on demand (`/verify-tests`) rather than being delegated by the main agent.

- **verify-tests** — Runs mutation testing on changed source files and analyzes surviving mutants as concrete test gaps. Tool-agnostic: detects the project's ecosystem (`package.json` / `pyproject.toml` / `pom.xml` / `*.csproj`) and dispatches to the appropriate runner (StrykerJS, mutmut, PIT, Stryker.NET). Use after writing tests, before committing — coverage gates miss tests that execute code without asserting its behavior; this skill catches them. Methodology: [`flows/test-quality-loop.md`](flows/test-quality-loop.md).

### Skills Installation

```bash
mkdir -p .claude/skills
cp path/to/claude-agentic-flow/skills/* .claude/skills/
```

Then invoke from your Claude Code prompt:

```
/verify-tests
```

## Flows

Methodology playbooks. The skills automate execution; flows explain the protocol so you can adapt it, integrate with CI, and decide when *not* to apply it.

- [`flows/test-quality-loop.md`](flows/test-quality-loop.md) — Why coverage isn't enough, how mutation testing fills the gap, how to read mutation reports, equivalent-mutants triage, realistic targets (75–85% mutation score, not 100%), and three CI integration patterns (warn-only → diff-line gate → trend gate).

Flows live in this repo as reference material. They aren't copied into your project — read them once and adapt your `CLAUDE.md` / `workflow.md` accordingly.

## Project Setup (Self-Improving Documentation)

The [`claude-code-project-setup.md`](claude-code-project-setup.md) file is a ready-to-use instruction that configures any project for effective work with Claude Code. It creates a system of files that gives the AI full project context on every launch, auto-updates after each task, and prevents repeated mistakes.

### What It Creates

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Main context file — Claude Code reads it automatically on every launch |
| `.claude/rules/conventions.md` | Code conventions extracted from your actual codebase |
| `.claude/rules/workflow.md` | Post-task checklist — what to update after every completed task |
| `.claude/rules/lessons-learned.md` | Error log — auto-filled on mistakes, checked before working on problem areas |
| `CHANGELOG.md` | Human-readable change log (not a git log replacement) |
| `PROGRESS.md` | *(optional)* Milestone tracking for larger projects |
| `plans/` | *(optional)* Design documents and implementation plans |

### How to Use

**Option A — copy the file into your project:**

```bash
# From inside your target project
cp path/to/claude-agentic-flow/claude-code-project-setup.md .

# Then launch Claude Code and say:
# "Follow the instructions in claude-code-project-setup.md"
```

**Option B — pipe it directly:**

```bash
claude "$(cat path/to/claude-agentic-flow/claude-code-project-setup.md)"
```

**Option C — reference it from the conversation:**

```
Read claude-code-project-setup.md from /path/to/claude-agentic-flow/ and follow its instructions for this project.
```

Claude will analyze your project (stack, structure, conventions, commands), create all the files filled with real data, validate them, and prompt you to delete the setup file.

### How It Works After Setup

The `workflow.md` rule creates a self-improving loop:

1. **Every task** → Claude updates `CLAUDE.md`, `CHANGELOG.md`, and rules as needed
2. **Every mistake** → auto-logged in `lessons-learned.md` with cause and prevention rule
3. **Before working on a component** → Claude checks `lessons-learned.md` for prior issues
4. **New convention discovered** → added to `conventions.md`, followed in all future tasks

Over time, the project accumulates knowledge that persists across sessions and team members.

---

## Agents Installation

Copy the agent files to your project's `.claude/agents/` directory:

```bash
mkdir -p .claude/agents

# Copy agents you need
cp path/to/claude-agentic-flow/backend/* .claude/agents/
cp path/to/claude-agentic-flow/frontend/* .claude/agents/
cp path/to/claude-agentic-flow/shared/* .claude/agents/
```

## Agents Usage

### Option 1: Explicit Instructions

Tell Claude which agents to use directly:

```
Use the backend-feature-designer agent to create a plan for user authentication,
then use python-backend-engineer to implement it,
and finally use agent-code-reviewer to validate the code.
```

### Option 2: CLAUDE.md Configuration

Add instructions to your project's `CLAUDE.md` file to automate agent selection:

```markdown
## Agent Workflow

When working on this project, follow this workflow:

### For new backend features:
1. First use `backend-feature-designer` to create implementation plan
2. Then use `python-backend-engineer` to implement the plan (with tests)
3. Use `agent-code-reviewer` to validate the code
4. Run `/verify-tests` to validate the new tests via mutation testing

### For new frontend features:
1. First use `frontend-feature-designer` to create implementation plan
2. Then use `ui-engineer` to implement the plan (with tests)
3. Use `agent-code-reviewer` to validate the code
4. Run `/verify-tests` to validate the new tests via mutation testing

### For refactoring tasks:
1. Use `refactoring-planner` to analyze and create a plan
2. Implement changes incrementally
3. Use `agent-code-reviewer` after each significant change
4. Run `/verify-tests` if test code changed — refactors that don't change observable behavior should not change the mutation score
```

## Example Workflow

```
You: I need to add a user notifications feature to the backend

Claude: I'll use the backend-feature-designer agent to create an implementation plan.

[Agent analyzes codebase, creates /outputs/implementation_plan_notifications.md]

Claude: The plan is ready. Should I proceed with implementation?

You: Yes, implement it

Claude: I'll use the python-backend-engineer agent to implement the notifications feature.

[Agent implements the feature following the plan - this may take 15-45 minutes for complex features]

Claude: Implementation complete. Let me validate it with the code reviewer.

[agent-code-reviewer checks for hallucinations, missing imports, project consistency]

Claude: Review complete. Found 2 minor issues that I've fixed. Now I'll run /verify-tests to validate the new tests via mutation testing.

[/verify-tests detects ecosystem, scopes Stryker to changed files, parses surviving mutants]

Claude: Mutation testing found 1 surviving mutant in the notification-batching helper —
the test asserts toHaveBeenCalled() but not the argument value. Added a
toHaveBeenCalledWith() assertion; re-ran; all mutants killed. Feature is ready.
```

## Tips

### Execution Time

Planning and implementation agents can run for extended periods on complex tasks. My personal record is **45 minutes** for a single implementation session. This is normal — the agent is being thorough.

### Planning Outputs

All planning agents create detailed markdown files in `/outputs/` directory. Review these plans before proceeding with implementation — it's easier to adjust a plan than to refactor implemented code.

### Greenfield Projects

The agents handle empty repositories well. When no existing code is found, they propose industry best practices and standard patterns instead of trying to match non-existent conventions.

### Environment Variables

The planning agents explicitly warn about hardcoded values. All environment-specific configuration (URLs, API keys, feature flags) should come from environment variables with clear documentation for dev/staging/prod.

### Skills runtime

Skills like `/verify-tests` may take minutes (mutation testing scopes to the diff but still runs the test suite per mutant). Run them locally before commit, not as a pre-commit hook. Install only the runner that matches your stack — the skill detects which one is needed and bails out gracefully if the runner isn't installed.

## Creating Custom Agents

For information on creating your own agents, see the [official Claude Code documentation](https://code.claude.com/docs/en/sub-agents).

## Attribution

Some agents in this collection are based on work by [@hesreallyhim](https://github.com/hesreallyhim) from [a-list-of-claude-code-agents](https://github.com/hesreallyhim/a-list-of-claude-code-agents).

## License

MIT
