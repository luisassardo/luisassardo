# Security Audit & Profile Improvement Plan

**Repository:** luisassardo/luisassardo  
**Audit Date:** 2026-04-13  
**Scope:** Public profile, all public repositories (`cartas-a-desconocidos`, `monetization-tool`, `pulso-digital`)

---

## Executive Summary

No API keys, tokens, or credentials were found directly exposed across public repositories. However, **two critical vulnerabilities** were identified in `cartas-a-desconocidos` (hardcoded default credentials), one **data privacy concern** in `monetization-tool`, and several profile-level improvements that would significantly raise trust and credibility.

---

## Findings

### CRITICAL — `cartas-a-desconocidos`

#### 1. Hardcoded Default Admin Password
**File:** `server.js`  
```js
const ADMIN_PASS = process.env.ADMIN_PASSWORD || 'cartas-admin-2024';
```
If `ADMIN_PASSWORD` is not set in the environment, the admin password falls back to a publicly visible, predictable string. Anyone who reads the source code can authenticate as admin.

**Fix:** Remove the fallback. Fail loudly if the variable is not set:
```js
const ADMIN_PASS = process.env.ADMIN_PASSWORD;
if (!ADMIN_PASS) throw new Error('ADMIN_PASSWORD environment variable must be set');
```

#### 2. Weak Default Encryption Key
**File:** `server.js`  
```js
const ENC_KEY = process.env.ENCRYPTION_KEY || 'default-key';
```
If `ENCRYPTION_KEY` is missing, all personal data (names, addresses) is encrypted with `'default-key'`, making encryption completely ineffective.

**Fix:** Same pattern — no fallback, hard fail on startup:
```js
const ENC_KEY = process.env.ENCRYPTION_KEY;
if (!ENC_KEY || ENC_KEY.length < 32) throw new Error('ENCRYPTION_KEY must be set and at least 32 characters');
```

---

### HIGH — `cartas-a-desconocidos`

#### 3. Missing CSRF Protection
State-changing `POST`/`DELETE` endpoints have no CSRF token validation. An attacker can trick an authenticated user's browser into making forged requests.

**Fix:** Add `csurf` middleware or use `SameSite=Strict` cookies (already partially done) combined with a `double-submit cookie` pattern. Alternatively, use the `lusca` package for full CSRF protection.

#### 4. No Rate Limiting
Registration, pseudonym lookup, and status endpoints have no rate limits. This enables:
- Enumeration of pseudonyms/emails
- Brute force on admin login
- Flooding the service

**Fix:** Add `express-rate-limit`:
```js
import rateLimit from 'express-rate-limit';
app.use('/api/', rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));
```

