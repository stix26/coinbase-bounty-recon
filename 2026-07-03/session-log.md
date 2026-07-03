# Coinbase Bug Bounty Recon - July 3, 2026

**Target:** Coinbase (HackerOne handle: `coinbase`)  
**Duration:** 1 session  
**Status:** 1 CORS finding to evaluate

---

## Scope Summary

- **Eligible scopes:** 28 (URLs + wildcards + source code + mobile)
- **Key wildcards:** `*.coinbase.com` (critical), `*.cbhq.net` (critical), `*.coinbase-corp.com` (critical)
- **Key URLs:** coinbase.com, api.coinbase.com, pro.coinbase.com, prime.coinbase.com, institutional.coinbase.com, commerce.coinbase.com, custody.coinbase.com
- **Source code:** `github.com/coinbase/cb-mpc`, `github.com/coinbase/cb-mpc-go`
- **Bounties:** Yes | **State:** Public, open for submissions

---

## Recon Pipeline

### 1. Subdomain Enumeration (subfinder)

| Domain | Subdomains Found |
|--------|-----------------|
| coinbase.com | 202 |
| cbhq.net | 159 |
| coinbase-corp.com | 67 |
| **Total unique** | **428** |

### 2. Live Host Probing

- 428 subdomains probed → **116 live hosts** (2xx/3xx/403)
- 11 key URL targets probed separately

**Notable live hosts:**
- api.coinbase.com, wallets, exchange, prime, cloud, commerce
- Internal tools on *.coinbase-corp.com (munki, confluence, jira, go.zero)
- Status pages: status.coinbase.com, status.*.coinbase.com (7 total)
- Sandbox: public-sandbox.exchange.coinbase.com, ws-feed-public.sandbox.exchange.coinbase.com
- Dev/staging: accounts, cdp, cde, chd, keys-beta, tokenmanager

### 3. CORS Testing

**18 hosts returned CORS headers** across three categories:

#### Category 1: Wildcard CORS (no credentials) — Informational
- accounts.coinbase.com, blockchain.wallet.coinbase.com, cdpstatus.coinbase.com
- dma.prime.coinbase.com, exchange.coinbase.com, tokenmanager.coinbase.com
- widget.coinbase.com, public-sandbox.exchange.coinbase.com
- All status.*.coinbase.com pages (cde, coinbase, commerce, custody, exchange, international, prime)
- Standard `Access-Control-Allow-Origin: *` without credentials → not exploitable

#### Category 2: Credentialed CORS — GCP IAP (Not Exploitable)
- **go.zero.coinbase-corp.com**: Mirrors any Origin with credentials. Behind Google Cloud IAP (OAuth to Google). CORS headers on 302 redirect are from GCP infrastructure, not the app. Response: "Invalid IAP credentials"
- Same pattern as Shopify's admin-sitemap.shopify.io

#### Category 3: Credentialed CORS — Potentially Reportable
- **fp.coinbase.com**: Reflects any Origin with `Access-Control-Allow-Credentials: true` on all paths
  - Part of `*.coinbase.com` wildcard (eligible, max critical severity)
  - GET returns 200 with 0-byte body
  - POST returns JSON: `{"v":"2","error":{"code":"RequestCannotBeParsed","message":"bad request"},"products":{}}`
  - Likely a feature-flag or experimentation service
  - CORS is correctly configured permissive for its intended use (internal API), but the reflect-any-origin + credentials pattern is a finding

- **search.travel.coinbase.com**: ACAC: true on 301 redirect, but no ACAO header reflected → not exploitable

### 4. Exposed File Scanning

- Tested 19 paths on 9 interesting targets + key URLs
- **No exposed sensitive files confirmed** (all paths return 404, 403, or empty)

### 5. GitHub Secret Scanning

- Scanned recent commits on coinbase/cb-mpc and coinbase/cb-mpc-go
- **No secret leaks found** — only security hardening and MPC protocol fixes

### 6. API Discovery

- Checked fp.coinbase.com for multiple methods/endpoints
- POST returns structured JSON error with CORS headers
- No uncovered API surface on other targets

---

## Summary

| Category | Result |
|----------|--------|
| CORS (wildcard, no creds) | 15 hosts — informational only |
| CORS (GCP IAP) | 1 host — not exploitable |
| CORS (reflect + creds) | 1 host — fp.coinbase.com |
| Exposed files | None |
| GitHub secrets | None |
| **Reportable** | **Possibly fp.coinbase.com CORS** |

### fp.coinbase.com Assessment

- `*.coinbase.com` is in scope with max critical severity
- Reflected Origin + `Access-Control-Allow-Credentials: true` on a production subdomain
- Current impact limited: responses are empty (GET) or error JSON (POST)
- If any authenticated endpoint on this subdomain returns user data, cross-origin data theft is possible
- Recommend filing as medium-severity defense-in-depth finding
