# MCP Setup

Project-local MCP configuration lives in `.codex/config.toml`. Skills and agent definitions do not install MCP servers or grant credentials. Check the host's connected tools before relying on an integration.

## Salesforce skills

Salesforce workflows are installed project-locally in `.agents/skills/` from `forcedotcom/sf-skills`. This replaces the Salesforce DX local MCP integration, whose announced end of life is November 2, 2026. Do not add the retired MCP package when setting up this project.

The installed snapshot contains 49 curated Salesforce skills. Its exact upstream revision and inventory are recorded in `.agents/skills/salesforce-library.json`; its license is preserved in `.agents/skills/SALESFORCE-LICENSE.txt`. Refresh skill discovery if the host does not detect the new skills automatically. Org operations still require Salesforce CLI authentication and each skill's prerequisites. Review upstream updates explicitly and preserve existing project workflows.

---

### Curated Salesforce coverage

- Core development: Apex and tests, SOQL, data operations, metadata deployment/retrieval, custom objects, permissions, sharing, validation, value sets, integrations, org management, and Code Analyzer.
- Agentforce: generation, testing, observation, and architecture analysis.
- Local/CLI Digital Experience development: branding files, LWC development and migration, SLDS, accessibility, security, RTL, runtime observation, and LDS/GraphQL.

Skills whose primary workflow requires unavailable Salesforce MCP services have been removed: Flow generation, custom-field generation, portal provisioning, LWR site generation/configuration, CMS brand retrieval/application, CMS content authoring/rendering/type generation, and CMS search. This selection does not require installing Metadata Experts, Metadata Grounding, Headless 360, CMS read/write services, or a skill bridge. No Salesforce MCP connection was added during the connection investigation.

A retained skill may mention an optional MCP enhancement or an unavailable related skill. Use its documented local/CLI fallback where available; do not treat these references as instructions to install a server or restore excluded skills. For example, value-set grounding is optional, and the Apex skill allows reporting the unavailable analyzer integration; CLI code analysis remains available through the Code Analyzer skills.

The selection also excludes industry-specific products and DevOps Center. Review upstream updates explicitly and preserve the selection policy instead of reinstalling the full library.

---

## Codex connections

Context7 is configured in the project's Codex configuration. Keep API credentials in environment variables or the host credential store, rather than repository files. Additional services should be connected only when a task requires them.

Use the Codex host's MCP settings to manage connections. The CLI also provides `codex mcp list`, `codex mcp add --help`, and `codex mcp remove <name>` for user-level connection management. Host support for project configuration and live discovery varies; refresh discovery or start a new session after configuration changes.

The project uses local shell tools for files, Salesforce CLI for org operations, and the installed skills for workflows. No Salesforce MCP service is required by the curated setup.
