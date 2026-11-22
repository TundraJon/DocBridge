# DocBridge
Markdown to Docs pipeline

## Build Queue

### Q1. Copy for WhatsApp button — **Not yet authorized**

Add a "Copy for WhatsApp" button beside "Copy Rich HTML". It converts the Markdown source to WhatsApp markup and copies it as plain text. Nothing else changes.

- New button `#wa-copy-btn` in the footer button row, next to `#copy-btn`.
- New `markdownToWhatsApp(md)` in `index.html`. It walks the `marked.lexer()` tokens (marked is already loaded), not regex on the raw text, so code blocks and nesting stay correct.
- Copy via `navigator.clipboard.writeText()`, with `execCommand('copy')` fallback. Toast: "✓ WhatsApp text copied". Same empty-document check as `#copy-btn`.

Mapping:
- Bold `**x**` → `*x*`
- Italic `*x*` → `_x_`
- Strikethrough `~~x~~` → `~x~`
- Inline code → `` `x` ``
- Code block → ``` fenced ```
- Blockquote → `> `
- Bullets → `- `, numbered → `1. `
- Underline: none. WhatsApp has no underline.

Fallbacks (approved by Jon):
- Headings → `*TEXT*` on its own line
- Tables → monospace block, columns padded
- Mermaid and math → fenced block of their source
- Images → alt text

*(Added by Claude. Change before authorizing if wanted.)*
- Links → `text (url)`. WhatsApp has no link text.
- Horizontal rule → `———`.
- Task list → `☐` / `☑`.
- Bold+italic → `*_x_*`.
- Literal `*`, `_`, `~` in plain text are left as is. WhatsApp has no escape character, so they may trigger formatting. Known limit.
- Nested lists indent with spaces.

Risks: low. The change is additive and touches no existing function. The only shared spot is the footer button row layout, which may need a flex check at phone width.

Needs a real-device check after build:
- [ ] Paste into WhatsApp on Android and iOS: bold, italic, strike, code, quote and lists render.
- [ ] Long documents: WhatsApp's message length cap (about 65k chars) is not enforced by the button.

## Build Planner

- **Offline support (CDN dependencies).** `index.html` loads marked, KaTeX (CSS + JS), highlight.js (CSS + JS), and Mermaid from CDNs (jsdelivr, cdnjs). It does not work offline, and it breaks if a CDN is down or blocked. Possible fix: inline the libraries into the single file. Tradeoff: file size grows a lot (Mermaid is the heavy one). Not designed, not authorized.

## Build Log

### 1. Add starting file as index.html
- [x] `index.html` added at repo root, copied unchanged from `MC1.1.html` (1,019 lines).
- [x] `README.md`: offline/CDN issue added to Build Planner.

Verification: `cmp` shows `index.html` is byte-identical to `MC1.1.html`. Not run in a browser: the build sandbox blocks the CDNs, so marked, KaTeX, highlight.js and Mermaid never loaded.

Needs a real-device check:
- [ ] Opens with 0 JS errors with internet on.
- [ ] Markdown, table, math, Mermaid, code, and Copy Rich HTML paste into Google Docs and Word.
