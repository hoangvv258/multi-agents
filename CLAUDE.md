# Multi-Agent System — Project Rules

## Overview

This project implements a **Multi-Agent Agentic Workflow** using VS Code Copilot custom agents. The system consists of four specialized agents that collaborate through the **Coordinator and Worker** orchestration pattern using subagents.

## Architecture

```
User Request
     │
     ▼
┌─────────────┐
│ Orchestrator │  ← Entry point (user-invocable). Uses `tools: ['agent']`.
└──────┬──────┘
       │  subagent calls
       ├──▶ @Planner   → Decomposes task into steps
       ├──▶ @Worker    → Implements each step
       └──▶ @Reviewer  → Validates output, loops back to Worker if needed
```

## Agent Files

| Agent        | File                     | Invocable | Model                          | Tools                              |
| ------------ | ------------------------ | --------- | ------------------------------ | ---------------------------------- |
| Orchestrator | `orchestrator.agent.md`  | Yes       | Claude Sonnet 4, GPT-4.1      | `agent`                            |
| Planner      | `planner.agent.md`       | No        | Claude Sonnet 4, GPT-4.1      | `search/codebase`, `search/usages`, `read` |
| Worker       | `worker.agent.md`        | No        | Claude Sonnet 4, GPT-4.1      | `edit`, `run/terminal`, `search/codebase`, `read` |
| Reviewer     | `reviewer.agent.md`      | No        | Claude Sonnet 4, GPT-4.1      | `read`, `search/codebase`, `search/usages` |

## File Format

All `.agent.md` files follow the [VS Code custom agent file structure](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-structure):

```yaml
---
name: Agent Name
description: Short description
tools: ['tool1', 'tool2']
user-invocable: true|false
agents: ['SubAgent1']      # only for coordinator
handoffs:                   # optional guided transitions
  - label: Button Label
    agent: target-agent
    prompt: Pre-filled prompt
    send: false
---
# Markdown body with instructions
```

## Workflow and Handoff Flow

| Step | From         | To         | Trigger                          | Data passed                                   |
| ---- | ------------ | ---------- | -------------------------------- | --------------------------------------------- |
| 1    | User         | Orchestrator | User sends a request            | Natural language request                      |
| 2    | Orchestrator | Planner    | Request classified               | Classification, relevant files, constraints   |
| 3    | Planner      | Orchestrator | Plan complete                  | Step-by-step plan, risk assessment            |
| 4    | Orchestrator | Worker     | Plan received                    | Full execution plan from Planner              |
| 5    | Worker       | Orchestrator | Implementation done            | Changed files, test results, execution report |
| 6    | Orchestrator | Reviewer   | Implementation received          | Worker's report, plan for comparison          |
| 7a   | Reviewer     | Orchestrator | APPROVE                        | Quality score, summary, minor suggestions     |
| 7b   | Reviewer     | Worker     | REJECT (retry < 3)               | Issue list with file, line, problem, fix      |
| 8    | Orchestrator | User       | Approved or retries exhausted    | Final result or remaining issues report       |

## Conventions

- All system prompts in English for optimal AI performance.
- Subagent workers use `user-invocable: false` to stay hidden from the dropdown.
- The Orchestrator restricts subagents via `agents:` property.

## Documentation Writing Rules

All agents that produce written output (plans, reports, summaries, feedback) must follow these rules:

- Write clearly and concisely. Avoid long, complicated sentences.
- Use plain, friendly language. Write like you are explaining to a teammate, not writing a formal report.
- No emoji or icons of any kind.
- Prefer short paragraphs and bullet points over walls of text.
- Use specific, concrete language. Avoid vague words like "various", "several", or "appropriate".
- If something can be said in fewer words, use fewer words.
