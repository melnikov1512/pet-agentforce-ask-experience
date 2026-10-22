# Codex customization

## Repository layout

| Purpose | Location | Loading behavior |
| --- | --- | --- |
| Repository entry point | `AGENTS.md` | Read at task startup |
| Shared project rules | `.agents/instructions/project-rules.md` | Explicitly read by the bootstrap |
| Seven task-specific rule files | `.agents/instructions/*.instructions.md` | Select using the table in AGENTS.md |
| 18 specialist agents | `.codex/agents/*.toml` | Delegated roles when supported by the host |
| 20 project workflows | `.agents/skills/<name>/SKILL.md` | Explicit invocation or description matching |
| 49 Salesforce skills | `.agents/skills/<name>/SKILL.md` | Inventory in salesforce-library.json |
| Project MCP settings | `.codex/config.toml` | Loaded by supporting hosts |

Instruction files are ordinary Markdown. Their location does not activate automatic file-pattern routing; keep the explicit bootstrap table current.

## Agents

Each agent TOML defines `name`, `description`, and `developer_instructions`. Agents inherit the parent model, permissions, and connected tools. They read AGENTS.md and the shared rules before assigned work. A textual read-only instruction describes intended behavior; it does not create a filesystem sandbox.

Ask the main chat to use a specialist for a concrete task. Role files do not automatically replace the main chat agent. The orchestrator coordinates only when explicitly requested and respects host concurrency limits. Plans and supporting artifacts live in `docs/plans/<plan-name>/`.

## Skills

Each skill directory contains a SKILL.md with YAML `name` and `description`, plus workflow instructions. Supporting references, scripts, and assets stay beside it. Invoke a workflow with `$skill-name` or let its description match the task.

Salesforce skills are vendored at the revision recorded in `.agents/skills/salesforce-library.json`, with their license preserved. Retain workflows that operate through local files and the Salesforce CLI. Do not reinstall the full library or add MCP-dependent workflows without an explicit project need.

Installing skills does not install CLIs, authenticate GitHub or Salesforce, or connect external services. Check prerequisites and use documented fallbacks. Runtime permissions come from the host.

## Maintenance and verification

Update shared rules once in `.agents/instructions/` and keep every agent and skill reference aligned. Update AGENTS.md and README.md when locations, counts, or integration boundaries change.

Validate agent TOML and skill frontmatter after edits, check supporting paths, and search for stale references after moves. Refresh discovery or start a new session if the host does not detect changes. Configuration checks do not establish that external tools are connected.
