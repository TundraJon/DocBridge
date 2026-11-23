# DocBridge
Markdown to Docs pipeline

## Build Queue

### Q3. Replace placeholder icons with Jon's robot-on-bridge art — **Not yet authorized**

Swap the five placeholder PNGs in `icons/` for versions made from Jon's art (robot holding a document on a stone bridge, with MD, Docs and WhatsApp badges). Same filenames, so `manifest.json` and the `<head>` tags need no change.

- Keep the original as `icons/icon-source.jpg` so the PNGs can be regenerated later.
- `icon-192.png`, `icon-512.png` (any): crop slightly into the art to drop the JPG's white corners, then cut a rounded shape on a transparent background.
- `icon-maskable-512.png`: full-bleed square. Art scaled down so the MD, Docs and WhatsApp badges stay inside Android's circular safe zone, centered on a sky-to-water gradient sampled from the art's top and bottom edges, edges feathered so there is no hard seam.
- `apple-touch-icon-180.png`: full-bleed square crop, no transparency.
- `favicon-32.png`: crop to the robot's head and shoulders (the full scene is unreadable at 32px).
- Generation is a scratch script (not committed). Each PNG is viewed before commit.

*(Added by Claude. Change before authorizing if wanted.)*
- Maskable scale about 75%, adjusted after viewing so the badges are clear of the edge.
- Source art kept in the repo (adds the JPG's file size).

Risks: low, image files only. The maskable seam is the one thing that could look off.

Needs a real-device check after build (Jon uses a Google Pixel 7 only, no computer):
- [ ] Pixel 7 home screen: the icon is not clipped by the launcher's circle/squircle mask, and the MD, Docs and WhatsApp badges are visible.
- [ ] Chrome on the Pixel: the favicon is recognizable in the tab switcher.

### Q4. "Convert from WhatsApp" button — **Not yet authorized**

Add a "Convert from WhatsApp" button to the footer toolbar. The user pastes WhatsApp-formatted text into the Markdown pane, clicks it, and the pane's text is rewritten as Markdown in place. Manual button, no auto-detect (Jon's call). Nothing else changes.

- New button `#wa-import-btn` in the footer toolbar, next to `#clear-btn`.
- New `whatsAppToMarkdown(text)` in `index.html`. Line-based, not blind regex: fenced blocks and inline code are lifted out first and restored untouched.
- Mapping: `*x*` to `**x**`, `_x_` to `*x*`, `~x~` to `~~x~~`, `_*x*_` / `*_x_*` to `***x***`, `☑` / `☐` to `- [x]` / `- [ ]`, `———` to `---`. `> ` quotes, `- ` bullets, `1.` numbers and code already match Markdown.
- Standalone bold line: stays a bold paragraph, not a heading (Jon's call).
- Left as is, because it can't be told apart: monospace blocks (could be tables or code) and `text (url)` (could be a link or plain text).
- Boundary-aware, like WhatsApp itself: a marker needs non-space text inside it and a word boundary outside, so `snake_case_words` and `2 * 3 * 4` are not converted.
- Empty pane: toast "Nothing to convert yet". Success: toast "✓ Converted from WhatsApp", then the preview re-renders.

*(Added by Claude. Change before authorizing if wanted.)*
- Undo is in-app, because Jon has no keyboard shortcuts (Pixel 7 only). Before converting, the original text is kept in a variable. The success toast gets an **Undo** button, visible about 8 seconds. Tapping it puts the original text back and re-renders. No `execCommand` trick.
- With four buttons the toolbar wraps to two rows at Pixel 7 width (it already has `flex-wrap`). Not changing the layout.

Risks: low. Additive. The in-place rewrite is destructive, which is why the Undo toast exists. The toast currently has no buttons, so it needs a small tweak to hold one (new element inside `#toast`, same fade timing but longer for this case).

Needs a real-device check after build:
- [ ] On the Pixel 7: paste a real WhatsApp message (bold, italic, strike, list, quote), tap Convert, and confirm the preview looks right.
- [ ] Tap Undo in the toast within a few seconds and confirm the original text comes back.
- [ ] Footer buttons wrap sensibly at Pixel 7 width.

## Build Planner

**Ideas**
- **Telegram and Slack copy buttons.** Reuse `markdownToWhatsApp` as the base, with a per-target mapping table (Telegram MarkdownV2 and Slack mrkdwn differ from WhatsApp). Not designed, not authorized.
- **Auto-split long documents for WhatsApp.** Split at headings so each piece fits under WhatsApp's length cap (about 65k chars), with one copy button per piece, like the existing chunk bar. Not designed, not authorized.
- **Live WhatsApp preview pane.** A third pane (or toggle) showing the WhatsApp text as it will paste, so it can be checked before copying. Not designed, not authorized.

**Bugs (found, not fixed)**
- **Footnote paragraph split.** `Text[^1]` followed by a `[^1]:` definition splits into two paragraphs, in the preview and in the WhatsApp output. Cause: the footnote extension's block `start()` cuts the paragraph at `[^`. Not authorized.
- **Inline HTML dropped from WhatsApp output.** A literal `<tag>` typed in the source is treated as HTML and stripped, not copied as text. Not authorized.
- **Startup error when a CDN fails.** If marked, KaTeX, highlight.js or Mermaid fails to load, `mermaid.initialize(...)` throws "mermaid is not defined" at startup. Not authorized.

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
