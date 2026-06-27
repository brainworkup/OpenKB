---
sources: [summaries/SKILL.md]
brief: MDX authoring combines Markdown with JSX components to create structured, interactive documentation pages.
---

# MDX Authoring

MDX is a file format that combines standard Markdown with JSX (JavaScript XML) components, enabling authors to write prose-style documentation while embedding interactive or structured UI components inline. It is the core authoring format for platforms like Mintlify, where every documentation page is an `.mdx` file.

See [[summaries/SKILL]] for a detailed example of MDX authoring best practices in the context of Mintlify.

## How MDX Works

MDX files are processed and rendered into full documentation pages. The format supports:

- **Standard Markdown:** Headings, lists, bold/italic, code blocks, blockquotes, links.
- **YAML Frontmatter:** Metadata declared at the top of each file between `---` delimiters.
- **JSX Components:** Built-in or imported UI components used directly in content.

This combination allows documentation to be both human-readable in source form and richly structured when rendered — a key principle of [[concepts/documentation-as-code]].

## Frontmatter

Every MDX page begins with YAML frontmatter. At minimum, a `title` field is required. Common fields include:

```yaml
---
title: "Getting Started"
description: "A concise summary for SEO and navigation."
sidebarTitle: "Start Here"
icon: "rocket"
tag: "NEW"
mode: "wide"
keywords: ["setup", "install", "quickstart"]
---
```

Frontmatter drives navigation labels, SEO metadata, sidebar icons, and page layout — all without touching site-wide configuration files.

## Built-in Components

Platforms like Mintlify provide a library of MDX components that cover common documentation needs. Authors should prefer these over writing custom components:

| Component | Purpose |
|---|---|
| `<Steps>` | Sequential instructions |
| `<Tabs>` | User-selectable content variants |
| `<Accordion>` | Hide optional or advanced details |
| `<Card>` in `<Columns>` | Linked navigation cards |
| `<CodeGroup>` | Code in multiple languages |
| `<ParamField>` | API parameter documentation |
| `<ResponseField>` | API response field documentation |
| `<Note>`, `<Tip>`, `<Warning>` | Callouts by severity |

Built-in MDX components (provided by the platform) do not require explicit import statements. Custom JSX components stored in snippet files do require imports:

```mdx
import { MyComponent } from "/snippets/my-component.jsx"
```

## Reusable Snippets

When exact content appears on multiple pages, it can be extracted into a reusable snippet file. This keeps documentation DRY (Don't Repeat Yourself) and reduces maintenance overhead. Snippets are appropriate for:

- Shared warnings or prerequisites
- Complex components maintained in one place
- Content shared across teams or repositories

Snippets become counterproductive when per-page variations require complex conditional props.

## Code Blocks

All code blocks in MDX must include a language identifier for correct syntax highlighting:

````mdx
```python
print("Hello, world!")
```
````

Omitting the language tag is a common error that degrades the reading experience and may fail validation checks.

## Writing Standards for MDX

MDX authoring is as much about prose quality as technical structure. Key standards include:

- **Voice:** Second-person ("you"), active voice, direct language.
- **Headings:** Sentence case ("Getting started", not "Getting Started").
- **Context first:** Explain what something is before explaining how to use it.
- **No marketing language:** Avoid "powerful", "seamless", "robust", "cutting-edge".
- **No filler:** Avoid "it's important to note", "in order to", "simply".
- **Formatting:** Bold and italics only when they serve reader understanding; no decorative formatting.

These standards align with the broader philosophy of [[concepts/plain-text-documentation]] — keeping source files clean, readable, and maintainable by humans and tools alike.

## File and Link Conventions

- File names: kebab-case by default (`getting-started.mdx`, `api-reference.mdx`).
- Internal links: root-relative, no file extension (`/getting-started/quickstart`).
- Never use relative paths (`../`) for internal pages.
- Images: stored in `/images/`, referenced as `/images/example.png` with descriptive alt text.

## Relationship to Documentation-as-Code

MDX authoring is a concrete implementation of [[concepts/documentation-as-code]]: docs live alongside source code in Git repositories, are versioned with the same tools, and deploy automatically on push. The separation of content (`.mdx` files) from configuration (`docs.json`) mirrors the separation of concerns found in modern software architecture.

## Related Pages

- [[summaries/SKILL]] — Mintlify best practices using MDX authoring
- [[summaries/One-sentence-per-line]] — A related plain-text authoring discipline
- [[concepts/documentation-as-code]] — The broader philosophy MDX authoring embodies
- [[concepts/plain-text-documentation]] — Plain text as a durable documentation medium
