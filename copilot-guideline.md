# GitHub Copilot Guideline -- Agent and Chat

Guideline for using GitHub Copilot in VS Code, focused on agents and chat features. Based on the official VS Code documentation.

---

## 1. Chat Basics

### How to Access Chat

- Press `Ctrl+Alt+I` (Mac: `Cmd+Option+I`) to open the Chat view.
- Select **Chat** from the VS Code title bar.
- Use Quick Chat for fast, inline questions.
- Run `code chat` from the terminal CLI.

### Session Configuration

Each chat session is shaped by four choices:

| Setting          | What it controls                                     |
| ---------------- | ---------------------------------------------------- |
| Session type     | Where the agent runs: Local, Background, or Cloud    |
| Agent            | The role/persona: Agent, Plan, or Ask                |
| Permission level | How much autonomy the agent has over tool approvals  |
| Language model   | Which AI model powers the conversation               |

---

## 2. Built-in Agents

### Agent Mode

Best for complex coding tasks. The AI works autonomously -- determines context, plans the work, edits files, runs terminal commands, and iterates on problems.

Use when: multi-file edits, building features, debugging, refactoring.

### Plan Mode

Best for creating a structured implementation plan before coding. Breaks down a complex task into smaller steps. You can hand off the plan to Agent mode for implementation.

Use when: unfamiliar codebase, complex features, team planning.

### Ask Mode

Best for questions and research. Uses agentic capabilities to search your codebase and gather context. Responses include code blocks you can apply individually.

Use when: understanding code, exploring ideas, getting explanations.

---

## 3. Adding Context to Prompts

### Implicit Context

VS Code automatically includes: active file, current selection, file name. In Agent mode, the agent gathers additional context on its own.

### Explicit Context (#-mentions)

Use `#` in the chat input to reference specific context:

| Syntax              | What it references                      |
| ------------------- | --------------------------------------- |
| `#file`             | A specific file in your workspace       |
| `#folder`           | An entire folder                        |
| `#symbol`           | A function, class, or variable          |
| `#codebase`         | Your full codebase (semantic search)    |
| `#terminalSelection`| Current terminal output                 |
| `#fetch`            | Content from a URL                      |
| `#problems`         | Current linting/diagnostic issues       |

### @-mentions

Use `@` to invoke specialized participants:

- `@vscode` -- questions about VS Code settings and features.
- `@terminal` -- questions about terminal and shell commands.

### Visual Context

- Attach images or screenshots for visual analysis.
- Use the integrated browser to select page elements as context.

---

## 4. Agent Tools

### Enabling Tools

1. Open Chat view and select **Agent** from the agent picker.
2. Click the **Configure Tools** button in the chat input.
3. Select or deselect tools for the current request.

Select only relevant tools -- fewer active tools means faster, more accurate responses.

### Referencing Tools Explicitly

Type `#` followed by the tool name to force a specific tool:

- `"What is the latest version of Node.js #web"`
- `"Fix the issues in #problems"`
- `"Explain the authentication flow #codebase"`

### Permission Levels

| Level           | Behavior                                                |
| --------------- | ------------------------------------------------------- |
| Default         | Ask approval for each tool call, file edit, and command |
| Bypass Approvals| Auto-approve all tool calls without prompts             |
| Autopilot       | Continuous autonomous work, auto-retry, auto-respond    |

Autopilot keeps working until it determines the task is complete. No manual approval needed. Use only when you trust the task scope.

### Tool Sets

Group related tools under a single name. Built-in examples: `#edit`, `#search`. You can create custom tool sets in your project.

---

## 5. Custom Agents

### What They Are

Custom agents are `.agent.md` files that define a specific role with its own instructions, tools, and model. Store them in `.github/agents/` (workspace) or `~/.copilot/agents/` (user-level).

### File Structure

```yaml
---
name: Agent Name
description: What this agent does
tools: ['tool1', 'tool2']
model: 'Claude Sonnet 4'
handoffs:
  - label: Button Label
    agent: target-agent
    prompt: Pre-filled prompt for the next step.
    send: false
---
# Markdown body with instructions
```

### Key Frontmatter Fields

