# Methodology

A repeatable process for auditing a web app. Depth over breadth: a proven Critical beats twenty unverified "possibles".

## 1. Enumerate the attack surface

Grep-assisted, but verified by reading. Build an inventory of every place untrusted input enters the system.

- **Routes/endpoints:** route tables, decorators/annotations (`@app.route`, `@GetMapping`, `router.get`, `export async function GET`), framework file-based routing (`pages/`, `app/`, `routes/`), OpenAPI/Swagger specs, GraphQL schemas/resolvers.
- **Entry points:** request params, query strings, headers, cookies, JSON/XML/multipart bodies, path segments, websocket messages, webhook receivers, message-queue consumers, CLI args, env-driven config, uploaded file contents and filenames.
- **Trust boundaries:** client↔server, server↔DB, server↔internal service, server↔third-party API, privileged↔unprivileged code paths.
- **Auth model:** who can reach each endpoint? Classify each as public / authenticated / role-restricted / owner-restricted. Flag anything that should be restricted but is not gated.

Produce an Attack Surface table: `method | path | auth required | roles | input params | sink(s) touched | notes`.

## 2. Trace data flows (source → sink)

For each vulnerability class, work backward from dangerous **sinks** or forward from **sources**:

- Identify the sink (e.g. `cursor.execute`, `child_process.exec`, `eval`, `fs.readFile`, template render, HTTP client, `pickle.loads`).
- Walk the call graph to find whether any untrusted **source** reaches it.
- Check every mitigation on the path: validation (allowlist?), parameterization, output encoding, canonicalization, authz check. A mitigation only counts if it actually covers the tainted value.
- Note bypasses: partial encoding, blocklists, double-decoding, type juggling, mass assignment, second-order (stored then used).

## 3. Rank candidates

Score each candidate before investing in a PoC:
- **Reachability:** can an attacker actually hit it (auth, network, preconditions)?
- **Impact:** data read/write, code exec, auth bypass, lateral movement.
- **Confidence:** how sure are you from the code alone?

Spend dynamic-validation effort on high reachability × high impact first.

## 4. Validate (authorized dynamic env only)

Confirm the top candidates with safe PoCs (`safe-poc-patterns.md`). Capture evidence: the exact request, the response proving the condition, and the affected code. Prefer the minimal payload that demonstrates the flaw.

## 5. Correlate

Lay findings on the attack surface and look for chains. A low-sev open redirect plus an OAuth flow plus a permissive `postMessage` can be an account-takeover chain. Document preconditions and the full path.

## 6. Report

Write every finding with file/function/route/line, evidence, severity, CVSS, impact, fix, and priority. Mark unverified findings honestly. Emit the structured `findings.json` alongside.

## Tooling notes

- Use `Grep`/`Glob` to locate sinks and routes fast, then `Read` to confirm context — never report from a grep hit alone.
- For dynamic work use `curl` via Bash, or the browser tools against the **local** target only.
- If dependency scanners are available in-repo (`npm audit`, `pip-audit`, `osv-scanner`), you may run them read-only on the local project to enrich the dependency findings — never phone out to attack anything.
