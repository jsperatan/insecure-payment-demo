# Insecure Payment Demo

> **WARNING: This application is INTENTIONALLY VULNERABLE.** It exists only to test SAST / SCA / secret-scanning tools.
> Never deploy it, never expose it to a network, and never reuse any value from it. All secrets are fake.
> It binds to `127.0.0.1` by default.

A small Node.js/Express "payment system" (users, cards, transfers) with SQLite.

## Run

```bash
npm install
npm start          # http://127.0.0.1:3000
```

## Planted issues

### SAST / code
| Issue | CWE | Location |
|---|---|---|
| SQL injection (login, search, accounts, transfer) | 89 | `src/server.js` `/login`, `/transactions/search`, `/accounts/:id`, `/transfer` |
| OS command injection | 78 | `/receipt/export` |
| Path traversal | 22 | `/statements/download` |
| SSRF | 918 | `/webhook/test` |
| Open redirect | 601 | `/redirect` |
| `eval` of user input | 95 | `/calc/fee` |
| Insecure deserialization | 502 | `/session/restore` |
| Reflected / stored XSS | 79 | `/transactions/search`, `/notes` |
| ReDoS | 1333 | `/validate/email` |
| IDOR / broken access control / client-controlled admin | 639 / 285 | `/accounts/:id`, `/admin/users` |
| Mass assignment, prototype pollution | 915 / 1321 | `/register` |
| JWT not verified (`jwt.decode`), weak secret, no expiry | 347 | `currentUser`, `/login` |
| MD5 password hashing, ECB mode, hardcoded key, `Math.random` tokens | 327 / 321 / 338 | `src/db.js`, `/cards`, `/password-reset` |
| Plaintext PAN/CVV storage and logging | 312 / 532 | `src/db.js`, `/cards` |
| Cookie without HttpOnly/Secure/SameSite | 1004 / 614 | `/login` |
| Verbose errors / stack traces | 209 | `/login`, `/transactions/search` |
| Wildcard CORS, no security headers, no CSRF | 942 / 352 | middleware |
| TLS verification disabled | 295 | `src/config.js` |

### Secrets (`src/config.js`)
Hardcoded JWT/session secrets, DB connection string with password, AWS example keys, fake Stripe `sk_live_`, GitHub token, Slack webhook.

### SCA (`package.json`)
Pinned outdated versions with known CVEs: `express@4.16.0`, `lodash@4.17.11`, `jsonwebtoken@8.3.0`, `axios@0.18.0`, `handlebars@4.0.11`, `marked@0.3.5`, `minimist@1.2.0`, `moment@2.19.3`, `node-serialize@0.0.4`, `body-parser@1.18.2`, `cookie-parser@1.4.3`.

## Example exploits (local only)

```bash
# SQLi auth bypass
curl -X POST localhost:3000/login -H 'Content-Type: application/json' -d '{"username":"admin'"'"'--","password":"x"}'
# Reflected XSS
curl 'localhost:3000/transactions/search?q=<script>alert(1)</script>'
# Command injection
curl 'localhost:3000/receipt/export?name=x";id;"'
```