| Field                   | Purpose                                              |
| ----------------------- | ---------------------------------------------------- |
| `name`                  | Display name in the agents dropdown                  |
| `description`           | Short description of the agent's role                |
| `tools`                 | List of tools the agent can use                      |
| `model`                 | Which AI model to use                                |
| `user-invocable`        | `false` to hide from dropdown (subagent only)        |
| `agents`                | Restrict which subagents this agent can invoke       |
| `handoffs`              | Define guided workflow transitions to other agents   |
| `disable-model-invocation` | Prevent this agent from being auto-invoked as subagent |

### Handoffs

Handoffs create guided sequential workflows. After a response completes, handoff buttons appear to transition to the next agent with context.

Example: Planning agent finishes, shows "Start Implementation" button that sends the plan to an Implementation agent.

```yaml
handoffs:
  - label: Start Implementation
    agent: implementation
    prompt: Implement the plan above.
    send: false   # false = pre-fill only, true = auto-submit
```

---

## 6. Subagents

### What They Are

Subagents run as isolated child tasks within a chat session. They have their own context, tools, and model. Results are reported back to the parent agent.

### Invoking Subagents

- **Agent-initiated**: The AI decides to invoke a subagent based on the task.
- **User-invoked**: You prompt the AI to use a specific subagent.

Example prompts:
- "Run the Research agent as a subagent to research auth methods."
- "Use the Plan agent in a subagent to create a plan for this feature."

### Controlling Subagent Access

- `user-invocable: false` -- hide from dropdown, only usable as subagent.
- `disable-model-invocation: true` -- prevent any agent from auto-invoking this one.
- `agents: ['Agent1', 'Agent2']` -- restrict which subagents a coordinator can use.

### Orchestration Patterns

**Coordinator and Worker Pattern**

A coordinator agent manages the workflow and delegates to specialized workers:

```yaml
---
name: Feature Builder
tools: ['agent', 'edit', 'search', 'read']
agents: ['Planner', 'Implementer', 'Reviewer']
---
For each feature request:
1. Use the Planner agent to break down the feature.
2. Use the Implementer agent to write the code.
3. Use the Reviewer agent to check the result.
4. Iterate until the Reviewer approves.
```

Worker agents define their own narrower tool sets:

```yaml
---
name: Planner
user-invocable: false
tools: ['read', 'search']
---
Break down feature requests into implementation tasks.
```

---

## 7. Custom Instructions

### Types of Instruction Files

| File                               | Scope                        | Applied when                  |
| ---------------------------------- | ---------------------------- | ----------------------------- |
| `.github/copilot-instructions.md`  | Entire workspace             | Every chat request            |
| `AGENTS.md`                        | Entire workspace             | Every chat request            |
| `CLAUDE.md`                        | Entire workspace             | Every chat request            |
| `*.instructions.md`               | Pattern-matched files/tasks  | When working on matching files |

### Priority Order (highest to lowest)

1. Personal instructions (user-level)
2. Repository instructions (`copilot-instructions.md` or `AGENTS.md`)
3. Organization instructions

### Tips for Writing Instructions

- Keep each instruction short and self-contained.
- Include reasoning behind rules. Example: "Use date-fns instead of moment.js because moment.js is deprecated."
- Show preferred and avoided patterns with code examples.
- Focus on non-obvious rules. Skip what linters already enforce.
- Use `applyTo` patterns for language-specific or folder-specific rules.

---

## 8. Prompt Files

### What They Are

Reusable prompt templates stored as `.prompt.md` files. They pre-configure agent, model, tools, and instructions for a specific task.

### File Format

```yaml
---
agent: 'agent'
model: 'Claude Sonnet 4'
tools: ['search/codebase', 'web/fetch']
description: 'Generate a React form component'
---
Your goal is to generate a new React form component.
Requirements:
* Use react-hook-form for state management
* Define TypeScript types for form data
* Use yup for validation
```

### Using Prompt Files

Type `/` in the chat input to see available prompt files. Select one to pre-fill the prompt with its content and configuration.

---

## 9. Cheat Sheet

### Keyboard Shortcuts

