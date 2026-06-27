---
doc_type: short
full_text: sources/SECURITY.md
---

# Security Policy Summary

This document defines the security policy for the project, covering which versions receive security updates and how to report vulnerabilities.

## Supported Versions

Only specific versions of the project are actively supported with security updates:

| Version | Supported |
| ------- | --------- |
| 5.1.x   | ✅ Yes    |
| 5.0.x   | ❌ No     |
| 4.0.x   | ✅ Yes    |
| < 4.0   | ❌ No     |

Notably, the latest minor release (5.1.x) and an older major release (4.0.x) are supported, while 5.0.x and anything below 4.0 are not.

## Vulnerability Reporting

The document includes a placeholder section for [[concepts/vulnerability-disclosure]] procedures. It instructs maintainers to specify:

- **Where** to submit vulnerability reports
- **Update frequency** — how often reporters can expect status updates
- **Outcomes** — what happens when a vulnerability is accepted or declined

This follows the standard responsible disclosure model, where reporters are kept informed throughout the triage and remediation process.

## Key Takeaways

- Not all versions are maintained; users should migrate to supported versions (5.1.x or 4.0.x).
- A formal [[concepts/vulnerability-disclosure]] channel is expected but not yet fully defined in this template.
- The policy structure is typical of open-source projects following [[concepts/security-policy]] best practices.