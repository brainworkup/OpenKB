---
doc_type: short
full_text: sources/One-sentence-per-line.md
---

# One Sentence Per Line

## Overview
A writing practice recommendation for [[concepts/plain-text-documentation]] formats such as Markdown, reStructuredText, and AsciiDoc: place every sentence on its own line rather than using fixed-column word-wrapping.

## Core Recommendation
- Write one sentence per line in markup-based documentation.
- Avoid fixed-column wrapping (e.g., wrapping at 80 characters).

## Advantages

### Version Control & Diffs
- Prevents **reflows**: a change early in a paragraph won't cause surrounding lines to reposition, keeping code diffs clean and minimal.
- Improves GitHub's *"suggest changes"* workflow — reviewers can target individual sentences.

### Editing & Manipulation
- Easier to swap, separate, or join sentences.
- Sentences can be commented out or annotated individually.
- Bulk editor actions (e.g., converting sentences to list items by prepending `-`) are straightforward.

### Writing Quality
- Helps spot sentences that are too long.
- Highlights sentences that vary widely in length.
- Reveals redundant or repetitive patterns in writing.

## Related Concept
This practice is also known as [[concepts/semantic-linefeeds]] or **Semantic Line Breaks** (see [sembr.org](https://sembr.org)), a term coined/popularized by Brandon Rhodes (2012). It is also an officially recommended practice in AsciiDoc documentation.

## References
- Brandon Rhodes, [Semantic Linefeeds](https://rhodesmill.org/brandon/2012/one-sentence-per-line/), 2012
- [AsciiDoc Recommended Practices: One Sentence Per Line](https://asciidoctor.org/docs/asciidoc-recommended-practices/#one-sentence-per-line)
- [Semantic Line Breaks](https://sembr.org)