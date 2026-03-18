# TODOMAYBE

Ideas for future improvements. None are committed to.

---

## Done

- ~~**Deflate compression**~~ — `CompressionStream('deflate-raw')` + base64url; lz-string removed entirely
- ~~**CJK encoding**~~ ✗ reverted — percent-encoding makes URLs longer than base64url
- ~~**KaTeX math**~~ — `$inline$` and `$$block$$` via marked extension + KaTeX CDN
- ~~**File import**~~ — Import button loads .md/.txt into editor
- ~~**File export**~~ — Export button downloads editor content as .md
- ~~**Code execution**~~ — 24 languages via Wandbox API with run buttons
- ~~**Live mode**~~ — Obsidian-style block editing with fence-aware splitting
- ~~**Mermaid diagrams**~~ — `mermaid` fenced blocks render as SVG, theme-aware
- ~~**Tab capture**~~ — Tab inserts 2 spaces in both editor views
- ~~**List editing**~~ — Obsidian-style indent/dedent/continue/exit for lists
- ~~**Short URL**~~ — LinkShrink API, 1-year TTL, instant redirect
- ~~**Line numbers**~~ — VS Code-style gutter with word-wrap support
- ~~**Word count / reading time**~~ — Settings toggle, 180 wpm estimate
- ~~**Excalidraw whiteboards**~~ — Interactive drawing in ` ```excalidraw ``` ` blocks, direct rendering (no iframe), save-back to markdown source, fold by default, edit button in live mode
- ~~**Sort visualizer**~~ — `sort:LANG` fenced blocks animate sorting algorithms; `sort_print(arr, compare=, swap=)` injected automatically for Python, JS, TS, Ruby, C++, Rust, Lisp
- ~~**Docs modal**~~ — `?` button in header opens full feature reference (languages, sort_print implementations, limits, etc.)

---

## Remaining ideas

### Password-protected pastes
Use Web Crypto API (PBKDF2 → AES-GCM) to encrypt compressed bytes with a user-supplied password. Salt + IV + ciphertext go in the URL hash with a prefix to distinguish from plain pastes. On load, detect prefix → prompt for password → decrypt. ~40-50 lines of JS, no extra libraries.

### Table of contents
Auto-generate from headings. Collapsible sidebar or injected at top of preview.

### Export as standalone HTML
Serialize the rendered preview into a self-contained HTML file and trigger a browser download. Zero backend.

### Find / replace
Ctrl+H in the textarea. Simple floating panel.

### Keyboard shortcuts cheat sheet
Modal or tooltip showing all keybindings (Tab, Shift+Tab, Enter in lists, Escape in live mode, etc.).

### Edit history (localStorage)
Store last N URL hashes in localStorage. Navigate backward through edits. Lightweight undo across page reloads.

### Mobile formatting toolbar
Floating buttons for bold, italic, code, link — mobile keyboards lack the special keys needed for markdown shortcuts.
