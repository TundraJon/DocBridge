# DocBridge
Markdown to Docs pipeline

## Build Queue

_Empty — nothing here yet._

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

### 2. Copy for WhatsApp button (Q1)
- [x] `index.html`: new `#wa-copy-btn` "Copy for WhatsApp" in the footer toolbar, next to `#copy-btn`.
- [x] `index.html`: new `markdownToWhatsApp(md)`. Walks `marked.lexer()` tokens. Bold `*x*`, italic `_x_`, strike `~x~`, inline code, fenced code, `> ` quotes, `- ` / `1.` lists, `☐` / `☑` tasks, `———` rule, headings as `*TEXT*`, tables as padded monospace blocks, Mermaid and `$$` math as fenced source, images as alt text, links as `text (url)`.
- [x] `index.html`: new `copyWhatsAppToClipboard()` (writes text/plain via `navigator.clipboard.writeText`) and `fallbackCopyText()` (textarea + `execCommand('copy')`). Same empty-document check and toast as `#copy-btn`.
- [x] No existing function touched.

Verification: Drove the real page in Chromium at 390px, with the CDN libraries served from local npm copies (marked 18.2.1, KaTeX, highlight.js, Mermaid). Empty doc shows "Nothing to copy yet". A mixed document copied the expected WhatsApp text (checked in the clipboard). The `navigator.clipboard`-missing fallback path ran and toasted. "Copy Rich HTML" still works. 0 JS errors; the only failed requests were KaTeX font files my test router didn't map. Screenshot shows three buttons in one row, no overflow.

Noted, not fixed:
- Bold+italic `***x***` comes out `_*x*_`, not `*_x_*` as the spec said. WhatsApp renders both the same.
- Inline HTML tags like `<tag>` in the source are dropped, not copied as text.
- Inline math `$x$` stays as plain `$x$` text; only display math `$$...$$` is fenced.
- Existing app quirk: `Text[^1]` followed by a `[^1]:` footnote splits into two paragraphs (the footnote extension's block `start()` cuts the paragraph). The WhatsApp output inherits this.

Needs a real-device check:
- [ ] Paste into WhatsApp on Android and iOS: bold, italic, strike, code, quote and lists render.
- [ ] Phone-width layout of the three footer buttons on a real device.
- [ ] Long documents: the button does not enforce WhatsApp's message length cap (about 65k chars).
