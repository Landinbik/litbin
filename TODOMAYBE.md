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
- ~~**Sort visualizer**~~ — `sort:LANG` fenced blocks animate sorting algorithms; `sort_print(arr, compare=, swap=)` injected automatically for Python, JS, TS, Ruby, C++, Rust, Lisp; left-to-right green sweep on completion
- ~~**Docs modal**~~ — `?` button in header opens full feature reference (languages, sort_print implementations, limits, etc.)
- ~~**PDF export**~~ — "PDF" button calls `window.print()`; `@media print` CSS hides everything except `#preview`; `@page` sets 1.5 cm margins
- ~~**Viewer mode**~~ — "View" button / `?view` URL param hides editor and all controls except Theme; preview goes fullscreen; "Edit" button exits; shareable URL preserves `?view`
- ~~**Slides mode**~~ — "Slides" button enters fullscreen presentation; split on `---`, navigate with arrow keys or buttons, Esc to exit
- ~~**Keyboard shortcuts cheat sheet**~~ — Modal showing all keybindings
- ~~**Find / replace**~~ — Ctrl+H floating panel in editor
- ~~**Command palette**~~ — Ctrl+K / Cmd+K fuzzy-searchable command menu
- ~~**Lit variable system**~~ — `data` blocks + `{varname}` injection + `random()` metafunction
- ~~**Video / media embedding**~~ — bare YouTube/Vimeo URLs auto-embed as iframes; `.mp4/.webm/.ogv` URLs embed as `<video>`; handled in marked paragraph renderer
- ~~**Live content-width resize handles**~~ — drag handles on left/right edges of `#liveContent` to adjust `max-width`; double-click resets to 760px; `position: fixed`, repositioned via `MutationObserver` on `livePane` style changes
- ~~**Sort viz cross-references**~~ — `#id` on sort blocks + `viz:id` reference blocks for side-by-side algorithm comparison; auto-grouped master controls; speed slider sync
- ~~**Embeddable fragments**~~ — `?embed` URL param hides all chrome; only rendered preview shown; "powered by litbin" badge links to `?view`; `postMessage` height sync for iframe auto-resize; `?embed&theme=dark|light` forces theme; "Embed" button opens modal with `<iframe>` + auto-resize script snippet; Excalidraw read-only in embed mode

---

## Remaining ideas

<!-- sorted: easiest → hardest; within same difficulty, highest benefit first -->

### PDF export customization — *Low*
Obsidian-style print options panel before calling `window.print()`: paper size (A4, Letter, Legal — via `@page { size: ... }`), content scaling (zoom % applied to `#preview` before print), and margins (none / narrow / normal / wide). Small modal or popover that sets CSS vars / inline styles, then calls `window.print()`, then resets them.

### Dynamic sort_print docs — *Low*
In `openDocs()` (lazy render), replace the hardcoded implementations section of `DOCS_MD` with dynamically built HTML: iterate `Object.entries(SORT_PREAMBLES)` (skipping aliases), wrap each in a `<pre><code>` block with hljs highlighting. Removes ~80 lines of duplicated content from the `DOCS_MD` string. Single source of truth: editing a preamble auto-updates the docs.

### Mobile formatting toolbar — *Low-Medium*
Floating buttons for bold, italic, code, link — mobile keyboards lack the special keys needed for markdown shortcuts.

### HTML export — *Low-Medium*
Serialize `preview.innerHTML` into a full `<!DOCTYPE html>` shell, inline the app's CSS into a `<style>` tag, and trigger a `<a download="document.html">` blob URL click. KaTeX and Mermaid output is already static HTML/SVG — works as-is. Excalidraw components won't be interactive in the export (show static placeholder).

### Edit history (localStorage) — *Low-Medium*
Store last N URL hashes in localStorage. Navigate backward through edits. Lightweight undo across page reloads.

### Table of contents — *Low-Medium*
Auto-generate from headings. Collapsible sidebar or injected at top of preview.

### PDF viewing mode (paper view) — *Low-Medium*
CSS-only print-preview simulation. Toggle via "Paper" button — applies `.paper-view` class to preview. Each "page" is a fixed-height div with box-shadow border and margin between pages; `break-after: page` and fixed page dimensions. No PDF.js needed — purely visual. (True PDF.js + html2canvas pipeline would be **High** difficulty for low gain.)

### Better tables (editor + live view) — *Medium*
Tab in a table row jumps to next cell (detect `|` delimiters in current line — needs priority over existing tab=2-spaces handler). Enter in last cell appends a new row with matching column count. Context menu "Insert table" opens a grid picker (like Google Docs) to choose NxM dimensions. Live mode: table blocks get a structured cell editor instead of raw textarea. Optional: auto-align column widths on save.

### Password-protected pastes — *Medium*
Use Web Crypto API (PBKDF2 → AES-GCM) to encrypt compressed bytes with a user-supplied password. Salt + IV + ciphertext go in the URL hash with a prefix to distinguish from plain pastes. On load, detect prefix → prompt for password → decrypt. ~40-50 lines of JS, no extra libraries.

