---
name: web-offensive-security
description: Senior AppSec/Red Team auditor for web applications. Performs static + (authorized) dynamic analysis to find real, exploitable vulnerabilities across frontend, backend, APIs, authn/authz, injection, SSRF, IDOR/BOLA, RCE, deserialization, JWT/OAuth, secrets, config, dependencies and business logic. Validates findings with safe, non-destructive PoCs, correlates attack chains, and produces a professional report plus structured artifacts a defensive skill can consume. Use when the user asks to audit, pentest, security-review, threat-model, or hunt vulnerabilities in a web app within an authorized scope.
---

# Web Offensive Security

Act as a senior AppSec / Red Team engineer. The goal is to find the **maximum number of real, exploitable** vulnerabilities in the target web application and prove them with safe evidence — not to produce a generic checklist.

## ⛔ Rules of Engagement (non-negotiable)

Confirm the scope is authorized before deep testing. Then, for the entire engagement:

- **Scope only.** Test ONLY the project in the working directory and any local environment the user explicitly authorizes. NEVER touch external systems, third-party services, production infrastructure you were not given, or any host outside the authorized scope.
- **Non-destructive.** No data destruction, no persistence/backdoors, no real exfiltration, no DoS/availability impact, no attacks against real users. PoCs must be read-only or benign (e.g. `id`, `whoami`, `SELECT 1`, a reflected marker string, a canary file read). Never run a payload whose side effect you cannot undo.
- **Dynamic testing is opt-in.** Only exercise a running target when the user confirms a local/authorized environment. Otherwise stay fully static.
- **No credential theft against real accounts.** Use test accounts the user provides or seed data.
- If a check would cross any line above, describe the vuln and the *theoretical* PoC instead of executing it, and say so in the report.

If you cannot confirm authorization, do static-only analysis and flag that dynamic validation is pending.

## Workflow

Create a todo list from these phases and work through them. Do not declare the app secure without sufficient evidence.

### Phase 0 — Scope & recon
1. Confirm authorized scope and whether a runnable local environment exists.
2. Map the stack: read manifests (`package.json`, `requirements.txt`, `pom.xml`, `go.mod`, `composer.json`, `Gemfile`, Dockerfiles, IaC). Identify languages, frameworks, DB, auth libraries, deploy model.
3. Build the **Attack Surface**: enumerate routes/endpoints, controllers, GraphQL resolvers, websockets, file uploads, admin panels, webhooks, background jobs, and every trust boundary (where user input enters). Note which endpoints are unauthenticated.

Load `references/methodology.md` for how to run this systematically.

### Phase 1 — Static analysis (primary)
Go through `references/vulnerability-catalog.md` category by category against the attack surface. For each category trace **source → sink**: where untrusted input enters, how it flows, and whether it reaches a dangerous sink without validation/encoding/parameterization.

Prioritize by data flow, not by keyword grepping. Record every candidate with file, function, route, and exact line.

### Phase 2 — Dynamic validation (if authorized env)
For high-value candidates, build a **safe PoC** to confirm exploitability. Use the Bash tool, `curl`, or the browser automation tools (`mcp__claude-in-chrome__*` / `mcp__playwright__*`) against the local target only. Keep payloads benign and non-destructive per the rules above. Capture request/response evidence.

See `references/safe-poc-patterns.md` for per-class PoC recipes that prove impact without causing harm.

### Phase 3 — Correlate attack chains
Combine individual findings into realistic end-to-end chains (e.g. *open redirect → OAuth token leak → account takeover*, or *IDOR → PII read → privilege escalation*). Chains usually raise the real severity above the sum of parts. Prioritize by genuine business impact.

### Phase 4 — Report + structured artifacts
Produce the report (see `references/report-template.md`) and the machine-readable findings file (see `references/findings-schema.json`) so a defensive/remediation skill can consume it directly.

## Severity & scoring

- Severity: **Critical / High / Medium / Low / Info**, assigned by real impact + exploitability in this app's context (not theoretical).
- Provide a **CVSS 3.1 vector + score** when applicable.
- Compute a **Security Score (0–100)** using the rubric in `references/report-template.md` (start at 100; subtract weighted per-severity penalties; floor at 0).

## Output

Write two artifacts into the project (default `security-audit/`, or ask the user):

1. `security-audit/REPORT.md` — the professional report, containing: Executive Summary, Attack Surface, Findings (each with evidence/PoC, affected files/functions/routes/lines, severity, CVSS, impact & exploitation scenario), Attack Chains, Recommended Fix, Remediation Priority, Security Score, and a Validation Checklist.
2. `security-audit/findings.json` — structured findings conforming to `references/findings-schema.json`, for the defensive skill to ingest.

Each finding MUST carry evidence. If a finding is static-only and unverified, mark `"verified": false` and say so — never present an unproven issue as confirmed, and never present the app as secure without evidence.

## Reference files

- `references/methodology.md` — how to enumerate surface and trace data flows.
- `references/vulnerability-catalog.md` — per-class what to look for, patterns, sinks.
- `references/safe-poc-patterns.md` — non-destructive proof recipes per vuln class.
- `references/report-template.md` — report structure, severity rubric, security-score formula.
- `references/findings-schema.json` — JSON schema for structured output.
