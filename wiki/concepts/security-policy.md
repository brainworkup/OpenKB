---
sources: [summaries/top_level.md, summaries/LICENSE.md, summaries/SECURITY.md]
brief: Formal documents establishing version support, vulnerability disclosure, and legal liability limits for software projects.
---

# Security Policy

A **security policy** is a formal document that communicates essential information to users and contributors of a software project, covering which versions receive security patches, how to report vulnerabilities responsibly, and the legal framework governing use and liability. Security policies are a standard practice in open-source and commercial software projects, establishing transparency and trust between maintainers and users.

## Version Support Matrix

A core component of any security policy is the **supported versions table**, which tells users which releases will receive fixes if a vulnerability is discovered. Unsupported versions receive no patches, giving users a clear signal to upgrade.

The policy documented in [[summaries/SECURITY]] uses this structure:

| Version | Supported |
| ------- | --------- |
| 5.1.x   | ✅        |
| 5.0.x   | ❌        |
| 4.0.x   | ✅        |
| < 4.0   | ❌        |

This pattern — supporting the latest minor release and a stable older major release, while dropping intermediate versions — is a common lifecycle strategy for balancing backward compatibility with maintenance burden.

## Vulnerability Disclosure

The second major component is the [[concepts/vulnerability-disclosure]] process. A well-defined disclosure procedure typically specifies:

- **Reporting channel** — where to submit a vulnerability (e.g., a private email, a GitHub Security Advisory form)
- **Response timeline** — how quickly reporters can expect an acknowledgment and status updates
- **Resolution outcomes** — what happens when a vulnerability is confirmed (patch, CVE issuance) or rejected

This follows the principle of responsible disclosure, where security issues are reported privately to maintainers before public announcement, giving teams time to develop and deploy fixes.

## Legal Framework: Licensing and Liability

Security policies do not exist in isolation — they operate alongside the software's **license**, which establishes the legal boundaries of use, redistribution, and liability. Multiple permissive license models appear among documented projects, each sharing a common philosophy of disclaiming warranties while permitting broad use.

### BSD 2-Clause License (Michele Simionato)

The [[summaries/LICENSE]] document for Michele Simionato's software (copyright 2005–2026) adopts the **BSD 2-Clause (Simplified BSD) License**, one of the most minimal permissive open-source licenses available. Its security-relevant provisions include:

- **"AS IS" Disclaimer**: The software is provided without any implied warranties of merchantability, fitness for a particular purpose, or non-infringement. Users bear full responsibility for assessing security risks.
- **Limitation of Liability**: Copyright holders and contributors are explicitly not liable for any direct, indirect, incidental, special, exemplary, or consequential damages — including those arising from security vulnerabilities, data loss, or system interruption.
- **Minimal Redistribution Conditions**: Source and binary redistributions must retain the copyright notice and disclaimer, ensuring the liability waiver travels with the code.

The BSD 2-Clause model imposes no restrictions on endorsement or promotion (unlike the 3-Clause variant), making it especially permissive and widely compatible with other licenses.

### BSD 3-Clause License (Pallets Project)

The Pallets project (copyright 2014) adopts the **BSD 3-Clause License**, adding one further restriction over the 2-Clause version. Its security-relevant provisions include:

- **"AS IS" Disclaimer**: Same warranty disclaimer as the 2-Clause variant.
- **Limitation of Liability**: Copyright holders and contributors are not liable for any damages arising from software use.
- **Name Restrictions**: The Pallets name and contributor names cannot be used to endorse or promote derived products without written permission, protecting the project's reputation in security communications.

The BSD 3-Clause model is common for foundational libraries and frameworks (such as those maintained by the [[concepts/pallets-project]]) where broad adoption is prioritized alongside minimal legal friction.

### Apache License, Version 2.0

The Apache License 2.0, used by aio-libs contributors, reinforces security policy goals through similar but more detailed provisions:

- **Disclaimer of Warranty (Section 7)**: The Work is provided "AS IS" without warranties of any kind.
- **Limitation of Liability (Section 8)**: Contributors are not liable for direct, indirect, special, incidental, or consequential damages arising from use of the Work.
- **Patent License Termination (Section 3)**: If a user initiates patent litigation against the project, their patent licenses terminate — discouraging adversarial legal action against security researchers or contributors.

### Common Thread

Despite their differences, the BSD 2-Clause, BSD 3-Clause, and Apache 2.0 licenses share a foundational security-policy implication: **legal protection for contributors is established by the license itself**, while maintainers separately commit to transparent vulnerability disclosure and version support through the security policy document. This division of responsibility — legal shield via license, operational commitment via security policy — is a hallmark of mature [[concepts/open-source-licensing]] practice.

## Relationship to Privacy-First Software

Security policies are closely related to broader privacy-first software principles. Projects handling sensitive data — such as those involving [[concepts/phi-data-handling]] or [[concepts/clinical-data-privacy]] — especially benefit from explicit, well-maintained security policies, as breaches in such contexts carry significant legal and ethical consequences. The "AS IS" warranty disclaimer present in all three license variants makes it even more critical for users of such software to independently verify security posture before deploying in sensitive environments.

## Related Pages

- [[summaries/SECURITY]] — Source document defining the version support matrix and vulnerability reporting template
- [[summaries/LICENSE]] — BSD 2-Clause License as applied to Michele Simionato's software, establishing warranty disclaimers and liability limits
- [[concepts/vulnerability-disclosure]] — The process by which security issues are reported and resolved
- [[concepts/open-source-licensing]] — Broader landscape of permissive and copyleft license models
- [[concepts/pallets-project]] — The Pallets organization, copyright holder of the BSD 3-Clause license documented here
- [[concepts/phi-data-handling]] — Handling of protected health information in sensitive software contexts
- [[concepts/clinical-data-privacy]] — Privacy considerations specific to clinical and health-related software

See also: [[summaries/top_level]]