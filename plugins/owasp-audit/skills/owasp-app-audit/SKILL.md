---
name: owasp-app-audit
description: >-
  Audit an application codebase (web app, API, backend, service) for security vulnerabilities against
  the classic OWASP Top 10 (A01–A10): broken access control, injection, cryptographic failures,
  insecure design, misconfiguration, vulnerable dependencies, auth failures, SSRF, and more. If the
  working directory has no source code, it asks for a GitHub repo, clones it, then audits. Writes a
  dated report (findings, severity, suggested fixes) to security-reports/<date>/. Use this whenever
  the user asks to check, audit, review, or scan an application or its code for security issues,
  vulnerabilities, OWASP risks, or insecure code — even if they don't name OWASP explicitly.
---

# OWASP Application Security Audit

Run a structured security audit of an application codebase against the **classic OWASP Top 10
(A01–A10)** and produce a dated, actionable report.

For AI agent skills, plugins, and MCP servers, the sibling skill `owasp-agentic-audit` applies the
OWASP Agentic Skills Top 10 instead.

Work through the four phases below in order. Create a TodoWrite item per phase so progress is visible.

## Phase 1 — Establish the target

The audit runs against a **target project directory**. Determine it before anything else.

1. Inspect the current working directory. Treat it as **empty / no project** if it contains no
   source code — i.e. only hidden config (`.git`, `.claude`), READMEs, or is literally empty. A good
   check: `git ls-files 2>/dev/null | head` plus a listing of non-hidden files. Reports the skill
   itself produced (`security-reports/`) do not count as source.
2. **If a real project is present**, that directory is the target. Proceed to Phase 2.
3. **If empty**, ask the user for a GitHub repository to audit (URL or `owner/repo`). Then clone it:
   ```bash
   git clone --depth 1 <repo-url> <target-dir>
   ```
   Clone into the working directory if empty, otherwise into a clearly named subdirectory. Confirm
   the clone succeeded and use the cloned directory as the target. Never push, modify, or commit to
   the cloned repo — this is read-only analysis.

Tell the user which directory you're auditing before continuing.

## Phase 2 — Scope check

Read `references/owasp-web-top10.md` now — it contains, per risk, what it means and concrete things
to look for.

Do a quick scan of the tree (don't search exhaustively) for application surface:
- Dependency manifests (`package.json`, `requirements.txt`, `go.mod`, `pom.xml`, `Gemfile`, `composer.json`)
- Web/app frameworks (Express, Django, Flask, Rails, Spring, Next.js, etc.)
- Route/controller/handler code, database access, auth code, templates

- If there is no application code at all (e.g. only `SKILL.md` files and prompts), tell the user this
  checklist mostly doesn't apply, and suggest `owasp-agentic-audit` instead. Continue only if they
  want to.
- If you also see agentic surface (`SKILL.md`, `.claude-plugin/`, MCP server definitions, agent
  prompts, `allowed-tools`), mention it and suggest also running `owasp-agentic-audit` for that part.
  Don't run it yourself — this skill covers A01–A10 only.

## Phase 3 — Audit

Go through every risk A01–A10. For each risk:
- Inspect the files and patterns the reference file calls out (use Grep/Glob/Read; targeted searches
  beat reading everything).
- Record each concrete finding with its evidence: the `file:line`, the offending snippet or pattern,
  and *why* it's a problem.
- A risk with no issues found is still worth one line in the report ("A10 — no findings") so the
  reader knows it was checked, not skipped.

**Be honest and evidence-based.** Only report a finding you can point to in the code. Don't pad the
report with hypotheticals. If you couldn't fully evaluate a risk (e.g. needs runtime, or a file you
couldn't read), say so explicitly rather than implying it's clean.

### Severity

Rate each finding **Critical / High / Medium / Low**. Start from the checklist's baseline severity
for that risk, then adjust for the actual context:
- Raise it when the issue is reachable by untrusted input, exposes secrets/credentials, or enables
  code execution / full-system compromise.
- Lower it when access is gated, the surface is internal-only, or exploitation is impractical.

Briefly justify the rating when it deviates from the baseline.

## Phase 4 — Write the report

Determine the date with `date +%F` and write the report to:

```
<target>/security-reports/<YYYY-MM-DD>/app-audit-<repo-or-dir-name>-<HH-MM>.md
```

The timestamp keeps repeated same-day runs from overwriting each other. Create the dated subfolder if
needed (`mkdir -p`).

### Keep reports out of Git

A report is a ranked list of exploitable weaknesses with `file:line` pointers — effectively an
attacker's roadmap if it leaks. So reports should not be committed by default. To enforce this
*without* mutating a file the skill didn't author, drop a **self-contained `.gitignore` inside the
`security-reports/` directory** (which the skill creates) the first time you write a report there:

```
# <target>/security-reports/.gitignore
*
!.gitignore
```

This ignores every report but keeps the ignore rule itself, works whether or not the project is a git
repo, never touches the project's root `.gitignore`, and a team that *wants* reports tracked can simply
delete it. Create it only if it doesn't already exist. Mention to the user that you did this and why —
don't do it silently.

**Do not** write this (or any report artifact you can avoid) into a **cloned third-party repo** — that
target is read-only analysis; in that case write the report next to the clone, tell the user where it
is and that the clone is disposable, rather than leaving files behind in someone else's tree.

Use this structure:

```markdown
# Application Security Audit — <project name>

- **Date:** <YYYY-MM-DD HH:MM>
- **Target:** <path or repo URL> (commit <hash> if cloned)
- **Checklist applied:** OWASP Top 10 (A01–A10)
- **Auditor:** Claude Code (owasp-app-audit skill)

## Executive summary
<2–4 sentences: overall risk posture, counts by severity, the most urgent item.>

## Findings summary
| ID | Risk | Severity | Location | Status |
|----|------|----------|----------|--------|
| A03 | Injection | High | api/users.py:88 | ⚠️ Finding |
| ... | ... | ... | ... | ✅ No findings |

## Detailed findings
### [SEVERITY] <ID> — <Risk title>
- **Location:** `file:line` (list each affected site)
- **Evidence:** the offending code/pattern (short snippet or quote)
- **Why it matters:** the concrete attack/impact this enables
- **Suggested fix:** specific, actionable remediation — preferably a corrected snippet or exact steps

<Repeat one block per finding. Order by severity, Critical first.>

## Checklist coverage
<One line per risk A01–A10 confirming it was evaluated, including the "no findings" ones, and any
risk you could not fully assess and why. Note here if agentic surface was seen that should be
covered by owasp-agentic-audit.>
```

Every finding **must** include a concrete suggested fix — that's the part the user acts on.

After writing, tell the user the report path and give a short spoken summary (counts by severity and
the top 1–3 things to fix first).

## Reference files
- `references/owasp-web-top10.md` — Classic OWASP Top 10 (A01–A10): what each risk is and what to
  look for.
