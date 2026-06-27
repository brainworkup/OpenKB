---
sources: [summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md]
brief: Browser-native clipboard APIs for copying rich text, HTML, and plain content from web apps.
---

# Clipboard API Patterns for Web Apps

Modern browsers expose two clipboard APIs with different capabilities and use cases. Choosing the right one — and combining them correctly — determines whether pasted content arrives as plain text or fully formatted rich content.

## Two APIs, Two Use Cases

### `navigator.clipboard.writeText()`
The simpler API. Writes a plain string to the clipboard.

```js
navigator.clipboard.writeText(markdownString).then(() => {
  // success
});
```

**Best for:** Markdown editors, code snippets, any context where the receiver renders the text itself (Notion, Obsidian, terminal).

### `navigator.clipboard.write()` with `ClipboardItem`
The richer API. Accepts one or more MIME-typed blobs, allowing the paste target to choose the best representation.

```js
const htmlBlob = new Blob([htmlString], { type: 'text/html' });
const textBlob = new Blob([htmlString], { type: 'text/plain' });
navigator.clipboard.write([
  new ClipboardItem({
    'text/html': htmlBlob,
    'text/plain': textBlob
  })
]);
```

**Best for:** Pasting into Word, Google Docs, Outlook, or email clients where formatting (headings, bold, numbered lists) must be preserved.

Including both `text/html` and `text/plain` in the same `ClipboardItem` provides a graceful fallback: rich targets use the HTML, plain-text targets use the fallback.

## Architecture Pattern: Hidden Source Elements

A common pattern in server-rendered or framework apps (like [[concepts/shiny-for-python]]):

1. **Hidden `<textarea>`** holds the raw markdown/text source:
   ```html
   <textarea id="generated-markdown-text" style="display:none">...</textarea>
   ```
2. **Visible rendered container** (e.g., `card_body`) has an `id` for innerHTML access:
   ```html
   <div id="generated-html-content"><!-- rendered HTML --></div>
   ```
3. **JavaScript functions** read from each element type:
   ```js
   function copyAsMarkdown() {
     var md = document.getElementById('generated-markdown-text').value;
     navigator.clipboard.writeText(md);
   }
   function copyAsHtml() {
     var html = document.getElementById('generated-html-content').innerHTML;
     // ... ClipboardItem write
   }
   ```

This is particularly useful when the framework controls rendering and JavaScript cannot directly access the application state.

## Injecting JavaScript in Framework Apps

In frameworks that don't expose raw HTML, client-side JS must be injected. In Shiny for Python:

```python
ui.tags.script("""
  function copyAsMarkdown() { ... }
  function copyAsHtml() { ... }
""")
```

Buttons call these functions via `onclick` attributes:

```python
ui.tags.button(
    "Copy Markdown",
    onclick="copyAsMarkdown()",
    class_="btn btn-outline-secondary btn-sm"
)
```

## Visual Feedback Pattern

Preventing user confusion after a copy requires brief feedback. A clean approach using Bootstrap classes:

```js
var btn = document.getElementById('copy-md-btn');
btn.textContent = 'Copied!';
btn.classList.replace('btn-outline-secondary', 'btn-success');
setTimeout(function() {
  btn.textContent = 'Copy Markdown';
  btn.classList.replace('btn-success', 'btn-outline-secondary');
}, 2000);
```

`classList.replace()` atomically swaps one class for another, avoiding flickering that can occur with `classList.remove()` + `classList.add()`.

## Security Constraints

- Clipboard access requires a **secure context** (HTTPS or localhost)
- `navigator.clipboard.write()` with `ClipboardItem` requires the **Clipboard Write** permission, which browsers grant automatically for user-initiated events (onclick)
- Reading from the clipboard (`readText`, `read`) requires explicit user permission grant — write-only operations do not

## Two-Format Copy Buttons: UX Rationale

Providing both Copy Markdown and Copy HTML buttons serves distinct workflows:

| Format | Target Applications |
|---|---|
| Markdown | Notion, Obsidian, GitHub, text editors, terminals |
| HTML | Microsoft Word, Google Docs, Outlook, Gmail |

See [[summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co]] for a concrete implementation in a clinical recommendation generation app.

## Related Concepts

- [[concepts/shiny-for-python]] — framework where this pattern was implemented via `ui.tags.script()`
- [[concepts/narrative-report-generation]] — the content type being copied (clinical recommendations)
- [[concepts/clinical-nlp-pipelines]] — upstream pipeline producing the markdown output
- [[concepts/retrieval-augmented-generation]] — the RAG system generating the content that gets copied
