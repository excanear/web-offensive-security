# Safe PoC Patterns

Prove exploitability **without** causing harm. Every PoC here is read-only or benign. Only run against the authorized local target. Capture the exact request and the response that proves the condition.

Golden rules:
- Prove the *mechanism*, then stop. Don't escalate beyond what demonstrates the flaw.
- Use benign markers and read-only commands. Never destroy, persist, exfiltrate real data, or degrade availability.
- Use test accounts / seed data only.
- If proving it safely is impossible, document the theoretical PoC and mark `verified: false`.

## SQL Injection
- Boolean/`SELECT 1` style: show that `' OR '1'='1' -- ` changes results, or that `1 AND 1=1` vs `1 AND 1=2` differ. For blind, a **short** time-based probe (`SLEEP(2)`) is acceptable as a one-off; never large sleeps or loops (DoS).
- Prove read access with a harmless query (e.g. `@@version`, `current_user`). **Do not** dump tables, modify, or drop anything.

## NoSQL Injection
- Send `{"$ne": null}` / `{"$gt": ""}` to show auth/filter bypass returning data that should be hidden. Stop at the first proof.

## Command Injection / RCE
- Benign proof only: `; id`, `& whoami`, `$(echo CANARY_$RANDOM)` and show the reflected output. Never read secrets, write files outside a temp canary, open shells, or establish any persistence.

## SSTI
- `{{7*7}}` / `${7*7}` → expect `49` in the response. Confirm engine, then stop (do not pivot to RCE gadgets on shared/unsafe systems; note the potential).

## XSS
- Reflected/DOM: inject a unique benign marker like `<img src=x onerror="window.__xss_canary=1">` or `"><svg/onload=console.log('XSS_CANARY')>` and show it executes (console log / DOM attribute), using the browser tools. No cookie theft, no real redirect, no beaconing to external hosts.
- Stored: use a clearly-labeled test record you created; show it renders unescaped. Clean up the test record after.

## SSRF
- Point the server at a **local** canary you control (e.g. `http://127.0.0.1:<your-listener>/ssrf-canary`) and show the inbound hit. Demonstrating it *can* reach `169.254.169.254` or internal hosts: prove connectivity/response-shape only; do **not** retrieve or display real cloud credentials or internal secrets — note the reachability and stop.

## Path Traversal / LFI
- Read a known-harmless marker file you placed, or a world-readable, non-sensitive file (e.g. `/etc/hostname`) to prove traversal. Do **not** read secrets, keys, or `/etc/shadow`.

## File Upload
- Upload a benign file with a mismatched extension/magic bytes to prove the filter is bypassable (e.g. a GIF-header file named `.php` containing only a harmless marker). Do **not** upload a working webshell; if the server would execute it, stop at demonstrating the upload+path and note the impact.

## IDOR / BOLA
- With test account A, request an object belonging to test account B (`/api/orders/1002`) and show A receives B's data. Use only seeded/test objects. Read-only; never modify B's data.

## Authorization / privilege escalation
- As a low-priv test user, call an admin endpoint and show it succeeds (or returns privileged data). For mass assignment, add `"role":"admin"` to a test account's update and show it sticks — then revert.

## JWT
- `alg:none`: craft an unsigned token for a **test** user and show it's accepted. Algorithm confusion: sign with the public key as HMAC secret for a test user. Never forge a real/admin identity to access real data — prove acceptance of a tampered token and stop.

## CSRF
- Build a minimal auto-submitting HTML form hitting a state-changing endpoint and show it succeeds cross-origin without a token — using a test account and a reversible/no-op action. Revert any change.

## CORS
- Send `Origin: https://attacker.example` and show the response reflects it with `Access-Control-Allow-Credentials: true`. Header inspection only — no actual cross-origin data theft.

## Open redirect
- Show `/redirect?url=https://example.com/` (a neutral, authorized destination) returns a redirect to the attacker-controlled value. Don't chain to real token theft.

## Secrets exposure
- Show the secret is present in the served bundle/response/history via a read. **Redact** the actual secret value in evidence (show first/last few chars). Recommend rotation. Never use the secret.

## Evidence to capture per PoC
- The exact request (method, path, headers, body — secrets redacted).
- The response excerpt proving the condition.
- The affected source: file, function, route, line.
- Cleanup performed (for any test record created).
