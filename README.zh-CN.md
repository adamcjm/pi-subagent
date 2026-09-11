# pi-subagent

[pi](https://github.com/earendil-works/pi) 的子代理工具：把任务委派给专用 agent，每个 agent 运行在**独立上下文窗口**中。

主 agent 调用 `subagent`，子 `pi` 进程干活，只有最终结果返回主会话——主上下文保持干净。

[English](README.md)

## 特性

- **上下文隔离** —— 每个子代理运行在独立 `pi` 进程中，拥有自己的上下文窗口
- **三种模式**：
  - `single` —— 单个 agent、单个任务
  - `parallel` —— 最多 8 个任务、最多 4 个并发，全部同时流式输出
  - `chain` —— 顺序执行，后续任务可用 `{previous}` 引用上一步输出
- **流式输出** —— 工具调用与进度实时渲染，最终结果按 markdown 渲染
- **用量统计** —— 每个子代理的轮数、token、费用和上下文占用
- **中止支持** —— Ctrl+C 会传播并终止子代理进程
- **Plan 模式传播** —— 主会话处于 plan mode 时（读取 [pi-plan-mode](https://github.com/adamcjm/pi-plan-mode) 的会话状态），子进程以 `--plan` 启动；嵌套子代理通过 `PI_SUBAGENT=1` 继承
- **模型与思考级别继承** —— agent 未固定 `model` 时，继承父会话当前模型与 thinking level
- **项目信任感知** —— 项目已信任时，不再弹出项目级 agent 确认
- **工具描述列出 agent** —— 可用 agent 列表写入工具描述，便于模型选择
- **Agent 发现** —— 用户级 `~/.pi/agent/agents` 与项目级 `.pi/agents`
- **Frontmatter 支持** —— agent markdown 支持 `name`、`description`、`tools`、`model`

## 安装

```bash
pi install npm:@adamcjm/pi-subagent
```

仅当前运行使用：

```bash
pi -e npm:@adamcjm/pi-subagent
```

> **安全提示：** 扩展以完整系统权限执行任意代码，安装前请审阅源码。

## 使用

主 agent 自行判断何时委派，也可以直接要求，例如：

> 用子代理 review 一下这个 diff。

工具参数：

| 参数 | 说明 |
|---|---|
| `agent` | agent 名称（single 模式） |
| `task` | 任务文本（single 模式） |
| `tasks` | `[{agent, task}, ...]`，并行执行 |
| `chain` | `[{agent, task}, ...]`，顺序执行；后续任务可用 `{previous}` |
| `agentScope` | `user`（默认）、`project` 或 `both` |
| `confirmProjectAgents` | 运行项目级 agent 前是否确认（默认 `true`） |
| `cwd` | 子进程工作目录（single 模式） |

## Agent 定义

Agent 是带 frontmatter 的 markdown 文件：

```markdown
---
name: code-reviewer
description: 资深代码评审员……
tools: read, bash, grep, glob
model: anthropic/claude-sonnet-4-5   # 可选；省略时继承父会话模型
---

正文即该 agent 的 system prompt。
```

- **用户级 agent**：`~/.pi/agent/agents/*.md`（始终可用）
- **项目级 agent**：`.pi/agents/*.md`（需 `agentScope: "project"` 或 `"both"` 启用）

同名时，`both` 作用域下项目级 agent 覆盖用户级 agent。

## 说明

基于 pi 官方 `subagent` 示例扩展（pi 0.85.1）构建：`agents.ts` 与官方保持逐字节一致；`index.ts` 保留官方实现（dispatch 默认值、thinking level 继承、YAML frontmatter 解析、项目信任处理），并增加：

- **plan 模式传播** —— 主会话处于 plan mode 时，子进程以 `--plan` 启动；嵌套子代理通过 `PI_SUBAGENT=1` 继承
- **agent 列表注入** —— 把可用 agent 列表写入工具描述

官方示例 agent 与 workflow 提示（scout/planner/reviewer/worker、implement 系列）位于 pi 仓库的 `examples/extensions/subagent/`。
