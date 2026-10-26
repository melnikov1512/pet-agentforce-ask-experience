<div style="text-align: center">

# Salesforce DX Project Template

![Salesforce API](https://img.shields.io/badge/Salesforce_API-v66.0-00A1E0)
![Node.js](https://img.shields.io/badge/Node.js-18.11%2B-339933)

_Production-ready Salesforce DX starter with quality tooling, CI/CD, and Codex configuration built in_

[Getting Started](#getting-started) • [Commands](#commands) • [CI/CD](#cicd-pipelines) • [Codex Configuration](#codex-configuration) • [Commit Policy](#commit-policy) • [Troubleshooting](#troubleshooting)

</div>

Clone this template, add your metadata to `force-app/`, configure the `SF_AUTH_URL` secret, and you're ready to ship. Everything else — linting, formatting, testing, security gates, deployments, and AI-assisted code review — is already wired up.

## What's included

- **Linting** — ESLint flat config for LWC and Aura JS (`@salesforce/eslint-config-lwc`)
- **Formatting** — Prettier with Apex and XML plugins; 4-space indent for all code files; enforced in CI and on pre-commit
- **LWC unit tests** — `@salesforce/sfdx-lwc-jest` with watch, debug, and coverage modes
- **Apex SAST** — `sf code-analyzer` (SARIF output) on every PR; blocks on `error`-level findings
- **Pre-commit hooks** — Husky + lint-staged: format → lint → related LWC tests, automatically
- **CI pipelines** — PR checks, metadata validate-only gate, security scanning, AI PR summaries, and guarded manual deploys
- **Codex** — 18 specialist agents, shared project instructions, 20 project workflows, and 49 curated Salesforce skills

## Prerequisites

| Requirement                                                                   | Version / Notes                                    |
| ----------------------------------------------------------------------------- | -------------------------------------------------- |
| [Node.js](https://nodejs.org)                                                 | 18.11+                                             |
| [Salesforce CLI (`sf`)](https://developer.salesforce.com/tools/salesforcecli) | Latest; required for Apex tests and org operations |
| Authenticated Salesforce org                                                  | Required for `test:apex` and deploy commands       |
| `SF_AUTH_URL` repository secret                                               | Required for CI metadata validation                |

## Getting started

```bash
# 1. Clone the repo and install dependencies (uses package-lock.json as-is)
git clone https://github.com/your-org/your-repo.git
cd sfdx-project-template
npm ci

# 2. Authenticate your Dev Hub
sf org login web --alias devhub --set-default-dev-hub

# 3. Create a scratch org
sf org create scratch \
  --definition-file config/project-scratch-def.json \
  --alias dev \
  --duration-days 30 \
  --set-default

# 4. Deploy metadata to the scratch org
npm run deploy -- --target-org dev

# 5. Validate everything looks good
npm run validate
```

> [!NOTE]
> `test:apex` is skipped gracefully when no `@isTest` classes exist, `sf` is not installed, or no default org is configured — no false failures on a fresh clone.

## Project structure

```
sfdx-project-template/
├── force-app/
│   └── main/default/          # Add your Salesforce metadata here
├── config/
│   └── project-scratch-def.json
├── data/
│   └── knowledge/all-articles.json # Reusable creation data for 100 Knowledge articles
├── docs/
│   └── plans/                 # Implementation plans
├── scripts/
│   └── deploy.js              # Local deploy helper (mirrors deploy.yml)
├── .agents/
│   ├── instructions/          # Shared rules and seven task-specific rule files
│   └── skills/                # 20 project workflows and 49 Salesforce skills
├── .codex/
│   ├── agents/                # 18 specialist agents
│   └── config.toml            # Project MCP configuration
├── .github/
│   └── workflows/             # CI/CD pipeline definitions
├── sfdx-project.json          # API v66.0, package dir: force-app
├── .prettierrc                # Prettier config: 4-space indent, Apex + XML plugins
├── eslint.config.js
├── jest.config.js
└── package.json
```

## Commands

Knowledge metadata is stored under `force-app/main/default/` in `objects/Knowledge__kav/`, `layouts/`, `profiles/`, and `settings/`. The reusable content for all 100 articles is saved in `data/knowledge/all-articles.json`, without source-org record IDs. See [the Knowledge data guide](data/knowledge/index.md) for destination-org prerequisites and import considerations. The JSON must be adapted to the chosen import tool; metadata deployment does not create or publish article records.

### Daily development

| Command                     | Description                               |
| --------------------------- | ----------------------------------------- |
| `npm run lint`              | ESLint for LWC/Aura JS                    |
| `npm run prettier`          | Format all supported files in-place       |
| `npm run prettier:verify`   | Check formatting without writing          |
| `npm run test:lwc`          | Run LWC Jest tests                        |
| `npm run test:lwc:watch`    | Jest in watch mode                        |
| `npm run test:lwc:coverage` | Jest with coverage report                 |
| `npm run test:apex`         | Run Apex tests on connected org           |
| `npm run test`              | LWC CI + Apex tests                       |
| `npm run validate`          | Lint + format check + LWC CI + Apex tests |
| `npm run lint:apex`         | Salesforce SAST via `sf code-analyzer`    |
| `npm run deploy`            | Deploy to an org (see Local deploy below) |

### Scratch org workflow

```bash
# Create a scratch org and set it as default
sf org create scratch \
  --definition-file config/project-scratch-def.json \
  --alias dev \
  --duration-days 30 \
  --set-default

# Deploy source using the local deploy helper
npm run deploy -- --target-org dev

# Open in browser
sf org open
```

### Local deploy

`npm run deploy` is a thin wrapper around `sf project deploy start/validate` that saves results to `.artifacts/deploy/`.

```bash
# Deploy to an org (authenticated alias required)
npm run deploy -- --target-org <alias>

# Validate only — no actual deploy
npm run deploy -- --target-org <alias> --validate-only

# Deploy and run all local Apex tests
npm run deploy -- --target-org <alias> --tests

# Deploy a custom metadata directory
npm run deploy -- --target-org <alias> --source-dir force-app/main/default/classes

# Increase wait timeout (minutes, default: 30)
npm run deploy -- --target-org <alias> --wait 60
```

> [!NOTE]
> Results and logs are written to `.artifacts/deploy/results/`. For CI/CD-governed deploys to QA and higher envs, use the **Actions → Deploy** workflow instead — see `RUNBOOK.md` §17.

## CI/CD pipelines

| Workflow                      | Trigger                      | What it does                                                     |
| ----------------------------- | ---------------------------- | ---------------------------------------------------------------- |
| `pr-check.yml`                | PR → `main`                  | Lint, format check, LWC tests, Apex SAST, metadata validate-only |
| `security-gates.yml`          | PR + push `main` + weekly    | Secret scanning, dependency audit (blocks on `high`/`critical`)  |
| `ai-pr-summary.yml`           | PR opened/updated            | AI-generated PR summary via GitHub Models (`gpt-4o-mini`)        |
| `ai-test-recommendations.yml` | PR opened/updated            | AI-suggested test improvements                                   |
| `codeql.yml`                  | PR + push `main`             | CodeQL static analysis                                           |
| `deploy.yml`                  | Manual (`workflow_dispatch`) | Deploy any branch to QA and higher envs with a validate gate     |

> [!IMPORTANT]
> Metadata validation in CI requires the `SF_AUTH_URL` repository secret. PRs that touch `force-app/` (or the path set in `SFDX_METADATA_DIR`) will fail explicitly if the secret is absent.

> [!TIP]
> Set the optional repository variable `SFDX_METADATA_DIR` to change the metadata root without modifying any code. Defaults to `force-app`.

## Deployment

Deployments to QA and higher envs are triggered manually via **Actions → Deploy → Run workflow**.

- A validate-only gate always runs before the actual deploy.
- Higher environments require Required Reviewers and enforce a `main`-only branch policy.
- For rollback steps, see `RUNBOOK.md` §17.

## Codex configuration

- `AGENTS.md` is the repository instruction entry point. It loads shared rules from `.agents/instructions/project-rules.md` and relevant `.agents/instructions/` files.
- `.codex/agents/` contains 18 specialist agents in TOML format, including Apex, LWC, data architecture, implementation, review, and orchestration roles.
- `.agents/skills/` contains 20 project workflows and 49 curated upstream Salesforce skills, including the converted `create-exception-handler` prompt. Invoke a workflow with `$skill-name`.
- Agent definitions inherit the parent model, runtime permissions, and connected tools.
- Start a new Codex session after migration if the host does not refresh discovery. Custom-agent support depends on the host integration.
- The Salesforce skills library (`forcedotcom/sf-skills`) is installed locally in `.agents/skills/`. Its upstream revision and inventory are recorded in `.agents/skills/salesforce-library.json`; the upstream license is in `.agents/skills/SALESFORCE-LICENSE.txt`. Updates are explicit, reviewable changes. Skills that operate on an org require local Salesforce CLI authentication and any prerequisites specified by the skill. The retired Salesforce DX MCP integration is no longer required.

See [Codex customization](docs/codex-customization-guide.md) for file formats and recommendations, and [MCP setup](docs/mcp-setup.md) for the repository's integration guidance. Skill discovery does not install MCP servers or external CLIs.

## Commit policy

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

- **Description**: present tense, imperative mood, max 72 characters (`add` not `added`)
- **Body**: explain _why_, not _what_ — wrap at 72 characters
- **Footer**: reference issues (`Closes #123`) or breaking changes

### Types

| Type       | When to use                                      |
| ---------- | ------------------------------------------------ |
| `feat`     | New feature or capability                        |
| `fix`      | Bug fix                                          |
| `docs`     | Documentation only (README, AGENTS.md, comments) |
| `style`    | Formatting, whitespace — no logic change         |
| `refactor` | Code restructure — no new feature, no bug fix    |
| `perf`     | Performance improvement                          |
| `test`     | Add or update tests / test scripts               |
| `build`    | Dependencies, package.json, npm scripts          |
| `ci`       | CI/CD workflow changes                           |
| `chore`    | Config, tooling, minor housekeeping              |
| `revert`   | Revert a previous commit                         |

### Scopes

| Scope      | Maps to                                               |
| ---------- | ----------------------------------------------------- |
| `apex`     | `force-app/**/classes/**`, `force-app/**/triggers/**` |
| `lwc`      | `force-app/**/lwc/**`                                 |
| `aura`     | `force-app/**/aura/**`                                |
| `metadata` | Other Salesforce metadata under `force-app/`          |
| `ci`       | `.github/workflows/**`                                |
| `agents`   | `.codex/agents/**`, `.agents/skills/**`, `AGENTS.md`, `.agents/instructions/**` |
| `config`   | `sfdx-project.json`, `config/**`, `eslint.config.js`  |
| `scripts`  | `scripts/**`                                          |
| `docs`     | `docs/**`, `RUNBOOK.md`, `README.md`                  |
| `deps`     | `package.json`, `package-lock.json`                   |

Omit scope when the change spans multiple unrelated areas.

### Breaking changes

```
feat(apex)!: rename AccountSelector.getRecentAccounts to findRecentAccounts

BREAKING CHANGE: callers must update method references before deploying
```

### Examples

```
feat(apex): add AccountSelector with getRecentAccounts query
test(apex): add AccountSelectorTest covering limit and ordering
fix(ci): use RunLocalTests for validate-only deploys
refactor(lwc): extract date formatting into shared utility
docs: add commit policy section to README
build(deps): bump @salesforce/eslint-config-lwc to 4.x
ci: add Apex test coverage gate to pr-check workflow
```

### Rules

1. One logical change per commit — if the diff spans unrelated concerns, split it.
2. Never commit secrets — skip `.env`, auth files, or any credential material.
3. Do not use `--no-verify` unless explicitly required and documented.
4. Do not force-push to `main`.

---

## Troubleshooting

For common issues — bootstrap failures, lint false negatives, Apex test skips, CI metadata validation failures, and security exception lifecycle — see **`RUNBOOK.md`**.
