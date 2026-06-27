---
sources: [summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342.md, summaries/EMBEDDINGS_COMPLETE.md, summaries/One-sentence-per-line.md]
brief: A writing practice placing each sentence on its own line in plain-text markup documents.
---

# Semantic Linefeeds / One Sentence Per Line

Semantic Linefeeds (also called *Semantic Line Breaks* or the *One Sentence Per Line* practice) is a writing convention for plain-text markup formats in which **each sentence is placed on its own line**, rather than using fixed-column word-wrapping.

This concept is closely related to [[concepts/plain-text-documentation]], as it applies specifically to formats like Markdown, reStructuredText, and AsciiDoc where line breaks in source text do not affect rendered output.

## How It Works

In markup languages, a single line break in source text is usually rendered as a space (not a paragraph break), so wrapping sentences across lines does not affect the final output. This means authors can freely structure their source text line-by-line for their own editorial and version-control benefit.

Instead of writing:
```
This is the first sentence. This is the second sentence. This is the
third sentence which wraps at column 80.
```

You write:
```
This is the first sentence.
This is the second sentence.
This is the third sentence which wraps at column 80.
```

## Benefits

### For Version Control
- **Clean diffs**: A change to one sentence does not cause surrounding lines to reflow, keeping diffs minimal and readable.
- **Better code review**: On platforms like GitHub, reviewers can make a *"suggest changes"* comment targeting a single sentence.

### For Writing Quality
- Long sentences become visually obvious.
- Repetitive or redundant patterns are easier to spot.
- Sentence length variation is immediately visible.

### For Editing
- Sentences can be reordered by moving lines.
- Sentences can be commented out individually.
- Paragraphs can be split or merged by adding/removing blank lines.
- Bulk editor operations (e.g., converting sentences to list items) are easy to apply.

## Origins & References

The term *Semantic Linefeeds* was popularized by **Brandon Rhodes** in a 2012 blog post. The practice is also formally recommended in the AsciiDoc documentation as a best practice, and the [Semantic Line Breaks specification](https://sembr.org) provides a formal description.

## Sources

- [[summaries/One-sentence-per-line]] — Primary source document covering the rationale and advantages of this practice.


See also: [[summaries/EMBEDDINGS_COMPLETE]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]]