| Action                     | Mac                       | Windows / Linux              |
| -------------------------- | ------------------------- | ---------------------------- |
| Open Chat view             | `Cmd+Option+I`            | `Ctrl+Alt+I`                 |
| Voice chat prompt          | `Cmd+I`                   | `Ctrl+I`                     |
| New chat session           | `Cmd+N`                   | `Ctrl+N`                     |
| Switch to Agent mode       | `Shift+Cmd+I`             | `Ctrl+Shift+I`               |
| Inline chat (editor/terminal) | `Cmd+I`               | `Ctrl+I`                     |
| Accept inline suggestion   | `Tab`                     | `Tab`                        |
| Dismiss inline suggestion  | `Escape`                  | `Escape`                     |
| Quick Chat                 | `Shift+Option+Cmd+L`     | `Ctrl+Shift+Alt+L`           |

### Built-in Tools

Agent mode provides these built-in tools (reference with `#tool-name`):

| Tool Set     | Tools                                                                    |
| ------------ | ------------------------------------------------------------------------ |
| `#agent`     | `runSubagent` -- delegate to subagents                                   |
| `#browser`   | Interact with integrated browser                                         |
| `#edit`      | `createDirectory`, `createFile`, `editFiles`, `editNotebook`             |
| `#execute`   | `runInTerminal`, `getTerminalOutput`, `createAndRunTask`, `testFailure`  |
| `#read`      | `readFile`, `problems`, `terminalLastCommand`, `terminalSelection`       |
| `#search`    | `codebase`, `changes`, `fileSearch`, `textSearch`, `listDirectory`, `usages` |
| `#web`       | `fetch` -- retrieve content from URLs                                    |
| `#selection` | Current editor selection                                                 |
| `#todos`     | TODO comments in workspace                                               |
| `#vscode`    | `askQuestions`, `extensions`, `runCommand`, `installExtension`, `VSCodeAPI` |
| `#newWorkspace` | Scaffold a new project                                                |

### Slash Commands

| Command             | What it does                                      |
| ------------------- | ------------------------------------------------- |
| `/doc`              | Generate documentation                            |
| `/explain`          | Explain selected code                             |
| `/fix`              | Fix a problem in the code                         |
| `/tests`            | Generate tests                                    |
| `/setupTests`       | Set up testing framework                          |
| `/clear`            | Clear chat history                                |
| `/compact`          | Compact context to free up space                  |
| `/fork`             | Fork current chat session                         |
| `/new`              | Scaffold a new project                            |
| `/newNotebook`      | Create a new Jupyter notebook                     |
| `/init`             | Generate starter config files                     |
| `/plan`             | Create an implementation plan                     |
| `/search`           | Search workspace                                  |
| `/startDebugging`   | Generate launch.json and start debugging          |
| `/debug`            | Open chat debug logs                              |
| `/troubleshoot`     | Troubleshoot agent issues                         |
| `/agents`           | List custom agents                                |
| `/hooks`            | Manage agent hooks                                |
| `/instructions`     | Manage custom instructions                        |
| `/prompts`          | List prompt files                                 |
| `/skills`           | List agent skills                                 |
| `/create-agent`     | Generate a custom agent file                      |
| `/create-prompt`    | Generate a prompt file                            |
| `/create-instruction` | Generate an instructions file                  |
| `/create-skill`     | Generate an agent skill                           |
| `/create-hook`      | Generate a hook                                   |
| `/yolo`             | Enable auto-approval for current session          |
| `/disableYolo`      | Disable auto-approval                             |
| `/memory`           | Manage agent memory                               |

### Chat Participants

| Participant   | Purpose                                           | Example                                        |
| ------------- | ------------------------------------------------- | ---------------------------------------------- |
| `@github`     | GitHub skills: PRs, issues, repos, search         | `@github What are all open PRs assigned to me?`|
| `@terminal`   | Terminal/shell commands and explanations           | `@terminal list the 5 largest files`            |
| `@vscode`     | VS Code settings, features, and configuration     | `@vscode how to enable word wrapping?`          |

---

## 10. Settings Reference

