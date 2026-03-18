# TODOMAYBE

Ideas for future improvements. None are committed to.

---

## URL compression

### ~~Better compression algorithm~~ ✓ done
Implemented: `CompressionStream('deflate-raw')` replaces LZ-string. Old URLs still decode via LZ-string fallback.

### ~~Denser encoding (CJK alphabet)~~ ✗ reverted
CJK chars get percent-encoded on copy (%E4%B8%80 → 9 chars each), making URLs *longer* than base64. Switched to deflate-raw + base64url, which is the current codec.

---

## Done

- ~~**KaTeX math**~~ — `$inline$` and `$$block$$` via marked extension + KaTeX CDN
- ~~**File import**~~ — Import button loads .md/.txt into editor
- ~~**Code execution**~~ — 28+ languages via Wandbox API with run buttons
- ~~**Live mode**~~ — Obsidian-style block editing with fence-aware splitting

---

## Remaining ideas

### ~~Mermaid diagrams~~ ✓ done
Implemented via mermaid@11 CDN. Fenced ` ```mermaid ``` ` blocks render as SVG diagrams. Theme-aware (light/dark).

### Word count / reading time
Cheap to add to the header bar alongside the URL size indicator.

### Table of contents
Auto-generate from headings. Collapsible sidebar or injected at top of preview.

### Export as standalone HTML
Serialize the rendered preview into a self-contained HTML file and trigger a browser download. Zero backend.

### Find / replace
Ctrl+H in the textarea. Simple floating panel.

### Line numbers in editor
Overlay or a paired element synced to textarea scroll.

### Password-protected pastes
Encrypt with a user-supplied passphrase before compressing. Key shared out-of-band or appended after a second `#` in the URL.
