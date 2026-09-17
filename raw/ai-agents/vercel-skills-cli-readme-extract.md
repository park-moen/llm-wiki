# skills CLI README — find-skills 관련 원문 발췌

> Source: https://github.com/vercel-labs/skills
> Collected: 2026-09-16
> Published: Unknown

아래 내용은 `find-skills`의 저장소 위치, 검색 명령, 설치 범위와 Codex 경로를 확인하는 데 필요한 공식 README 원문 발췌다.

# skills

The CLI for the open agent skills ecosystem.

Supports **OpenCode**, **Claude Code**, **Codex**, **Cursor**, and [75 more](#supported-agents).

## Install a Skill

```bash
npx skills add vercel-labs/agent-skills
```

### Options

| Option                    | Description                                                                                                                                        |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-g, --global`            | Install to user directory instead of project                                                                                                       |
| `-a, --agent <agents...>` | Target specific agents (e.g., `claude-code`, `codex`). See Supported Agents                                                                         |
| `-s, --skill <skills...>` | Install specific skills by name (use `'*'` for all skills)                                                                                         |
| `-l, --list`              | List available skills without installing                                                                                                           |
| `--copy`                  | Copy files instead of symlinking to agent directories                                                                                              |
| `-y, --yes`               | Skip all confirmation prompts                                                                                                                      |
| `--all`                   | Install all skills to all agents without prompts                                                                                                   |

### Installation Scope

| Scope       | Flag      | Location            | Use Case                                      |
| ----------- | --------- | ------------------- | --------------------------------------------- |
| **Project** | (default) | `./<agent>/skills/` | Committed with your project, shared with team |
| **Global**  | `-g`      | `~/<agent>/skills/` | Available across all projects                 |

### Installation Methods

When installing interactively, you can choose:

| Method                    | Description                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| **Symlink** (Recommended) | Creates symlinks from each agent to a canonical copy. Single source of truth, easy updates. |
| **Copy**                  | Creates independent copies for each agent. Use when symlinks aren't supported.              |

## Other Commands

| Command                      | Description                                   |
| ---------------------------- | --------------------------------------------- |
| `npx skills use <source>`    | Use one skill without installing              |
| `npx skills list`            | List installed skills (alias: `ls`)           |
| `npx skills find [query]`    | Search for skills interactively or by keyword |
| `npx skills remove [skills]` | Remove installed skills from agents           |
| `npx skills update [skills]` | Update installed skills to latest versions    |
| `npx skills init [name]`     | Create a new SKILL.md template                |

### `skills find`

Search for skills interactively or by keyword.

```bash
# Interactive search (fzf-style)
npx skills find

# Search by keyword
npx skills find typescript

# Search across every repository owned by an organization or user
npx skills find react --owner vercel
```

## What are Agent Skills?

Agent skills are reusable instruction sets that extend your coding agent's capabilities. They're defined in `SKILL.md` files with YAML frontmatter containing a `name` and `description`.

Skills let agents perform specialized tasks like:

- Generating release notes from git history
- Creating PRs following your team's conventions
- Integrating with external tools (Linear, Notion, etc.)

Discover skills at **[skills.sh](https://skills.sh)**

## Supported Agents

Skills can be installed to any of these agents:

| Agent | `--agent` | Project Path | Global Path |
|-------|-----------|--------------|-------------|
| Claude Code | `claude-code` | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `codex` | `.agents/skills/` | `~/.codex/skills/` |
| Cursor | `cursor` | `.agents/skills/` | `~/.cursor/skills/` |
| Gemini CLI | `gemini-cli` | `.agents/skills/` | `~/.gemini/skills/` |
| GitHub Copilot | `github-copilot` | `.agents/skills/` | `~/.copilot/skills/` |

The CLI automatically detects which coding agents you have installed. If none are detected, you'll be prompted to select which agents to install to.

### Skill Discovery

The CLI searches for skills in these locations within a repository. Each skill container directory is walked up to three levels deep, covering flat layouts (`skills/<name>/SKILL.md`) and catalog layouts with one or two category levels (`skills/<category>/<name>/SKILL.md` or `skills/<category>/<category>/<name>/SKILL.md`). A `SKILL.md` discovered at a shallower level shadows anything nested below it. Use `--full-depth` to also discover `SKILL.md` files outside these container directories (e.g. under `examples/` or `tests/`).
