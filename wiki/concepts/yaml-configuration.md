---
sources: [summaries/PERMANENT_SOLUTION_SUMMARY.md, summaries/copilot-instructions.md, summaries/cognition.instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/report-rendering-pipeline.md, summaries/brand-yml-integration.md, summaries/style-extensions.md, summaries/report-template.md, summaries/brand-and-skills.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/0004-soul-style-profile-json.md, summaries/SETUP_SUMMARY.md, summaries/README.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/report-generation.md, summaries/mcp-integration.md, summaries/template-system.md, summaries/quarto-extensions.md, summaries/overview.md, summaries/003-modular-template-structure.md, summaries/002-mcp-llm-integration.md, summaries/001-choose-quarto-typst.md, summaries/quarto.md, summaries/brand-yml-spec.md, summaries/brand-yml-in-r.md, summaries/SKILL.md]
brief: YAML configuration files encode settings, metadata, and parameters across branding, reporting, and clinical pipeline tools.
---

# YAML-Based Configuration Files

YAML (YAML Ain't Markup Language) configuration files are human-readable, structured text files used to encode settings, metadata, and parameters for software tools and frameworks. They balance machine parseability with authoring convenience, making them a popular choice for everything from CI/CD pipelines to branding systems to clinical report orchestration.

## Core Characteristics

- **Indentation-based hierarchy**: Structure is conveyed through consistent space indentation (never tabs)
- **Key-value pairs**: `key: value` syntax with optional nesting
- **Lists**: Denoted with leading hyphens (`- item`)
- **Scalars**: Strings, numbers, booleans, and nulls — with optional quoting
- **Comments**: Lines beginning with `#` are ignored by parsers
- **Array shorthand**: Inline arrays `[400, 600]` and block list style are both valid

## YAML in Branding: brand.yml

The `_brand.yml` format (see [[summaries/brand-yml-spec]] and [[summaries/brand-yml-in-r]]) is a prominent example of YAML as a configuration contract. It encodes brand guidelines into a single file that multiple tools can consume automatically. The file has five optional top-level sections: `meta`, `logo`, `color`, `typography`, and `defaults`.

### The Single Source of Truth Pattern

As established in ADR 0006 (see [[summaries/0006-brand-yml-for-cross-platform-theming]]), `_brand.yml` is adopted as the canonical source for visual identity — colors, typography, and logos — across multiple rendering targets. Rather than duplicating theme configuration per output format, a single `brand/_brand.yml` file is referenced by `_quarto.yml` via the `brand:` key, and auto-discovered by compatible tooling.

This design eliminates configuration drift across output targets and reduces maintenance burden. The same brand identity drives:
- **Quarto documents** (≥ v1.4, auto-discovery at project root)
- **Shiny apps** (R and Python, ≥ v1.9, auto-discovery)
- **Typst PDFs** (indirect, via Quarto-mediated variable translation)

| Target | Integration Method | Auto-discovery |
|---|---|---|
| [[concepts/quarto]] | `brand:` key in `_quarto.yml` | ≥ v1.4 |
| Shiny (R/Python) | Native brand.yml support | ≥ v1.9 |
| Typst (via [[concepts/typst-typesetting]]) | Quarto-mediated variable translation | Indirect |

**Typst limitation:** The brand.yml → Typst integration is indirect. Quarto translates brand colors into Typst variables, but not all brand.yml features map cleanly to the Typst typesetting system. See [[summaries/issue_branding_typst]] for known issues.

### Meta Configuration
```yaml
meta:
  name:
    short: Acme
    full: Acme Corporation International
  link:
    home: https://www.acmecorp.com
    github: https://github.com/acmecorp
```
- Supports both simple string values and extended objects with multiple sub-fields
- All URLs must include the `https://` protocol prefix
- Custom fields beyond the defined schema are permitted

### Logo Configuration
```yaml
logo:
  images:
    header: logos/header-logo.png
    icon: logos/icon.png
  small: icon
  medium:
    light: header
    dark: header-white
```
- Named image resources in `logo.images` act as reusable references
- Size variants: `small`, `medium`, `large`
- Each size accepts `light` and `dark` sub-keys for theme-aware selection
- Alt text supported via object form with `path` and `alt` keys

### Color Configuration
```yaml
color:
  palette:
    brand-blue: "#0066cc"
    brand-gray: "#666666"
  primary: brand-blue
  background: "#ffffff"
```
- Hex values **must be quoted** to avoid YAML parsing them as comments or invalid tokens
- Named palette entries act as reusable references — define once, reference everywhere
- Semantic color names (`primary`, `secondary`, `tertiary`, `success`, `info`, `warning`, `danger`, `light`, `dark`, `foreground`, `background`) map to palette entries
- Palette colors become Sass variables (`$brand-{color_name}`) when used in `defaults`

See [[concepts/brand-color-system]] for broader color strategy patterns.

### Typography Configuration
```yaml
typography:
  fonts:
    - family: Inter
      source: google
      weight: [400, 600, 700]
  base:
    family: Inter
    size: 16px
    line-height: 1.5
  monospace-inline:
    color: "#7d12ba"
    background-color: "#f8f9fa"
  link:
    weight: 600
    decoration: underline
```
- Font sources: `file` (local/remote), `google`, `bunny` (GDPR-compliant), `system`
- Typographic elements: `base`, `headings`, `monospace`, `monospace-inline`, `monospace-block`, `link`
- Each element supports: `family`, `weight`, `style`, `size`, `line-height`, `color`, `background-color`
- Simple format accepts a string (font family name); extended format accepts an object
- Weight values: numbers (`400`), arrays (`[400, 700]`), ranges (`400..900`), or named (`thin`, `normal`, `bold`)
- Base text color defaults to `color.foreground` — avoid setting it explicitly unless overriding

See [[concepts/brand-typography]] for broader typography system patterns.

### Defaults Configuration
```yaml
defaults:
  bootstrap:
    defaults:
      navbar-bg: $brand-orange
    rules: |
      .btn-primary {
        border-radius: 0.5rem;
      }
```
- Use sparingly — only when `color`, `typography`, and other standard sections are insufficient
- Supports `bootstrap` (functions, defaults, mixins, rules), `quarto` (format options), and `shiny` (theme defaults and rules)

## Skill-Driven brand.yml Workflow

The `skills/brand-yml/SKILL.md` file provides a decision tree and reference documentation for creating, modifying, and troubleshooting `_brand.yml` files. This skill-driven approach ensures practitioners have a structured guide for integrating brand identity across all supported targets, rather than relying on scattered format-specific documentation.

## YAML in Neuropsychological Report Templates

The template system for neuropsychological reporting relies heavily on YAML configuration at multiple levels. See [[summaries/template-system]] and [[concepts/modular-report-architecture]] for the broader architecture.

### _variables.yml — Patient Variables

Patient-specific data is centralized in `_variables.yml`, decoupling clinical content from report structure. In the typst-report template (see [[summaries/report-template]]), this file holds patient demographics, clinician information, pronouns, and diagnoses:

```yaml
patient: Biggie
first_name: Biggie
last_name: Smalls
age: 18
dob: "XXXX-XX-XX"
doe: "2025-01-01"
case_number: "CASE-001"
```

These variables are consumed across three contexts:
- **Quarto markdown**: `{{< var patient >}}`
- **Typst layout**: `#let patient = [{{< var patient >}}]`
- **R code**: `Sys.getenv("PATIENT")`

This multi-consumer pattern illustrates how a single YAML source of truth can drive rendering across different language and typesetting layers. See [[concepts/neuropsychological-report-variables]] for related patterns.

### _quarto.yml — Template Project Configuration

Template-level Quarto configuration controls rendering scope and metadata sourcing. In the typst-report template, `_quarto.yml` defines four format targets — pediatric, adult, forensic, and vanilla Typst — each with per-format font and paper settings:

```yaml
project:
  type: default
  title: "typst-report"
  execute-dir: project

render:
  - template.qmd

metadata-files:
  - _variables.yml
```

The `metadata-files` key links `_variables.yml` into the Quarto rendering context, making patient variables available throughout the document. The `brand:` key similarly links `_brand.yml` for visual identity. See [[summaries/quarto]] and [[concepts/quarto]] for broader Quarto configuration patterns.

### config.yml — Pipeline Processing Configuration

A tertiary `config.yml` file controls data I/O, processing flags, report format selection, and LLM/MCP endpoint settings. This separation of concerns — patient data in `_variables.yml`, rendering in `_quarto.yml`, pipeline behavior in `config.yml` — is a hallmark of the modular report architecture:

```yaml
default:
  patient:
    name: Biggie
    age: 18
  processing:
    use_duckdb: yes
    parallel: yes
```

The `use_duckdb` flag connects to [[concepts/duckdb-as-vector-store]] patterns used elsewhere in the pipeline for clinical data management. LLM endpoint settings in `config.yml` configure the Ollama backend used by the `neuro2` R package for narrative generation.

### YAML Frontmatter in QMD Files

Individual Quarto Markdown section files carry YAML frontmatter for report metadata:

```yaml
---
title: NEUROCOGNITIVE EXAMINATION
patient: Biggie
name: Smalls, Biggie
doe: "YYYY-MM-DD"
date_of_report: last-modified
---
```

This follows the standard Quarto convention where frontmatter acts as per-document configuration, overridable by project-level `_quarto.yml` settings.

## The Three-File Configuration Pattern

The typst-report template exemplifies a clean three-file configuration pattern common to well-structured Quarto projects:

| File | Scope | Consumer |
|---|---|---|
| `_brand.yml` | Visual identity | Quarto, Shiny, Typst (indirect) |
| `_variables.yml` | Patient/clinical data | Quarto shortcodes, Typst, R |
| `_quarto.yml` | Project structure | Quarto CLI |
| `config.yml` | Pipeline behavior | R/Python processing code |

This layering avoids mixing concerns: visual style, clinical content, rendering rules, and pipeline behavior each have a dedicated file.

## Common Pitfalls

| Mistake | Correct Form |
|---|---|
| Unquoted hex `#0066cc` | `"#0066cc"` |
| Tab indentation | Spaces only |
| Missing space after colon | `key: value` not `key:value` |
| Undefined references | Define palette before semantic colors |
| Missing protocol in URLs | `https://example.com` not `example.com` |
| Objects when strings suffice | `base: Inter` not `base:\n  family: Inter` |
| Variable name mismatch | Variable name in template must match `_variables.yml` exactly |
| Duplicate theme config per target | Use `_brand.yml` as single source of truth |

## Validation Rules

1. All fields optional — include only what's needed
2. Prefer hex colors with quotes (`"#447099"`)
3. Prefer simple string syntax over objects when possible
4. Sass naming: lowercase with hyphens for color and font names
5. Always include `https://` in URLs
6. Define colors and fonts before referencing them
7. Keep files concise — simpler is better
8. Ensure variable names are consistent across all consumers (Quarto, Typst, R)

## YAML as a Configuration Contract

When a YAML file is auto-discovered by tooling (e.g., `_brand.yml` by Shiny or Quarto, `_quarto.yml` by the Quarto CLI, `_variables.yml` by a report template), it forms an implicit contract:

- **File naming conventions** signal intent (`_brand.yml`, `_quarto.yml`, `_variables.yml`)
- **Location conventions** (project root, template subdirectories) determine scope
- **Schema validation** is applied by the consuming tool
- **Custom file names** are supported but require explicit path arguments

This pattern is consistent with [[concepts/documentation-as-code]] principles — configuration lives alongside source files, is version-controlled, and is human-editable.

## Auto-Discovery Patterns

Many tools implement filesystem-based auto-discovery for YAML config files:

- Shiny (R and Python) searches the app directory and parent directories for `_brand.yml`
- Quarto looks for `_brand.yml` and `_quarto.yml` at the project root (Quarto ≥ 1.4)
- Report templates link `_variables.yml` via `metadata-files` in `_quarto.yml`
- Custom file names are supported but require explicit path arguments

This pattern reduces boilerplate — placing a correctly named file in the right location is sufficient for the tool to apply the configuration.

## Relationship to Other Configuration Approaches

- **vs. JSON**: YAML is a superset of JSON; YAML adds comments, multi-line strings, and less punctuation noise
- **vs. TOML**: TOML favors explicit tables; YAML favors indentation hierarchy — both are common in modern toolchains
- **vs. environment variables**: YAML encodes structured, nested config; env vars are flat key-value pairs
- **vs. code-based config**: YAML is language-agnostic and readable by non-developers; code-based config (e.g., R or Python theme objects) offers more expressiveness but less portability
- **vs. CSS/SCSS variables only**: YAML-based brand specs (like `_brand.yml`) extend beyond CSS targets to reach non-web output formats like Typst PDFs

## Related Concepts

- [[concepts/brand-theming]] — brand.yml as a specific YAML configuration use case
- [[concepts/plain-text-documentation]] — YAML shares the plain-text, version-control-friendly philosophy
- [[concepts/documentation-as-code]] — treating configuration and docs as first-class source artifacts
- [[concepts/r-python-integration]] — brand.yml bridges R and Python toolchains through a shared config format
- [[concepts/brand-color-system]] — semantic color mapping patterns used in brand.yml
- [[concepts/brand-typography]] — font sourcing and typographic element configuration
- [[concepts/modular-report-architecture]] — template system that YAML variables power
- [[concepts/neuropsychological-report-variables]] — patient variable patterns in clinical report templates
- [[concepts/quarto]] — Quarto project and metadata configuration via YAML
- [[concepts/typst-typesetting]] — Typst integration via Quarto-mediated brand variable translation
- [[concepts/architecture-decision-records]] — ADR 0006 formalizes brand.yml adoption

See also: [[summaries/brand-yml-spec]], [[summaries/brand-yml-in-r]], [[summaries/0006-brand-yml-for-cross-platform-theming]], [[summaries/quarto]], [[summaries/001-choose-quarto-typst]], [[summaries/003-modular-template-structure]], [[summaries/template-system]], [[summaries/report-generation]], [[summaries/overview]], [[summaries/issue_branding_typst]], [[summaries/PROJECT_SETUP_COMPLETE]], [[summaries/README]], [[summaries/SETUP_SUMMARY]], [[summaries/0004-soul-style-profile-json]], [[summaries/brand-and-skills]], [[summaries/report-template]]

See also: [[summaries/style-extensions]]

See also: [[summaries/brand-yml-integration]]

See also: [[summaries/report-rendering-pipeline]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/LLM_INTEGRATION]]

See also: [[summaries/cognition.instructions]]

See also: [[summaries/copilot-instructions]]

See also: [[summaries/PERMANENT_SOLUTION_SUMMARY]]