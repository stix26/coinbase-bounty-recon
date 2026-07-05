# Coinbase Bug Bounty Reconnaissance

Reconnaissance pipeline output for the Coinbase bug bounty program on HackerOne.

## Session: 2026-07-03

- **Subdomains enumerated:** 428 (coinbase.com, cbhq.net, coinbase-corp.com)
- **Live hosts found:** 116
- **CORS findings:** 18 (1 potentially reportable)
- **Exposed files:** None confirmed
- **GitHub secrets:** None found
- **Reportable findings:** 1 candidate — fp.coinbase.com CORS

## Scope

- `*.coinbase.com` (max severity: critical)
- `*.cbhq.net` (max severity: critical)
- `*.coinbase-corp.com` (max severity: critical)
- Key URLs: api.coinbase.com, pro.coinbase.com, prime.coinbase.com, commerce.coinbase.com, custody.coinbase.com, cloud.coinbase.com
- Source code: `github.com/coinbase/cb-mpc`, `github.com/coinbase/cb-mpc-go`

## Methodology

1. Subdomain enumeration (subfinder)
2. HTTP probing (curl)
3. CORS misconfiguration testing
4. Exposed file scanning
5. GitHub commit analysis for secret leaks
6. API endpoint discovery
