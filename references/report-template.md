# Report Template & Scoring

Write `security-audit/REPORT.md` with the sections below. Be precise and evidence-driven. Never call the app secure without evidence.

---

# Security Audit Report — <app name>

**Date:** <YYYY-MM-DD> · **Auditor:** Web Offensive Security (Claude Code) · **Scope:** <repos/paths/env> · **Authorization:** <who authorized, date> · **Method:** Static <+ dynamic if applicable>

## 1. Executive Summary
2–4 paragraphs for leadership: overall posture, the few things that matter most, aggregate counts by severity, headline attack chains, and the **Security Score** with one line on what drives it.

| Severity | Count |
|----------|-------|
| Critical | |
| High | |
| Medium | |
| Low | |
| Info | |

## 2. Attack Surface
The enumerated surface: stack, entry points, trust boundaries. Include the route/endpoint table (`method | path | auth | roles | inputs | sinks | notes`). Call out unauthenticated and privileged surfaces.

## 3. Findings
One subsection per finding, ordered by severity then impact. Use this block:

### [SEV] <ID> — <Title>
- **Severity:** Critical/High/Medium/Low/Info · **CVSS 3.1:** `<vector>` = `<score>` (if applicable)
- **CWE:** CWE-xxx · **Verified:** yes/no
- **Location:** `path/to/file.ext:line` — function/route `<name>`
- **Description:** what the flaw is and why it exists (source→sink).
- **Evidence / PoC:** exact request + response excerpt (secrets redacted), or code excerpt. State it was run against the authorized local env, or mark theoretical.
- **Impact & exploitation scenario:** what an attacker achieves, preconditions, realistic scenario.
- **Recommended fix:** concrete, code-level remediation (parameterize, encode, add authz check, pin dep, set header…). Include a corrected snippet where useful.

## 4. Attack Chains
Each chain: ordered steps linking finding IDs, preconditions, and the escalated end impact (e.g. ATO, RCE, full data breach). Explain why the chain's severity exceeds its parts.

## 5. Remediation Priority
Ranked backlog: `Priority | Finding ID(s) | Severity | Effort (S/M/L) | Why now`. P0 = fix immediately.

## 6. Security Score
State the score and show the subtraction. Formula below.

## 7. Validation Checklist
Per OWASP-style category (Injection, XSS, Authn, Authz/IDOR, Session, CSRF, CORS, SSRF, Path/Upload, Deserialization, JWT/OAuth, Secrets, Headers, Deps, Config/Infra, Business logic): `Tested? (static/dynamic/no) · Result (pass/finding IDs/not-applicable) · Notes`. This makes coverage auditable and honest about gaps.

---

## Security Score formula (0–100)

Start at **100**, subtract per finding by severity, floor at 0:

| Severity | Penalty each |
|----------|--------------|
| Critical | 25 |
| High | 12 |
| Medium | 5 |
| Low | 2 |
| Info | 0 |

- Cap total so any single **Critical** forces the score ≤ 40, and any **High** forces ≤ 70 (posture must reflect the worst real risk, not the average).
- Verified findings count at full penalty; unverified static-only findings at 60% penalty (round up).
- Band: 90–100 Strong · 75–89 Good · 50–74 Needs work · 25–49 Weak · 0–24 Critical.

Always show the arithmetic so the score is reproducible.
