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

### 3. Browser install icon + manifest (Q2)
- [x] `manifest.json` added: name "DocBridge Pro", short_name "DocBridge", `start_url` and `scope` "./", `display` standalone, theme `#1a73e8`, background `#f1f3f4`, three icons (192 any, 512 any, 512 maskable). All paths relative for the `/DocBridge/` subfolder.
- [x] `icons/` added: `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon-180.png`, `favicon-32.png`. Generated placeholder: white "DB" with a bridge arch on app blue. Swap for real art later (same filenames).
- [x] `index.html` `<head>`: added `<link rel="manifest">`, `<link rel="icon">`, `<link rel="apple-touch-icon">`, `<meta name="theme-color">`. Nothing else touched.
- [x] No service worker, no offline caching (Jon's call).

Verification: Served the repo under `/DocBridge/` on localhost in Chromium and ran Chrome's own checks (`Page.getAppManifest`, `Page.getInstallabilityErrors`): manifest parses with no errors, installability errors none. Head tags resolve to the right URLs. Re-ran the WhatsApp and Rich HTML copy checks with the libraries served locally: 0 page errors. Viewed the 512 and maskable icons: all artwork is inside the maskable safe zone.

Noted, not fixed:
- Existing: if a CDN fails to load, `mermaid.initialize(...)` throws "mermaid is not defined" at startup (offline item in the Planner).

Needs a real-device check:
- [ ] Chrome Android: "Install app" appears (no service worker, so older versions may only offer "Add to Home screen") and the home screen icon looks right.
- [ ] iOS Safari: Add to Home Screen uses the apple-touch icon.
- [ ] Desktop tab shows the favicon.
