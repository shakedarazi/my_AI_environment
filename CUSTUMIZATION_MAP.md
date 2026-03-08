# AI Customization Map

This document maps the control layers for GitHub Copilot and Claude Code in VS Code, with the focus on:

- global vs repo scope
- what injects context
- what changes behavior without injecting prompt text
- where hooks, prompts, skills, agents, and instructions live
- which commands open or create each customization type

## 1. Global vs Repo Matrix

| Scope  | Copilot                                                                                        | Claude Code                                                                                                                                                                                          |
| ------ | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Global | VS Code user settings: `C:\Users\shaked arazi\AppData\Roaming\Code\User\settings.json`         | Claude global instructions: `C:\Users\shaked arazi\.claude\CLAUDE.md`                                                                                                                                |
| Global | Personal Copilot instructions: `C:\Users\shaked arazi\.copilot\instructions\*.instructions.md` | Claude global rules: `C:\Users\shaked arazi\.claude\rules\*.md`                                                                                                                                      |
| Global | Personal prompt files: `C:\Users\shaked arazi\AppData\Roaming\Code\User\prompts\*.prompt.md`   | Claude settings and hooks: `C:\Users\shaked arazi\.claude\settings.json`                                                                                                                             |
| Global | Personal custom agents: `C:\Users\shaked arazi\AppData\Roaming\Code\User\prompts\*.agent.md`   | Claude plugins: `C:\Users\shaked arazi\.claude\plugins\`                                                                                                                                             |
| Global | Personal skills: `C:\Users\shaked arazi\.copilot\skills\<skill>\SKILL.md`                      | Claude personal skills: `C:\Users\shaked arazi\.claude\skills\<skill>\SKILL.md`                                                                                                                      |
| Global | Built-in Copilot prompt, tool catalog, safety rules, model behavior                            | Built-in Claude prompt, tool catalog, safety rules, model behavior                                                                                                                                   |
| Global | Optional organization instructions and custom agents if enabled                                | Claude history and state: `C:\Users\shaked arazi\.claude\history.jsonl`, `C:\Users\shaked arazi\.claude\projects\`, `C:\Users\shaked arazi\.claude\cache\`, `C:\Users\shaked arazi\.claude\backups\` |
| Repo   | `.github/copilot-instructions.md`                                                              | `CLAUDE.md` in repo root                                                                                                                                                                             |
| Repo   | `AGENTS.md` in repo root or subfolders                                                         | `.claude/CLAUDE.md`                                                                                                                                                                                  |
| Repo   | `.github/instructions/*.instructions.md`                                                       | `.claude/rules/*.md`                                                                                                                                                                                 |
| Repo   | `.github/prompts/*.prompt.md`                                                                  | `.claude/agents/*.md`                                                                                                                                                                                |
| Repo   | `.github/agents/*.agent.md`                                                                    | `.claude/skills/<skill>/SKILL.md`                                                                                                                                                                    |
| Repo   | `.github/skills/<skill>/SKILL.md` or `.agents/skills/<skill>/SKILL.md`                         | `.claude/settings.json` and `.claude/settings.local.json` for repo-level Claude hooks/config                                                                                                         |
| Repo   | `.github/hooks/*.json`                                                                         | Repo files Claude reads, searches, or attaches                                                                                                                                                       |
| Repo   | Repo files Copilot reads, searches, or attaches                                                | Repo files Claude reads, searches, or attaches                                                                                                                                                       |

## 2. What Actually Injects Context

| Type                                        | Copilot                                                                                                                                                                                 | Claude Code                                                                                                                 |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Injects prompt text automatically           | built-in system prompt, session history, active instructions, active agents, loaded skills, prompt file body when invoked, hook additionalContext                                       | built-in system prompt, session history, `CLAUDE.md`, `.claude/rules`, active agents, loaded skills, hook additionalContext |
| Can inject context conditionally            | `.instructions.md` by `applyTo` or semantic match, skills by description match, agents by selection or subagent delegation, prompt files by slash command, hooks by `additionalContext` | `CLAUDE.md`, `.claude/rules`, Claude agents, Claude skills, hooks via Claude settings, project files read into context      |
| Changes behavior without being prompt text  | VS Code `settings.json`, extension settings, tool approvals, organization policy switches                                                                                               | `.claude/settings.json`, plugin enablement, permissions, model selection                                                    |
| Not rules but still grows effective context | session history, attached files, tool outputs, code search results, errors, terminal output                                                                                             | session history, attached files, tool outputs, project state, history and prior Claude continuity                           |

## 3. Current Machine State

This is the effective state after the Copilot separation changes.

| Area                                      | Current State                                                                                                |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Copilot loading `CLAUDE.md`               | Disabled by `chat.useClaudeMdFile: false`                                                                    |
| Copilot loading `.claude/rules`           | Disabled in `chat.instructionsFilesLocations`                                                                |
| Copilot loading Postman temp instructions | Disabled in `chat.instructionsFilesLocations`                                                                |
| Copilot personal instructions             | Enabled from `C:\Users\shaked arazi\.copilot\instructions\copilot-chat-core.instructions.md`                 |
| Copilot workspace instructions            | Still available if a repo contains `.github/instructions`, `.github/copilot-instructions.md`, or `AGENTS.md` |
| Claude global instructions                | Still active from `C:\Users\shaked arazi\.claude\CLAUDE.md`                                                  |
| Claude global settings/hooks              | Still active from `C:\Users\shaked arazi\.claude\settings.json`                                              |
| Claude plugin layer                       | Active, including `superpowers@claude-plugins-official`                                                      |

## 4. ASCII Layer Diagram

```text
                             +--------------------------------+
                             |      BUILT-IN PLATFORM         |
                             | Copilot/Claude core prompt     |
                             | built-in tools, policies       |
                             +---------------+----------------+
                                             |
                      +----------------------+----------------------+
                      |                                             |
          +-----------v-----------+                     +-----------v-----------+
          |   GLOBAL COPILOT      |                     |   GLOBAL CLAUDE       |
          | settings.json         |                     | ~/.claude/CLAUDE.md   |
          | ~/.copilot/instructions|                    | ~/.claude/settings.json|
          | User/prompts/*.prompt |                     | ~/.claude/rules/      |
          | User/prompts/*.agent  |                     | ~/.claude/plugins/    |
          | ~/.copilot/skills/    |                     | ~/.claude/skills/     |
          +-----------+-----------+                     +-----------+-----------+
                      |                                             |
                      +----------------------+----------------------+
                                             |
                                  +----------v----------+
                                  |   REPO / WORKSPACE  |
                                  | .github/...         |
                                  | AGENTS.md           |
                                  | CLAUDE.md           |
                                  | .claude/...         |
                                  +----------+----------+
                                             |
                                  +----------v----------+
                                  |   SESSION CONTEXT   |
                                  | chat history        |
                                  | tool outputs        |
                                  | attached files      |
                                  | active selection    |
                                  +---------------------+
```

## 5. Primitives and Their Jobs

| Primitive                 | Main Job                                                       | Usually Auto-Loaded                                            | Typical Location                                                                                          |
| ------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `copilot-instructions.md` | always-on repo rules for Copilot                               | yes                                                            | `.github/copilot-instructions.md`                                                                         |
| `AGENTS.md`               | always-on repo rules for agents                                | yes                                                            | repo root or subfolders                                                                                   |
| `*.instructions.md`       | targeted instructions by file pattern or semantic match        | sometimes                                                      | `.github/instructions/` or personal instruction folder                                                    |
| `CLAUDE.md`               | always-on Claude instructions                                  | yes for Claude, optionally yes for Copilot unless disabled     | repo root, `.claude/CLAUDE.md`, or `~/.claude/CLAUDE.md`                                                  |
| `.claude/rules/*.md`      | smaller Claude rules                                           | yes when enabled                                               | repo `.claude/rules/` or `~/.claude/rules/`                                                               |
| `*.prompt.md`             | reusable one-shot tasks                                        | no, only when invoked                                          | `.github/prompts/` or user prompts folder                                                                 |
| `*.agent.md`              | a custom agent persona with selected tools and instructions    | no unless selected or delegated to                             | `.github/agents/` or user prompts folder                                                                  |
| `.claude/agents/*.md`     | Claude-format custom agent definitions                         | no unless selected or delegated to                             | `.claude/agents/`                                                                                         |
| `SKILL.md`                | reusable workflow with assets, scripts, examples, references   | metadata is discoverable, body loads only when relevant        | `.github/skills/`, `.agents/skills/`, `.claude/skills/`, `~/.copilot/skills/`                             |
| hooks JSON                | deterministic automation and enforcement                       | yes if enabled for that event                                  | `.github/hooks/*.json`, `.claude/settings.json`, `.claude/settings.local.json`, `~/.claude/settings.json` |
| MCP server config         | external tools and data systems                                | not prompt text by itself, but expands available tools/context | extension settings or custom agent config                                                                 |
| plugins                   | bundled skills/capabilities from an extension or Claude plugin | depends on installation and enablement                         | extension package or `~/.claude/plugins/`                                                                 |

## 6. Prompts vs Skills vs Agents vs Hooks

| Item             | Use It When                                                       | Auto Context Cost                       | Why It Exists                  |
| ---------------- | ----------------------------------------------------------------- | --------------------------------------- | ------------------------------ |
| Prompt           | one focused task with arguments                                   | low until invoked                       | reusable slash command         |
| Skill            | repeatable workflow with docs, scripts, and examples              | low at discovery, higher only when used | portable capability            |
| Agent            | you want a role with specific tools and behavior                  | medium when selected                    | mode/persona and tool boundary |
| Hook             | behavior must be enforced, blocked, or injected deterministically | runtime, not mainly prompt text         | policy and automation          |
| Instruction file | a rule should influence many tasks automatically                  | can be high if too broad                | behavior guidance              |

## 7. Commands You Will Actually Use

### Chat slash commands

| Command               | Purpose                                |
| --------------------- | -------------------------------------- |
| `/instructions`       | open instruction/rule configuration UI |
| `/create-instruction` | generate a new instruction file        |
| `/agents`             | open custom agents UI                  |
| `/create-agent`       | generate a custom agent                |
| `/skills`             | open skills UI                         |
| `/create-skill`       | generate a new skill                   |
| `/hooks`              | open hooks UI                          |
| `/create-hook`        | generate a hook config                 |
| `/init`               | generate workspace instructions        |

### VS Code commands from the Command Palette

| Command Palette Item             | Purpose                                   |
| -------------------------------- | ----------------------------------------- |
| `Chat: Open Chat Customizations` | open the customization overview/editor    |
| `Chat: Configure Instructions`   | inspect and edit loaded instruction files |
| `Chat: New Instructions File`    | create a new `.instructions.md` file      |
| `Chat: New Custom Agent`         | create a new custom agent                 |
| `Chat: Configure Hooks`          | inspect and edit hook configuration       |
| `Chat: Run Prompt...`            | run a saved prompt file                   |

## 8. Hook Events and What They Can Do

| Event              | What It Fires On                 | Why You Would Use It                                        |
| ------------------ | -------------------------------- | ----------------------------------------------------------- |
| `SessionStart`     | first prompt in a new session    | inject project state, branch, environment                   |
| `UserPromptSubmit` | every user prompt                | audit prompts or add context                                |
| `PreToolUse`       | before any tool runs             | block dangerous tools, require approval, rewrite tool input |
| `PostToolUse`      | after a tool succeeds            | run formatters, log activity, add follow-up context         |
| `PreCompact`       | before context compaction        | save summaries or important state                           |
| `SubagentStart`    | before a subagent runs           | tag or constrain delegated work                             |
| `SubagentStop`     | after a subagent finishes        | validate or continue subagent work                          |
| `Stop`             | when the session wants to finish | force final checks before the agent stops                   |

## 9. Fine-Tuning Rules of Thumb

| If You Want To Change...                   | Edit This First                                                                                                               |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| how Copilot behaves in every repo          | `C:\Users\shaked arazi\.copilot\instructions\*.instructions.md`                                                               |
| how Claude behaves in every repo           | `C:\Users\shaked arazi\.claude\CLAUDE.md`                                                                                     |
| tool permissions or Claude automation      | `C:\Users\shaked arazi\.claude\settings.json`                                                                                 |
| a repo-wide policy for everyone            | workspace files under `.github/` or repo `AGENTS.md`                                                                          |
| a reusable one-command workflow            | a prompt or skill                                                                                                             |
| a specialized planning/review persona      | a custom agent                                                                                                                |
| hard enforcement before or after tools run | hooks                                                                                                                         |
| behavior only for a language or folder     | targeted `.instructions.md` with narrow `applyTo`                                                                             |
| lower context usage                        | trim always-on instructions, avoid `applyTo: "**"` unless the file is very small, prefer prompts/skills for optional behavior |

## 10. Practical Control Strategy

Use this split if you want maximum control with minimum confusion.

| Layer                  | What Should Live There                                                     |
| ---------------------- | -------------------------------------------------------------------------- |
| Copilot personal layer | short coding preferences, response style, bug explanation style            |
| Claude personal layer  | Claude workflows, plugin-oriented behavior, heavier personal process rules |
| Repo shared layer      | project conventions, build/test commands, architecture notes               |
| Skill layer            | reusable multi-step capabilities                                           |
| Agent layer            | role-based modes with tool restrictions                                    |
| Hook layer             | deterministic security, automation, approvals, guardrails                  |

## 11. Debug Checklist

If behavior feels wrong, check in this order.

| Step | Check                                                                                                                                                                         |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | start a fresh chat to remove old session context                                                                                                                              |
| 2    | open `Chat: Configure Instructions` and inspect loaded instructions                                                                                                           |
| 3    | open chat diagnostics to see sources of instructions, agents, prompts, and skills                                                                                             |
| 4    | inspect `settings.json` for `chat.useClaudeMdFile`, `chat.instructionsFilesLocations`, `chat.hookFilesLocations`, `chat.agentSkillsLocations`, and `chat.agentFilesLocations` |
| 5    | review broad files using `applyTo: "**"`                                                                                                                                      |
| 6    | inspect hooks if behavior is being enforced rather than merely suggested                                                                                                      |
| 7    | inspect extension-contributed plugins, skills, or agents                                                                                                                      |

## 12. One-Line Mental Model

Instructions tell the model what to prefer, agents tell it what role and tools to use, skills teach reusable workflows, prompts trigger reusable tasks, hooks enforce runtime policy, settings decide what sources are even allowed to participate, and the session adds the live context from the current conversation.
