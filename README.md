# Claude Agentic Flow

A collection of specialized agents for Claude Code that implement an efficient workflow for software development. These agents work together to plan, implement, and review code changes.

## Why This Workflow?

Instead of asking Claude to do everything at once, this workflow separates concerns:

1. **Planning agents** analyze the codebase and create detailed implementation plans
2. **Implementation agents** execute the plan with deep domain expertise
3. **Review agents** validate the output before it reaches human review

This separation leads to better results because each agent focuses on what it does best. It also **reduces context usage** in your main Claude window — agents run in their own context, so you can complete more tasks before hitting conversation compacting.

## Agents Overview

```
├── backend/
│   ├── backend-feature-designer.md   # Plans backend features
│   └── python-backend-engineer.md    # Implements Python backend code
├── frontend/
│   ├── frontend-feature-designer.md  # Plans frontend features
│   └── ui-engineer.md                # Implements frontend code
└── shared/
    ├── agent-code-reviewer.md        # Reviews AI-generated code
    └── refactoring-planner.md        # Plans refactoring tasks
```

### Planning Agents

- **backend-feature-designer** — Analyzes existing backend patterns and creates comprehensive implementation specifications with API contracts, data models, and phased roadmaps
- **frontend-feature-designer** — Analyzes existing frontend patterns and creates component hierarchies, state models, and implementation plans with focus on reusability and accessibility

### Implementation Agents

- **python-backend-engineer** — Senior Python engineer specializing in FastAPI, Django, SQLAlchemy, async programming, and modern tooling like uv
- **ui-engineer** — Expert frontend engineer for React, TypeScript, modern CSS, state management, and accessibility

### Shared Agents

- **agent-code-reviewer** — Quality gate that validates AI-generated code for hallucinations, context blindness, and project consistency
- **refactoring-planner** — Analyzes code problems and creates actionable refactoring plans with metrics and risk assessment

## Installation

Copy the agent files to your project's `.claude/agents/` directory:

```bash
# Create agents directory if it doesn't exist
mkdir -p .claude/agents

# Copy agents you need
cp path/to/claude-agentic-flow/backend/* .claude/agents/
cp path/to/claude-agentic-flow/frontend/* .claude/agents/
cp path/to/claude-agentic-flow/shared/* .claude/agents/
```

## Usage

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
2. Then use `python-backend-engineer` to implement the plan
3. Finally use `agent-code-reviewer` to validate the code

### For new frontend features:
1. First use `frontend-feature-designer` to create implementation plan
2. Then use `ui-engineer` to implement the plan
3. Finally use `agent-code-reviewer` to validate the code

### For refactoring tasks:
1. Use `refactoring-planner` to analyze and create a plan
2. Implement changes incrementally
3. Use `agent-code-reviewer` after each significant change
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

Claude: Review complete. Found 2 minor issues that I've fixed. The feature is ready.
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

## Creating Custom Agents

For information on creating your own agents, see the [official Claude Code documentation](https://code.claude.com/docs/en/sub-agents).

## Attribution

Some agents in this collection are based on work by [@hesreallyhim](https://github.com/hesreallyhim) from [a-list-of-claude-code-agents](https://github.com/hesreallyhim/a-list-of-claude-code-agents).

## License

MIT
