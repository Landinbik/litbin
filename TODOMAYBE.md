# TODOMAYBE

Ideas for future improvements. None are committed to.

---

## URL compression

The current stack uses LZ-string `compressToEncodedURIComponent` (base64url, 6 bits/char).

### ~~Better compression algorithm~~ ✓ done
~~Switch to the native `CompressionStream` API (deflate-raw).~~ Implemented: deflate-raw via `CompressionStream` replaces LZ-string. Old URLs still decode via LZ-string fallback.

### ~~Denser encoding~~ ✓ done
~~A hand-picked ~2048-character Unicode alphabet.~~ Implemented: 2048-char CJK alphabet (U+4E00–U+55FF), 11 bits/char vs base64's 6 — ~1.8× denser, combined with better deflate compression.

---

## Features

### Mermaid diagrams
Render fenced ` ```mermaid ``` ` blocks as flowcharts, sequence diagrams, etc. One CDN script. Very popular in markdown tools and a natural fit.

### KaTeX math
Render `$inline$` and `$$block$$` LaTeX via KaTeX (CDN). No backend needed.

### Table of contents
Auto-generate from headings. Could live as a collapsible sidebar or injected at the top of the preview.

### Export as standalone HTML
Serialize the rendered preview into a self-contained HTML file and trigger a browser download. Zero backend.

### Find / replace
Ctrl+H in the textarea. Could be a simple floating panel.

### Word count / reading time
Cheap to add to the header bar alongside the URL size indicator.

### Line numbers in editor
Overlay or a paired element synced to textarea scroll.

### Password-protected pastes
Encrypt the content with a user-supplied passphrase before LZ-compressing. Key lives only in the URL (after a second `#`) or is shared out-of-band.
