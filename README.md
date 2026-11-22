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
