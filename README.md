# Secure-Web-Infra-Lab
# Secure Web Infrastructure Project

A practical cybersecurity training project: creating a web application that can be accessed by the public. Protected, watched and ready for a test of its security.

## Overview

This project was given as part of a cybersecurity co-op training program. The goal was to create a web application that can be seen on the internet and is protected by multiple layers of security. Then the security was tested by doing red-teaming on my own. All to get ready, for a real penetration test done by mentors.

**Architecture flow:**

```
User => Cloudflare (WAF / DNS / DDoS) => Nginx (reverse proxy) => Flask (app) => PostgreSQL (database)
                                              ↓
                          Wazuh (SIEM/EDR) <= Suricata (NDR)
```

## Stack

| Layer | Tool |
|---|---|
| Hosting | Linux VPS (Ubuntu 24.04) |
| Edge / CDN | Cloudflare (DNS, WAF, DDoS protection, geo-restriction) |
| Reverse proxy | Nginx |
| Application | Flask (Python) |
| Database | PostgreSQL |
| Containerization | Docker Compose |
| SIEM / EDR | Wazuh (manager, indexer, dashboard) |
| NDR / IDS | Suricata |
| Brute-force protection | fail2ban + Wazuh Active Response |
| Vulnerability scanning | Trivy (container images), Nikto (live web app) |
| Dashboards | Grafana (connected to Wazuh's OpenSearch data) |

## What was built

- **Hardened SSH access** — key-only authentication (ed25519), password login disabled
- **Firewall layering** — host-level firewall combined with Cloudflare's edge protection, restricting access to legitimate traffic paths
- **Full application stack** deployed via Docker Compose: reverse proxy, backend, and database, each isolated in its own container
- **SIEM/EDR deployment** — Wazuh for centralized logging, file integrity monitoring, and automated active response to brute-force attempts
- **NDR integration** — Suricata network monitoring piped directly into the SIEM for unified visibility
- **Web application** with a staff login portal (session-based auth, hashed credentials) and additional pages built out to give the environment more realistic attack surface

## Security testing performed

- **Vulnerability scanning** of all container images using Trivy — identified and remediated fixable issues (e.g., outdated base images), documented and risk-assessed issues that couldn't be fixed locally (e.g., vulnerabilities in upstream helper binaries with no exploitable code path in this context)
- **Live web application scanning** using Nikto
- **Manual penetration testing** — directory traversal attempts, cookie security audits, SQL injection review
- **Real-world validation** — the deployed detection stack (Suricata + Wazuh + fail2ban) successfully identified and responded to genuine external brute-force attempts against SSH, confirmed by cross-referencing source IPs in the SIEM

## Vulnerabilities found and fixed

- Session cookies missing the `Secure` flag (cookie-theft/MITM risk over unencrypted connections)
- Hardcoded, guessable application secret key left over from a project template
- Outdated base container images carrying known CVEs
- Missing explicit security headers on web responses

## Key lessons

- Container orchestration tools (like Docker) can manage their own firewall rules independently of the host's firewall — a mismatch here can silently undermine intended network restrictions
- A spike in security alerts isn't automatically an attack — verifying source data before reacting is essential
- Backup/restore tooling that works well on physical machines doesn't always work reliably in virtualized cloud environments — always test restore procedures, not just backup creation
- Layered defense works: even when one control has a gap, other independent layers (authentication method, network restrictions, detection/response automation) can still hold the line

## Status

Core infrastructure is complete, self-tested, and monitored. A formal third-party penetration test is the next planned milestone.

---

*This repository documents the architecture and lessons learned. Live infrastructure details (domain, IP addresses, credentials, and unresolved security gaps) are intentionally omitted pending completion of the penetration test.*
