# DocBridge
Markdown to Docs pipeline

## Build Queue

_Empty — nothing here yet._

## Build Planner

**Ideas**
- **Telegram and Slack copy buttons.** Reuse `markdownToWhatsApp` as the base, with a per-target mapping table (Telegram MarkdownV2 and Slack mrkdwn differ from WhatsApp). Not designed, not authorized.
- **Auto-split long documents for WhatsApp.** Split at headings so each piece fits under WhatsApp's length cap (about 65k chars), with one copy button per piece, like the existing chunk bar. Not designed, not authorized.
- **Live WhatsApp preview pane.** A third pane (or toggle) showing the WhatsApp text as it will paste, so it can be checked before copying. Not designed, not authorized.

**Bugs (found, not fixed)**
- **Footnote paragraph split.** `Text[^1]` followed by a `[^1]:` definition splits into two paragraphs, in the preview and in the WhatsApp output. Cause: the footnote extension's block `start()` cuts the paragraph at `[^`. Not authorized.
- **Inline HTML dropped from WhatsApp output.** A literal `<tag>` typed in the source is treated as HTML and stripped, not copied as text. Not authorized.
- **Startup error when a CDN fails.** If marked, KaTeX, highlight.js or Mermaid fails to load, `mermaid.initialize(...)` throws "mermaid is not defined" at startup. Not authorized.
- **WhatsApp import merges single line breaks.** In WhatsApp a single newline is a line break, but the app renders Markdown with `breaks: false`, so consecutive lines in converted text merge into one paragraph in the preview. Possible fix: have the converter add a hard break (two trailing spaces) where a plain text line is followed by another plain text line. Not authorized.

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

### 4. Robot-on-bridge icon art (Q3)
- [x] `icons/icon-source.jpg` added: Jon's original art, kept so the PNGs can be regenerated.
- [x] `icons/icon-192.png`, `icons/icon-512.png` replaced: full art, transparent rounded corners (3px inset to lose the JPG's white fringe).
- [x] `icons/apple-touch-icon-180.png` replaced: full-bleed square, white corners filled from neighboring pixels.
- [x] `icons/icon-maskable-512.png` replaced: art at 78% on a vertical sky-to-water gradient sampled from the art's top, middle and bottom rows, soft-edged.
- [x] `icons/favicon-32.png` replaced: robot head crop with rounded corners.
- [x] No change to `manifest.json` or `index.html` (same filenames).

Differences from the Q3 spec:
- The spec said to crop into the art to drop the JPG's white corners. The MD and WhatsApp badges sit within about 40px of the edges, so cropping would have clipped them. The "any" icons use a transparent rounded mask instead, and the Apple icon fills the corners instead of cropping.
- Maskable scale is 78% (spec said about 75%, adjust after viewing).

Verification: Generated with a scratch script (not committed). Viewed every size. On the maskable icon I drew the 80% safe circle and also simulated a full circular launcher mask: the robot and the MD, Docs and WhatsApp badges all stay visible. A first maskable backdrop (blurred copy of the art) looked muddy with a visible halo, so I replaced it with the plain gradient. Chrome's `Page.getAppManifest` and `Page.getInstallabilityErrors` on the new files: no errors.

Needs a real-device check (Pixel 7 only):
- [ ] Home screen icon is not clipped by the launcher mask and the badges are visible.
- [ ] Favicon is recognizable in Chrome's tab switcher.
- [ ] Clear the old icon first: remove the installed app and reinstall, since Android caches the installed icon.

### 5. Convert from WhatsApp button (Q4)
- [x] `index.html`: new `#wa-import-btn` "Convert from WhatsApp" in the footer toolbar, right after `#clear-btn` (same secondary style).
- [x] `index.html`: new `whatsAppToMarkdown(text)`. Line-based; fenced blocks and inline code are never touched. `*x*` to `**x**`, `_x_` to `*x*`, `~x~` to `~~x~~`, `_*x*_` / `*_x_*` to `***x***`, `☑` / `☐` to `- [x]` / `- [ ]`, `———` to `---`. Emphasis uses WhatsApp's own boundary rule (non-space inside, non-word outside), so `snake_case_words` and `2 * 3 * 4` are left alone. Monospace blocks and `text (url)` stay as they are. A standalone bold line stays a bold paragraph.
- [x] `index.html`: new `showUndoToast(msg, onUndo)` (builds its own toast content with an Undo button, hides after 8s) and the click handler for `#wa-import-btn` (reads the pane, converts, writes back, calls `render()`).
- [x] `index.html` CSS: new rules `#toast .toast-undo` and `#toast:has(.toast-undo)`. Nothing existing changed, and `showToast` is untouched.

Added beyond the spec (small, in the spirit of it):
- Undo refuses if the pane text was edited after converting ("Text changed since converting, can't undo"), so it can never wipe newer typing.
- If the text has no WhatsApp formatting to convert, toast "No WhatsApp formatting found" instead of offering a pointless Undo.
- While the Undo button shows, the toast is sized to its content and lifted to `bottom: 140px`. At the base position it wrapped onto two lines (it is `left: 50%`, so it only gets half the screen width) and sat over the toolbar.

Verification: Drove the real page in Chromium with a Pixel 7 emulation (412x915, touch, taps not clicks), libraries served locally. 13 checks pass: empty-pane toast; a 19-line mixed document converts to the exact expected Markdown (bold, italic, strike, bold+italic both orders, nested lists, tasks, quote, fenced block untouched, inline code untouched, rule, link text, accented letters); preview re-renders; Undo restores the original; Undo is refused after an edit; no-formatting toast; toast hides after about 8s and the hidden button ignores taps; a newer plain toast is not cut short by the old timer; Markdown to WhatsApp and back keeps formatting, lists and quotes; no horizontal overflow. 0 JS errors. Screenshots viewed: the footer wraps into two rows (Clear + Convert on top, the two copy buttons below) and the Undo toast sits on one line clear of the buttons.

Noted, not fixed:
- Single line breaks in WhatsApp text merge into one paragraph in the preview (added to the Planner).
- `:has()` is used for the toast rule. Fine on current Chrome (Pixel 7); an old WebView would show the narrow wrapped toast, which still works.

Needs a real-device check (Pixel 7 only):
- [ ] Paste a real WhatsApp message (bold, italic, strike, list, quote), tap Convert, and confirm the preview looks right.
- [ ] Tap Undo in the toast within a few seconds and confirm the original text comes back.
- [ ] Footer buttons wrap sensibly and the Undo toast is easy to tap.