#### 5. Missing Security Headers
No `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, or `Strict-Transport-Security` headers.

**Fix:** Add `helmet`:
```js
import helmet from 'helmet';
app.use(helmet());
```

---

### MEDIUM — `cartas-a-desconocidos`

#### 6. File Upload MIME-type Spoofing
File type validation relies only on the MIME type sent by the client, which can be forged. A malicious file with a spoofed `image/png` MIME type can be uploaded.

**Fix:** Use `file-type` (reads magic bytes) instead of trusting client-provided MIME:
```js
import { fileTypeFromBuffer } from 'file-type';
const type = await fileTypeFromBuffer(buffer);
if (!['image/jpeg', 'image/png'].includes(type?.mime)) return res.status(400).send('Invalid file');
```

#### 7. No Input Validation on Email/Text Fields
Email format is not validated; field lengths are not enforced. Malformed data can be stored and may cause downstream issues.

**Fix:** Use `zod` or `express-validator` for input validation on all endpoints.

---

### MEDIUM — `monetization-tool`

#### 8. `data/accounts.csv` — Aggregated Facebook Account Data in Public Repo
**File:** `data/accounts.csv`  
The file contains Facebook Page IDs, account handles, real names, subscriber counts, monetization dates, and geographic data for Central American content creators.

While individual data points may be public on Facebook, committing an aggregated dataset to a public GitHub repo raises:
- **GDPR concerns** (you are based in Germany, EU law applies — even public data aggregation can fall under Art. 6 GDPR)
- **Meta/Facebook Terms of Service** concerns regarding data collection and redistribution
- **Reputational risk** for a security researcher: profiling third-party accounts without clear consent

**Fix:** 
- Remove `data/accounts.csv` from the repository
- Add `data/` to `.gitignore`
- Replace with an anonymized sample or a data dictionary showing schema only
- Add a note in README about how to obtain/generate the dataset

#### 9. Missing License — `monetization-tool`
The repository has no `LICENSE` file. This means the code is technically "All Rights Reserved" and no one can legally use, modify, or contribute to it.

**Fix:** Add an appropriate license (MIT, Apache 2.0, or GPL depending on your intent).

---

### LOW — Profile README

#### 10. Surveillance Tool Forks Listed Prominently
The profile README lists three forks of surveillance/OSINT tools (`Osintgram`, `TGcollector`, `CCTV`) as featured projects. For someone positioning as a **responsible security researcher**, this may raise questions without context around purpose and ethics.

**Recommendation:** Add a brief ethical disclaimer alongside these tools, e.g.:
> *These forks are used for defensive research and OSINT methodology study. All use is performed in authorized, controlled environments.*

Or move them out of Featured Projects and into a separate "Research Tools" section with that disclaimer.

#### 11. No Pinned Repositories
No repositories are pinned on the profile, so GitHub shows repos by default sorting. Pin your best/most representative work to make a strong first impression.

---

## Improvement Plan — Priority Order

| Priority | Action | Repository | Effort |
|---|---|---|---|
| P0 | Remove hardcoded fallback `ADMIN_PASSWORD` | cartas-a-desconocidos | 5 min |
| P0 | Remove hardcoded fallback `ENCRYPTION_KEY` | cartas-a-desconocidos | 5 min |
| P1 | Add `express-rate-limit` to all auth endpoints | cartas-a-desconocidos | 30 min |
| P1 | Add `helmet` for security headers | cartas-a-desconocidos | 10 min |
| P1 | Remove `data/accounts.csv` from public repo | monetization-tool | 10 min |
| P2 | Add CSRF protection | cartas-a-desconocidos | 1 hour |
| P2 | Replace MIME-only file validation with `file-type` | cartas-a-desconocidos | 30 min |
| P2 | Add input validation with `zod` or `express-validator` | cartas-a-desconocidos | 1 hour |
| P2 | Add `LICENSE` to `monetization-tool` | monetization-tool | 5 min |
| P3 | Add ethical disclaimer to surveillance tool links | luisassardo/luisassardo | 10 min |
| P3 | Pin best repositories on GitHub profile | Profile settings | 5 min |
| P3 | Add SECURITY.md to `monetization-tool` | monetization-tool | 20 min |

---

## Profile Trustworthiness Improvements

Beyond security hardening, these steps would significantly improve how the profile is perceived by employers, collaborators, and the security community:

1. **Enable GitHub 2FA** (if not already active) — Verifiable via the "Security" badge GitHub displays
2. **Add a CONTRIBUTING.md** to active projects — Shows maturity and openness to collaboration
3. **Add CI/CD badges** to active repos (e.g., passing tests, linting) — Signals code quality
4. **Tag repositories with topics** — Makes them discoverable (`osint`, `security`, `ghost-theme`, etc.)
5. **Pin repositories** — Show `cartas-a-desconocidos` and `pulso-digital` first; they are the most polished
6. **Add a public GPG key** to your GitHub account — Lets you sign commits, which is a strong signal of identity and integrity for a security researcher
7. **Enable secret scanning alerts** on all repos (GitHub Settings > Security) — GitHub will automatically alert you if a secret is ever committed
8. **Add Dependabot** to active projects — Automated dependency vulnerability alerts and PRs

---

## What Is Already Good

- `cartas-a-desconocidos` has a `SECURITY.md`, MIT license, `.env.example`, and gitignored database
- `pulso-digital` has `SECURITY.md`, `.gitignore`, MIT license, and clean structure
- No API keys, tokens, or credentials found directly committed in any public repo
- Private repos (`osint-training`, `tweet2ghost`) are correctly kept private
- Profile README is professional, well-structured, and accurately represents your work

---

*Generated by Claude Code security audit — luisassardo/luisassardo*
