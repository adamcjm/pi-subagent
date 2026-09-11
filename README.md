# pi-subagent

Subagent tool for [pi](https://github.com/earendil-works/pi): delegate tasks to specialized agents, each running in an **isolated context window**.

The main agent calls `subagent`, a child `pi` process does the work, and only the final answer comes back — keeping the main context clean.

[中文文档](README.zh-CN.md)

## Features

- **Isolated context** — every subagent runs in a separate `pi` process with its own context window
- **Three modes**:
  - `single` — one agent, one task
  - `parallel` — up to 8 tasks, at most 4 running concurrently, all streaming simultaneously
  - `chain` — sequential steps, pass the previous output with the `{previous}` placeholder
- **Streaming output** — tool calls and progress render live; final output is markdown-rendered
- **Usage tracking** — turns, tokens, cost, and context usage per subagent
- **Abort support** — Ctrl+C propagates and kills subagent processes
- **Plan-mode propagation** — when the main session is in plan mode (via [pi-plan-mode](https://github.com/adamcjm/pi-plan-mode)`s persisted session state), child processes start with `--plan`; nested subagents inherit it through `PI_SUBAGENT=1`
- **Model & thinking inheritance** — an agent that does not pin a `model` inherits the parent session's model and thinking level
- **Project-trust aware** — project-agent confirmation is skipped when the project is trusted
- **Agent list in description** — available agents are listed in the tool description so the model can pick the right one
- **Agent discovery** — user agents from `~/.pi/agent/agents` and project agents from `.pi/agents`
- **Frontmatter-aware** — agent markdown supports `name`, `description`, `tools`, and `model`

## Install

```bash
pi install npm:@adamcjm/pi-subagent
```

Or for the current run only:

```bash
pi -e npm:@adamcjm/pi-subagent
```

> **Security:** extensions execute arbitrary code with full system access. Review the source before installing.

## Usage

The main agent decides when to delegate. You can also ask explicitly, e.g.:

> Use a subagent to review this diff.

The tool accepts:

| Parameter | Description |
|---|---|
| `agent` | Agent name (single mode) |
| `task` | Task text (single mode) |
| `tasks` | `[{agent, task}, ...]` for parallel execution |
| `chain` | `[{agent, task}, ...]` for sequential execution; later tasks can use `{previous}` |
| `agentScope` | `user` (default), `project`, or `both` |
| `confirmProjectAgents` | Confirm before running project-local agents (default `true`) |
| `cwd` | Working directory for the child process (single mode) |

## Agent definitions

Agents are markdown files with frontmatter:

```markdown
---
name: code-reviewer
description: 资深代码评审员……
tools: read, bash, grep, glob
model: anthropic/claude-sonnet-4-5   # optional; inherits the parent model when omitted
---

System prompt for the agent lives in the body.
```

- **User agents**: `~/.pi/agent/agents/*.md` (always available)
- **Project agents**: `.pi/agents/*.md` (opt-in via `agentScope: "project"` or `"both"`)

When several agents share a name, project agents override user agents in `both` scope.

## Notes

Built on pi's official `subagent` example extension (pi 0.85.1). `agents.ts` is kept byte-identical to the official example; `index.ts` keeps the official implementation (dispatch defaults, thinking-level inheritance, YAML frontmatter parsing, project-trust handling) and adds:

- **Plan-mode propagation** — child processes start with `--plan` when the main session is in plan mode, and nested subagents inherit it through `PI_SUBAGENT=1`
- **Agent list injection** — available agents are listed in the tool description

Agent and workflow examples (scout/planner/reviewer/worker agents, implement prompts) live in pi's repository under `examples/extensions/subagent/`.
