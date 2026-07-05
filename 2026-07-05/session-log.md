# Bug Bounty Session — July 5, 2026

**HackerOne User:** stix26 (STILES SEYMENS)
**GitHub User:** stix26
**Date:** 2026-07-05
**Duration:** 1 session

---

## Summary

Recon against **8 targets** discovered via HackerOne API. **8 GitHub repos created/updated**, findings filed. Notable CORS findings on Coinbase Exchange and Priceline.

**Targets covered:**
- LinkedIn, Coinbase, X/xAI, Priceline, AT&T, Vimeo, Tinder, Phabricator

**Key findings:**
| Target | Finding | Severity |
|--------|---------|----------|
| Coinbase | `exchange.coinbase.com` — CORS wildcard (`*`) on production trading domain | Medium |
| Coinbase | `api.exchange.coinbase.com` — CORS wildcard (`*`) on REST API | Medium |
| Priceline | `www.priceline.com` — Reflective CORS + `allow-credentials: true` | Low (WAF-blocked) |
| X/xAI | `www.x.com`, `ads.x.com`, `business.x.com` — Reflective CORS | Informational |

---

## Repos Created/Updated

| Repo | Content | Status |
|------|---------|--------|
| linkedin-bounty-recon | 12,339 subdomains, CORS tests | Pushed |
| coinbase-bounty-recon | 489 subdomains, CORS wildcard finding | Pushed |
| x-bounty-recon | 706 subdomains, reflective CORS | Pushed |
| priceline-bounty-recon | CORS with credentials test | Pushed |
| att-bounty-recon | 22,202 subdomains | Pushed |
| vimeo-bounty-recon | Initial recon | Pushed |
| tinder-bounty-recon | Initial recon | Pushed |
| phabricator-bounty-recon | Initial recon | Pushed |

---

## Target Details

### 1. LinkedIn (`linkedin.com`)

**Method:** Subdomain enumeration → live probing → CORS testing → exposed file checks

**Results:**
- 12,339 subdomains enumerated
- Live public hosts: www(200), developer(200), engineering(200), business(200), premium(200), about(200)
- CORS: No wildcard origins — properly configured
- Exposed files: None
- Open redirects: Not tested

### 2. Coinbase (`coinbase.com`)

**Method:** Subdomain enumeration → live probing → CORS deep dive

**Results:**
- 489 subdomains enumerated
- Key hosts: www(403), prime(200), exchange(200), commerce(302)
- **CORS wildcard on `exchange.coinbase.com`:** `access-control-allow-origin: *` on root page and `/health`
- **CORS wildcard on `api.exchange.coinbase.com`:** REST API returns `access-control-allow-origin: *` with session-allow headers
- Coinbase Exchange returns 29KB HTML page with CSP and cookies

### 3. X / xAI (`x.com`)

**Method:** Subdomain enumeration → live probing → CORS testing

**Results:**
- 706 subdomains enumerated
- Key hosts: www(301 redirect), api(404), ads(302), business(302), help(403)
- **Reflective CORS:** `www.x.com`, `ads.x.com`, `business.x.com` echo back `access-control-allow-origin: <any-origin>`
- **Accepts `Origin: null`**
- No `Access-Control-Allow-Credentials: true`

### 4. Priceline (`priceline.com`)

**Results:**
- **Reflective CORS + credentials:** `www.priceline.com` echoes any origin with `access-control-allow-credentials: true`
- Accepts `Origin: null`
- Response blocked by Cloudflare WAF (403) — limited exploitability
- Sets `__cf_bm` cookie

### 5-8. AT&T, Vimeo, Tinder, Phabricator

**Results:**
- Initial recon complete
- No immediate CORS findings
- AT&T: 22,202 subdomains (large attack surface for future sessions)

---

## Tools Used

| Tool | Version |
|------|---------|
| subfinder | v2.14.0 |
| curl | system |
| HackerOne API | v1 |
| GitHub API | v3 |

---

## Next Steps

1. **Coinbase Exchange CORS wildcard** — Consider filing HackerOne report for `exchange.coinbase.com` CORS misconfiguration
2. **Priceline reflective CORS** — Investigate if any endpoints bypass Cloudflare WAF
3. **AT&T deep recon** — 22k subdomains deserves a dedicated session focusing on interesting public endpoints
4. **Vimeo/Tinder/Phabricator** — Full recon pipeline (subfinder → httpx → CORS → fuzz)
