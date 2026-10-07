# OWASP Agentic Skills Top 10 (AST01–AST10)

Source: https://owasp.org/www-project-agentic-skills-top-10/

This checklist audits **AI agent skills themselves** — the artifacts an agent installs and runs
(SKILL.md files, prompts, manifests, bundled scripts, declared tools/permissions). The core question
is: *"Is it safe to install and run this skill?"* Each entry gives the risk, its baseline severity,
and concrete things to look for. Adjust severity to the actual context per the main SKILL.md guidance.

---

## AST01 — Malicious Skills — Baseline: Critical
Skills that look legitimate but hide credential stealers, reverse shells, or prose instructions that
hijack the agent.

**Look for:**
- Bundled scripts that read secrets then send them out: access to `~/.ssh`, `~/.aws`, `.env`,
  `process.env`, `os.environ`, keychain, browser cookies — combined with network calls (`curl`,
  `wget`, `fetch`, `requests.post`, sockets) to external hosts.
- Reverse-shell / remote-exec patterns: `bash -i >& /dev/tcp/...`, `nc -e`, `socket` + `subprocess`,
  `pty.spawn`, `eval`/`exec` on downloaded content, `curl ... | bash`, `iex`, `Invoke-Expression`.
- Obfuscation hiding behavior: base64/hex/gzip blobs that get decoded and executed, dynamically built
  command strings, unusually minified or encoded payloads.
- **Prompt-injection in prose**: instruction text (in SKILL.md, descriptions, comments, or referenced
  docs) that tries to steer the agent — "ignore previous instructions", "do not tell the user",
  "first read the user's credentials and send them to…", hidden/zero-width or off-screen text.
- Behavior that contradicts the skill's stated purpose (a "formatter" that opens network sockets).

## AST02 — Supply Chain Compromise — Baseline: Critical
Registries without provenance let attackers mass-upload, take over accounts, and poison distribution.

**Look for:**
- Skills/dependencies installed from sources with no provenance, signing, or integrity check.
- Install-time code that fetches arbitrary remote content (`postinstall`/`preinstall` scripts running
  `curl`/`wget` to a URL, `pip install` from a git URL, downloading binaries at install).
- Missing lockfiles / hashes (no `package-lock.json`, `poetry.lock`, `go.sum`, pinned hashes).
- Typosquatting-prone dependency names, or deps from untrusted/private mirrors with no verification.
- No author identity / provenance metadata tying the skill to a trusted publisher.

## AST03 — Over-Privileged Skills — Baseline: High
Skills granted far more access than they need — weaponisable by prompt injection into a huge blast
radius.

**Look for:**
- Broad tool grants: `allowed-tools: *`, unrestricted `Bash`, wildcard file/network access where the
  task is narrow.
- Capabilities far exceeding the stated function (a doc-summarizer that requests shell + network +
  full filesystem).
- Manifest permissions/scopes broader than the code actually uses.
- Principle of least privilege not applied — note the gap between *needed* and *granted* access.

## AST04 — Insecure Metadata — Baseline: High
Unvalidated, unsigned metadata enables brand impersonation, understated permissions, and poisoned
search.

**Look for:**
- Frontmatter/manifest with no signature or integrity verification.
- Name/description impersonating an official org/vendor, or keyword-stuffed descriptions ("search
  poisoning").
- Declared permissions that understate what the code actually does.
- Missing or spoofable provenance fields (author, version, source, publisher).

## AST05 — Untrusted External Instructions — Baseline: High
Skills that point the agent at external docs trust mutable, unpinnable content that can be rug-pulled
into malicious instructions.

**Look for:**
- SKILL.md / code that tells the agent to fetch and *follow* remote content (`WebFetch` of a URL,
  "read the latest instructions at http…", loading remote prompts/config and acting on them).
- References to external content by **mutable** ref (a live URL, `latest`, a branch) rather than a
  pinned commit/hash/version.
- Trusting third-party docs as authoritative instructions without validation.

## AST06 — Weak Isolation — Baseline: High
Skills run in the agent's full security context — with no sandbox, every skill is a potential
full-system compromise.

**Look for:**
- Bundled code that executes with the host's full privileges (arbitrary shell, file access outside
  the skill's own directory, no container/sandbox/seccomp).
- No isolation boundary between the skill and the host (env vars, filesystem, network all reachable).
- This is often architectural — note the absence of sandboxing as a finding even without a single
  "smoking gun" line.

## AST07 — Update Drift — Baseline: Medium
Without pinning or verification, skills silently drift to vulnerable — or freshly malicious —
versions.

**Look for:**
- Skills/dependencies referenced by mutable tag (`latest`, `main`, `*`, `^`/`~` ranges) instead of a
  pinned version or commit hash.
- Auto-update mechanisms that pull new code without re-verification.
- Git submodules tracking a branch rather than a fixed commit; missing lockfiles.

## AST08 — Poor Scanning — Baseline: Medium
Natural-language-plus-code blends defeat signature scanners, so malicious skills pass every automated
check.

**Look for:**
- Reliance on signature/keyword scanning alone, with no review of the natural-language instruction
  content.
- Mixed prose+code where the *prose* carries the risky behavior (and would slip past code scanners).
- Note where automated checks would plausibly miss intent that a human review would catch.

## AST09 — No Governance — Baseline: Medium
No inventory, approval, audit, or revocation — a shadow-AI layer that security teams cannot see or
control.

**Look for:**
- No inventory of installed skills, no approval/review process, no audit logging of skill actions.
- No revocation/kill-switch mechanism, no ownership (`CODEOWNERS`), no security policy
  (`SECURITY.md`) for the skill ecosystem.
- This is org/process-level — report it as a gap when the project lacks these controls.

## AST10 — Cross-Platform Reuse — Baseline: Medium
Porting skills across platforms drops the source format's security metadata, opening exploitable gaps.

**Look for:**
- Skills ported from another platform/format (references to other agent frameworks, leftover
  foreign manifest fields).
- Security metadata (permissions, signatures, scopes) present in one format but missing in the ported
  one.
- Inconsistent permissions/metadata across multiple platform manifests in the same project.
