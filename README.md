# Security Posture Review — before and after hardening

🇵🇱 [Wersja polska](README.pl.md)

I deliberately misconfigured my own Azure lab, then audited it with **Microsoft Defender for Cloud**, **Azure Policy** and a **manual review**, fixed every finding and defined how to prevent it from happening again.

📄 **Full report:** [report_EN.pdf](report_EN.pdf) · [raport_PL.pdf](raport_PL.pdf)

## Results

| Metric | Before sabotage | After sabotage | After remediation* |
|---|---|---|---|
| Secure Score | 100% | 9% | 69% |
| MCSB controls passed | 57/63 | 46/63 | 55/63 |
| Unhealthy recommendations — Storage + Key Vault | — | 9 | 3 |

\* Measured after the VM had been deleted (cost reasons), so part of the increase comes from removing the machine. The 3 remaining recommendations are not caused by the sabotage (private link / VNet rules).

**Detection coverage of the 8 introduced misconfigurations:** Defender for Cloud **6/8**, Azure Policy **7/8**, manual review only **1/8**.

## Findings

| # | Finding | Severity | Detected by |
|---|---|---|---|
| 1 | Anonymous access to a blob container | Medium | Defender + Policy |
| 2 | HTTPS not enforced (secure transfer) | High | Defender + Policy |
| 3 | Storage reachable from all networks | Medium | Defender + Policy |
| 4 | No Key Vault diagnostic logs | Low | Defender + Policy |
| 5 | No Key Vault purge protection | Medium | Defender + Policy |
| 6 | Key Vault reachable from all networks | Medium | Defender + Policy |
| 7 | SSH (22) open to the internet | High (own assessment) | **Policy only** |
| 8 | Owner role (can assign all roles) for a developer | High (own assessment) | **Manual review only** |

## Key takeaways

1. **No recommendation ≠ no problem.** Defender missed the open SSH port (only Policy flagged it) and the over-privileged developer — a tool sees configuration, not business context.
2. **Audit detects, Deny prevents.** The default ASC initiative only audits. With Deny effects, 5 of 8 misconfigurations could not have been created; with DeployIfNotExists, 6 of 8 would have been blocked or fixed automatically.
3. **Open management ports get attacked within hours.** 19 failed SSH logins from 14 IP addresses in ~4 hours on an unused VM. Key-based login stopped them — a compensating control, not a fix.

## Scope

- Pay-as-you-go subscription, one resource group, West Europe
- Storage account, Key Vault, Linux VM (D2 v3) with NSG and public IP, Log Analytics workspace, test user in Entra ID
- Defender for Cloud free tier (Foundational CSPM), Azure Policy (ASC Default / MCSB), manual review of IAM, NSG rules and SSH logs
- November 2026

## Evidence

Anonymised screenshots are in [evidence/](evidence/). Identifiers (IP addresses, subscription IDs, account names, personal data) were removed on purpose — empty fields result from anonymisation, not from the configuration. CSV exports are not published because they contain identifiers.

