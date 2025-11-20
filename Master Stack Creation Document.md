# Master Stack Creation Document

# Company X – Full Stack Rebuild Guide (From Zero to Production)

Last updated: 2025-11-18 | Current version: v1

## Version History (reverse chronological)

- v28 – 2025-11-18 – Added Datadog RUM + Session Replay
- v27 – 2025-11-03 – Migrated secrets from Vault to AWS Secrets Manager + external-secrets-operator
- v26 – 2025-10-21 – Introduced ArgoCD ApplicationSets for multi-region deployments
- v25 – 2025-09-30 – Added Cloudflare Zero-Trust Gateway + WARP client enforcement
- … (keep going down)

### 0. Prerequisites

- 4 Ubuntu Servers (minimal install)
- Access to OpenEDR via cloud.
- 1 Windows 11 instance.

### 1. Bootstrap

1.1 Install Splunk Enterprise 

1.2 Getting data In