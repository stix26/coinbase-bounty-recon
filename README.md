# Coinbase Bug Bounty Recon — 2026-07-05

**HackerOne:** Coinbase  
**Scope:** coinbase.com  
**Methodology:** Full recon pipeline (subfinder → live probing → CORS → exposed files → nuclei → ffuf)

## Summary

| Step | Status | Findings |
|------|--------|----------|
| Subdomain Enumeration | ✅ | 195 subdomains found |
| Live Probing | ✅ | 8 key subdomains checked |
| CORS Testing | ✅ | **CRITICAL: exchange.coinbase.com allows wildcard origin** |
| Exposed Files | ✅ | All paths return 403 |
| Nuclei Scan | ✅ | Azure AD tenant info |
| FFUF Fuzzing | ✅ | Common paths scanned |

## CORS Results
- **exchange.coinbase.com** — `access-control-allow-origin: *`  ⚠️
- All others — No CORS headers

## Key Subdomains Live
- coinbase.com (302)
- exchange.coinbase.com (200)
- prime.coinbase.com (200)
- api.coinbase.com (301)
- pro.coinbase.com (301)
- commerce.coinbase.com (302)

## Nuclei
- **Azure Domain Tenant:** coinbase.com → Azure AD (tenant: 3d309027-3cd0-4b06-86b6-5150f8834c76)

## ⚠️ Findings to Report
1. **CORS Wildcard on Exchange** — `exchange.coinbase.com` returns `Access-Control-Allow-Origin: *`. This could allow malicious sites to make cross-origin requests. Verify if sensitive endpoints are accessible.