Key settings for configuring Copilot chat and agents. Open Settings (`Cmd+,`) and search for the setting name.

### General

| Setting                              | Default  | What it controls                               |
| ------------------------------------ | -------- | ---------------------------------------------- |
| `github.copilot.enable`             | `true`   | Enable/disable Copilot globally                |
| `chat.agent.enabled`                | `true`   | Enable agent mode in chat                      |
| `chat.agent.maxRequests`            | `25`     | Max tool calls per agent request               |

### Chat

| Setting                              | Default      | What it controls                           |
| ------------------------------------ | ------------ | ------------------------------------------ |
| `chat.editor.defaultMode`           | `"chatView"` | Default chat surface                       |
| `chat.checkpoints.enabled`          | `true`       | Enable automatic checkpoints               |
| `chat.editRequest.enabled`          | `false`      | Allow editing previous chat messages       |
| `chat.followUp.enabled`             | `true`       | Show follow-up suggestions                 |
| `chat.renderMathInMarkdown`         | `false`      | Render KaTeX math formulas                 |
| `chat.followUpMessage.behavior`     | `"queue"`    | What happens when you send while running: `queue` or `steer` |

### Agent

| Setting                              | Default  | What it controls                               |
| ------------------------------------ | -------- | ---------------------------------------------- |
| `chat.agent.enabled`                | `true`   | Enable agent capabilities                      |
| `chat.agent.tools.terminal.autoApprove` | `true` | Auto-approve terminal commands              |
| `chat.agent.tools.terminal.denylist`| _(dangerous cmds)_ | Commands that are never auto-approved |
| `chat.autopilot.enabled`            | `true`   | Enable Autopilot permission level              |
| `chat.agent.sandbox.enabled`        | `false`  | Sandbox agent terminal commands                |
| `chat.agent.maxTools`               | `128`    | Max tools per request                          |

### Custom Instructions

| Setting                              | Default  | What it controls                               |
| ------------------------------------ | -------- | ---------------------------------------------- |
| `chat.instructionFilesLocations`    | `{ ".github/instructions": true }` | Where to find .instructions.md files |
| `chat.copilotInstructionsEnabled`   | `true`   | Enable copilot-instructions.md                 |
| `chat.agentsMd.enabled`             | `true`   | Enable AGENTS.md files                         |
| `chat.claudeMd.enabled`             | `true`   | Enable CLAUDE.md files                         |

### Custom Agents and Prompt Files

| Setting                              | Default  | What it controls                               |
| ------------------------------------ | -------- | ---------------------------------------------- |
| `chat.agentsLocations`              | `{ ".github/agents": true }` | Where to find .agent.md files      |
| `chat.promptFilesLocations`         | `{ ".github/prompts": true }` | Where to find .prompt.md files    |
| `chat.subagents.enabled`            | `false`  | Enable subagent invocation                     |

### Memory and Observability

| Setting                              | Default  | What it controls                               |
| ------------------------------------ | -------- | ---------------------------------------------- |
| `chat.memory.enabled`               | `true`   | Enable agent memory tool                       |
| `chat.agent.opentelemetry.enabled`  | `false`  | Enable OpenTelemetry tracing for agents        |

---

## 12. Reference Links

| Topic              | URL                                                                                              |
| -------------------| ------------------------------------------------------------------------------------------------ |
| Chat overview      | https://code.visualstudio.com/docs/copilot/chat/copilot-chat                                    |
| Local agents       | https://code.visualstudio.com/docs/copilot/agents/local-agents                                  |
| Agent tools        | https://code.visualstudio.com/docs/copilot/agents/agent-tools                                   |
| Custom agents      | https://code.visualstudio.com/docs/copilot/customization/custom-agents                          |
| Subagents          | https://code.visualstudio.com/docs/copilot/agents/subagents                                     |
| Custom instructions| https://code.visualstudio.com/docs/copilot/customization/custom-instructions                    |
| Prompt files       | https://code.visualstudio.com/docs/copilot/customization/prompt-files                           |
| Cheat sheet        | https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features                    |
| Settings reference | https://code.visualstudio.com/docs/copilot/reference/copilot-settings                           |