### Image resizing with drag — *Medium*
After render, attach a resize handle (`<div class="img-resize-handle">`) to each `<img>` in preview. `mousedown` → track `mousemove` → update `img.style.width` live. `mouseup` → write back to markdown: find the matching `![alt](src)` in `editor.value`, replace with `<img src="..." width="Npx" alt="...">` (marked passes through HTML `<img>` tags). Handle touch events for mobile. Tricky part: reliable matching of image by src when writing back.

### Venn / set diagram blocks — *Medium*
New ` ```venn ``` ` fence type. Parse syntax: `A = {1,2,3}`, `B = {3,4,5}`, optional `show: A∩B` directives. Compute unions/intersections/differences in JS. Render as custom SVG (overlapping circles with labels) — no library needed for 2–3 sets. Hook into `render()` like mermaid. Theme-aware fill colors. Keep scope to 2–3 sets (4+ require Euler diagrams, much harder).

### Live collaboration rooms — *High*
P2P editing via WebRTC using PeerJS (free public signaling, no backend). URL structure: `?room=ROOMID#content`. Room creator generates random ID, opens PeerJS host, broadcasts editor diffs to all peers. Joiner connects, receives full doc on join, applies patches. Conflict resolution: last-write-wins with logical timestamp (simple) or OT/CRDT (complex). PeerJS public signaling is free but not guaranteed uptime.

---

## Everyday user ideas

### Image embedding (drag/drop + paste) — *Medium*
Drag/drop or paste an image → auto-upload to imgur or similar free host → embed `![](url)` inline. Currently no image story besides manually typing URLs. Needs: paste event handler, imgur API (anonymous upload, no auth required for <50/day), progress indicator.

### Table editor — *Medium*
Click-to-edit grid overlay on rendered markdown tables. Toolbar button to insert NxM table. Tab jumps between cells. See also "Better tables" above for editor-side improvements.

### Paste formatting preservation — *Low-Medium*
Intercept paste events containing `text/html`, convert common HTML structures (bold, italic, links, lists, headings) to markdown equivalents via simple regex/DOM walk. Drop unsupported formatting gracefully. No library needed for basic cases.

### Auto-save / recent documents — *Low-Medium*
`localStorage` snapshots on every URL sync (debounced). "Recent" menu lists last N documents with title (first `# heading` or first line) + timestamp. Click to restore. Cap storage at ~5 MB. Separate from edit history — this is document-level, not edit-level.

---

## Power user ideas

### Python notebooks (Pyodide) — *Medium-High*
` ```python:notebook ``` ` or ` ```pybook ``` ` fence blocks behave like Jupyter cells: run Python client-side via Pyodide (WASM), output renders inline (text, tables, matplotlib plots as inline SVG/PNG). Cells share a persistent Python session. `data` block variables injectable. No server needed.

### More viz block types — *Medium-High*
Generalize the sort visualizer pattern. `viz:plot` for charting arrays/data (line, bar, scatter — lightweight Canvas or SVG renderer). `viz:graph` for graph algorithm animation (BFS/DFS/Dijkstra with node/edge highlighting). `viz:table` for data structure state step-by-step. Each type has its own output format and renderer.

### WASM code execution — *High*
Replace Wandbox with client-side execution via WASM runtimes: Pyodide (Python), QuickJS (JS), wasm-compiled C/Rust toolchains. No rate limits, works offline, faster feedback. Fallback to Wandbox for languages without WASM runtimes. Big download on first use (~10 MB for Pyodide).

### Cell-style reactive execution — *Medium*
Jupyter-style dependency graph: `data` block outputs feed downstream code blocks, re-running upstream re-triggers downstream automatically. The lit variable system is 80% of the way there — needs execution ordering, dirty tracking, and output caching per cell.

### Custom themes / CSS injection — *Low*
` ```style ``` ` fence block injects scoped CSS into the document. Educators and speakers can brand their slides/documents. Sanitize to prevent breaking app UI (scope to `.md-body` or preview container only).

### Slide speaker notes + timer — *Low-Medium*
`<!-- notes: ... -->` HTML comments in slide content become speaker notes. "Present" opens two windows: audience view (current slide) and speaker view (current slide + notes + next slide + elapsed timer). Uses `window.open()` + `BroadcastChannel` for sync.

### Document composition / includes — *Medium*
`!include(url#hash)` directive pulls in content from another litbin URL. Fetched, decompressed, and spliced inline at render time. Enables reusable components (shared code blocks, common preambles). Cache fetched includes in `sessionStorage`.

### Offline PWA — *Low-Medium*
Service worker + `manifest.json` for installable offline app. Cache `index.html` + CDN libs on first visit. Power users giving talks or teaching with spotty wifi. Small scope: just cache everything on install, serve from cache when offline.

### Vim/Emacs keybindings — *Medium*
Toggle in settings. Vim: modal editing (normal/insert/visual modes), hjkl navigation, common commands (dd, yy, p, ciw, etc.). Emacs: C-a/C-e/C-k/C-y/M-f/M-b. Could use CodeMirror's vim/emacs extensions if we ever swap the textarea for CodeMirror, otherwise hand-roll a subset.
