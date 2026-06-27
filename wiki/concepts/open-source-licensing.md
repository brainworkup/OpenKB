---
sources: [summaries/top_level.md, summaries/entry_points.md, summaries/table_API_readme.md, summaries/LICENSE.md]
brief: Legal frameworks governing software reuse, distribution, and modification rights in open-source projects.
---

# Open-Source Licensing

Open-source licensing is the legal framework that governs how software can be used, copied, modified, and distributed. By attaching a license to a software project, copyright holders specify the rights and restrictions granted to downstream users and contributors.

## Why Licensing Matters

- **Legal clarity** — Without a license, copyright law defaults to "all rights reserved," meaning no one can legally use, copy, or modify the code without explicit permission.
- **Collaboration** — Clear licensing enables contributors to confidently participate in a project.
- **Commercial use** — Licenses define whether and how software may be used in commercial products.

## Common License Types

### MIT License

One of the most permissive and widely-used open-source licenses. Key characteristics:

- **Permissions**: Use, copy, modify, merge, publish, distribute, sublicense, and sell copies of the software.
- **Conditions**: The copyright notice and permission notice must be included in all copies or substantial portions of the software.
- **Warranty disclaimer**: Software is provided "AS IS" without warranty of any kind — no guarantees of merchantability, fitness for a particular purpose, or non-infringement.
- **Liability**: Authors and copyright holders are not liable for any claims or damages arising from use of the software.

### BSD 3-Clause License

The BSD 3-Clause License (also called the "New BSD" or "Modified BSD" license) is a permissive license closely related to the MIT License but with an explicit non-endorsement clause. It is used by [Encode OSS Ltd](https://www.encode.io/) (copyright © 2019) for projects such as Starlette and HTTPx. Key characteristics:

- **Condition 1 (source)**: Redistributions of source code must retain the original copyright notice, conditions, and disclaimer.
- **Condition 2 (binary)**: Redistributions in binary form must reproduce the copyright notice, conditions, and disclaimer in accompanying documentation or materials.
- **Condition 3 (non-endorsement)**: The name of the copyright holder and contributors may not be used to endorse or promote derived products without prior written permission.
- **Warranty disclaimer**: Software is provided "AS IS" with no express or implied warranties, including no warranty of merchantability or fitness for a particular purpose.
- **Liability**: Copyright holders and contributors are not liable for any direct, indirect, incidental, special, exemplary, or consequential damages arising from use of the software, including loss of use, data, or profits, or business interruption.

The BSD 3-Clause License is commonly adopted in the Python ecosystem and is considered a permissive license because derivative works are not required to carry the same license (unlike copyleft licenses). See [[summaries/LICENSE]] for the full BSD 3-Clause license text as used by Encode OSS Ltd.

### Apache 2.0

Permissive, with an explicit patent grant and attribution requirements.

### GPL (GNU General Public License)

A copyleft license requiring derivative works to carry the same license.

### LGPL

A weaker copyleft license often used for libraries.

## License Components

Most open-source licenses share common structural elements:

1. **Copyright notice** — Identifies the rights holder(s) and year.
2. **Grant of permissions** — Specifies what users are allowed to do.
3. **Conditions** — Requirements users must fulfill (e.g., preserving notices, non-endorsement).
4. **Disclaimer** — Warranty exclusions and liability limitations.

## Permissive vs. Copyleft

| License | Type | Derivative License Required? | Patent Grant | Non-Endorsement Clause |
|---|---|---|---|---|
| MIT | Permissive | No | No | No |
| BSD 3-Clause | Permissive | No | No | Yes |
| Apache 2.0 | Permissive | No | Yes | No |
| GPL | Copyleft | Yes (same) | No | No |
| LGPL | Weak Copyleft | Partial | No | No |

## Related Documents

- [[summaries/LICENSE]] — BSD 3-Clause License text (Encode OSS Ltd, 2019)
- [[summaries/entry_points]] — Project entry point configuration
- [[summaries/top_level]] — Top-level project structure
- [[summaries/table_API_readme]] — API documentation overview

See also: [[concepts/security-policy]], [[concepts/vulnerability-disclosure]], [[concepts/privacy-first-software]]