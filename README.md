# security-plugin

A Claude Code marketplace with two plugins that audit a project for security vulnerabilities against
the **OWASP Top 10**. Install one or both — each plugin is a single skill.

| Plugin / skill | Checklist | Use it for |
|-------|-----------|------------|
| `owasp-agentic-audit` | [OWASP Agentic Skills Top 10](https://owasp.org/www-project-agentic-skills-top-10/) (AST01–AST10) | AI agent skills, Claude Code plugins, MCP servers, prompts |
| `owasp-app-audit` | Classic [OWASP Top 10](https://owasp.org/Top10/) (A01–A10) | Web apps, APIs, backends |

For a hybrid project (e.g. an MCP server with a real backend), run both.

If the working directory has no source code, either skill asks for a GitHub repository, clones it
read-only, and audits that.

## Install

Add the marketplace once:

```text
/plugin marketplace add dmappel/security-plugin
```

Then install the checks you want (or browse them in `/plugin`):

```text
/plugin install owasp-agentic-audit@security-plugin
/plugin install owasp-app-audit@security-plugin
```

For a local checkout, pass the folder path instead: `/plugin marketplace add /path/to/security-plugin`.

Then start a new session (or `/reload-plugins`).

## Use

Call a skill directly:

- `/owasp-agentic-audit:owasp-agentic-audit`
- `/owasp-app-audit:owasp-app-audit`

Or just ask, and Claude picks the matching installed skill:

- "audit this skill for security issues" → agentic
- "check this API for OWASP vulnerabilities" → app
- "scan https://github.com/owner/repo for security issues"

## Output

Reports are written to `security-reports/<YYYY-MM-DD>/agentic-audit-<name>-<HH-MM>.md` or
`app-audit-<name>-<HH-MM>.md`, containing:

- an executive summary and a findings table,
- per-finding detail: **severity** (Critical/High/Medium/Low), location (`file:line`), evidence,
  impact, and a concrete **suggested fix**,
- a coverage list confirming every risk was evaluated.

Because reports list exploitable weaknesses, the skills drop a self-contained `.gitignore` inside
`security-reports/` so reports stay out of version control by default (delete it if your team wants
them tracked). They never write report artifacts into a cloned third-party repo.

## Layout

```
.claude-plugin/marketplace.json                      # marketplace "security-plugin"
plugins/owasp-agentic-audit/
  .claude-plugin/plugin.json                         # plugin "owasp-agentic-audit"
  skills/owasp-agentic-audit/SKILL.md
  skills/owasp-agentic-audit/references/agentic-skills-top10.md
plugins/owasp-app-audit/
  .claude-plugin/plugin.json                         # plugin "owasp-app-audit"
  skills/owasp-app-audit/SKILL.md
  skills/owasp-app-audit/references/owasp-web-top10.md
```
