# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**litbin** — a single-file (`index.html`) literate programming pastebin. Zero backend. The entire document is stored in the URL hash as deflate-compressed, base64url-encoded text — sharing a URL shares the content.

## URL storage model

```
https://example.com/#<deflate-raw compressed, base64url-encoded markdown>
```

- The hash fragment is never sent to the server, so there is no server-side involvement
- Practical limit: ~200 KB markdown fits comfortably; warning appears at 500 KB encoded chars, error at 1.5 MB
- **Codec**: `CompressionStream('deflate-raw')` → base64url encoding (A-Za-z0-9-_, URL-safe ASCII, never percent-encoded)
- **Why not denser Unicode encoding**: non-ASCII chars in URL fragments get percent-encoded on copy (each char → 9 ASCII chars), making them worse than base64url in practice

## Stack (all CDN, no build step)

| Library | Purpose |
|---|---|
| `marked@9` | Markdown → HTML |
| `highlight.js@11` (common bundle) | Code block syntax highlighting |
| `KaTeX@0.16` | LaTeX math rendering |
| `mermaid@11` | Diagram rendering |
| `react@18` + `react-dom@18` (via import map + esm.sh) | Excalidraw dependency |
| `@excalidraw/excalidraw@0.18` (via esm.sh) | Interactive whiteboard |

No Node.js, no npm, no build. Open `index.html` directly in a browser or serve with any static host.

## Architecture

Everything lives in `index.html` (~4,200 lines):

1. **CSS** — custom properties for light/dark theming, responsive split-pane layout, sort visualizer, docs modal
2. **HTML** — header bar (buttons, settings, URL size), docs modal overlay, mobile tab bar, split `main` with editor pane + divider + preview pane, live mode pane
3. **Import map** in `<head>` — maps `react`, `react-dom`, `react-dom/client`, `react/jsx-runtime` to esm.sh CDN
4. **JS** (one IIFE at bottom):
   - On load: decode hash → populate `<textarea>` → fold excalidraw blocks → render preview
   - On `input`: immediate render + debounced URL sync (400ms)
   - Marked renderer override: pipes code blocks through highlight.js
   - Resizable divider: mousedown/mousemove/mouseup on `#divider`
   - Mobile: tab buttons toggle `.mobile-active` on the two panes

## Key behaviors

- **Theme**: reads `prefers-color-scheme` on load; toggle button swaps CSS custom props + hljs stylesheet + mermaid theme
- **Share button**: `navigator.clipboard.writeText(location.href)`
- **Short URL**: LinkShrink API (`POST linkshrink.dev/api/v1/shorten`), 1-year TTL
- **URL size indicator**: color-coded (normal / yellow >500 KB / red >1.5 MB) in header
- **Word count**: optional (settings toggle), shows words + reading time at 180 wpm
- **Default content**: shown only when the hash is empty (new visit); `_showingDefault` flag prevents this content from being encoded into the URL hash — cleared on first edit or mode switch
- **Docs modal**: `?` button opens full feature reference rendered from `DOCS_MD` markdown constant; includes keyboard shortcuts reference

## Editor features

- **Line numbers**: VS Code-style gutter with word-wrap support, synced via hidden mirror div
- **List editing**: Obsidian-style — Enter continues lists, Tab/Shift+Tab indent/dedent, Enter on empty item exits list
- **Tab capture**: Tab inserts 2 spaces in both editor and live mode textareas
- **Excalidraw fold/unfold**: Collapses `excalidraw` blocks to `◆ N elements (folded)` placeholder; `getEditorContent()` always returns real (unfolded) content for URL encoding; **folds by default on load**
- **Command palette**: Ctrl+K / Cmd+K — fuzzy-searchable command menu
- **Find & Replace**: Ctrl+H — floating panel in editor
- **Keyboard shortcuts cheat sheet**: modal showing all keybindings

## Live mode

Obsidian-style block editing. Markdown is segmented into blocks (separated by blank lines, respecting code fences).

- Click a rendered block to edit it as a textarea
- Double-newline in a textarea splits the block
- Backspace at start merges with previous block
- Arrow keys navigate between blocks
- Escape blurs the active textarea
- Event delegation on `liveContent` for mousedown/click (avoids per-element listener leak)
- Excalidraw blocks get an "Edit source" button overlay
- Drag-reorderable blocks
- Side-by-side code + sort visualizer layout
- Content-width resize handles: fixed handles on left/right edges of `#liveContent`, drag to adjust `max-width`, double-click resets to 760px; positioned via `getBoundingClientRect()` and resynced via MutationObserver on `livePane` style changes
- `syncFromLive()` uses its own `liveDebounce` timer (separate from editor's `urlDebounce`)

## Embed mode

Embeds rendered litbin content into other websites via `<iframe>`.

- `?embed` URL param → `body.embed` class → hides all chrome (header, editor, tabs, overlays)
- Only the rendered preview is shown; `body.embed` CSS rules suppress everything else via `!important`
- "powered by litbin" badge (`.embed-badge`) fixed bottom-right, links to `?view` version
- `postMessage` sends `{ type: 'litbin-resize', height }` to parent frame; `ResizeObserver` on `#preview` triggers re-sends
- `?embed&theme=dark` or `?embed&theme=light` forces theme regardless of system preference
- `setHash()` preserves `location.search` (not just `?embed`) so `&theme=` survives hash updates
- Excalidraw blocks render as `viewModeEnabled: true` — no `onChange` save-back in embed mode
- "Embed" button in header opens modal (`#embedModal`) with `<iframe>` + auto-resize `<script>` snippet
- Generated snippet uses `id="litbin-embed"` on the iframe; the inline script listens for `litbin-resize` postMessage
- `sandbox="allow-scripts allow-same-origin allow-popups"` on generated iframes
- Theme select in modal regenerates snippet in real-time; copy button uses `navigator.clipboard`
- Command palette entry: "Embed / get iframe code"
- "Embed" button hidden in viewer mode (`body.viewer #embedBtn { display: none }`)

## Presentation mode (Slides)

Fullscreen slide deck view for presenting. Slides are split by `---` delimiters.

- "Slides" button in editor toggles fullscreen presentation mode
- URL parameter `?slides` opens slides directly on load (shareable link)
- Hides all UI except slides overlay when using `?slides` param
- Split content on `\n---\n` → array of slide markdown strings
- Each slide rendered through marked + KaTeX + Mermaid pipeline
- Navigation: arrow keys (← →), Prev/Next buttons, side arrows, slide counter
- Escape or × button exits back to editor
- Example: `https://example.com/?slides#encoded_content`

## Rich content rendering

### KaTeX math
`$inline$` and `$$block$$` via marked extension. Renders with `throwOnError: false`.

### Mermaid diagrams
Fenced ` ```mermaid ``` ` blocks render as SVG. Theme-aware (light → default, dark → dark). Each diagram gets a unique ID (`mermaid-N`).

### Video / media embedding

Marked's paragraph renderer intercepts bare URLs and auto-embeds media:

- **YouTube**: `youtube.com/watch?v=ID` or `youtu.be/ID` → responsive `<iframe>` embed
- **Vimeo**: `vimeo.com/ID` → responsive `<iframe>` embed
- **Direct video**: `.mp4`, `.webm`, `.ogv`, `.ogg` URLs → HTML5 `<video>` element
- Aspect ratio maintained via 56.25% padding trick; styled as `.video-embed`
- `loading="lazy"` on iframes

### Excalidraw whiteboards
Fenced ` ```excalidraw ``` ` blocks render as interactive whiteboards.

- **Direct rendering** (no iframe) — React + Excalidraw mounted directly into parent page DOM via `import()` and import maps
- **Why no iframe**: blob URL iframes have `null` origin, blocking all ESM module fetches from CDNs
- **esm.sh config**: `?external=react,react-dom` — do NOT add `react/jsx-runtime` to externals (causes 404)
- **Save-back**: `onChange` debounced 600ms → writes compact JSON back into `editor.value` via regex replacement; early-returns if data unchanged (prevents URL stamp on load)
- **Cache**: `_excalidrawWrapCache` Map keyed by ordinal ID — avoids re-mounting on every render
- **Fold-aware**: save-back updates `_foldedData` when blocks are folded instead of touching editor text; `renderExcalidraw` falls back to `_foldedData` when JSON parse fails (folded state)

## Code execution

Fenced code blocks with a supported language tag get a run button. Execution goes through the [Wandbox](https://wandbox.org/) public API (`POST https://wandbox.org/api/compile.json`). No backend required.

Each language is declared in the `EXEC_LANGS` map:

```js
{
  compiler: string,       // Wandbox compiler ID
  options: string,        // Wandbox predefined option switches
  rawOptions?: string,    // passed as compiler-option-raw (arbitrary compiler flags)
  transform?: fn(code)    // pre-process source before sending (e.g. Java class modifier strip)
}
```

**Known quirks:**
- **Nim** — uses `rawOptions: '--hints:off'` to suppress verbose build chatter
- **Java** — `transform` strips `public` from `class` declarations (Wandbox filename requirement)
- **Common Lisp** — source must be ASCII-only (CLISP 2.49 limitation)
- **Swift, OCaml, Erlang, Elixir** — disabled (commented out); broken server-side on Wandbox

### Sort visualizer

`sort:LANG` fence tag → **▶ Visualize** button. Output parsed by `parseSortOutput()` into frames; animated by `createSortVisualizer()`.

- `(N)` in output = compare (yellow bar), `[N]` = swap/placement (red bar), plain = default
- Final frame with no annotations triggers left-to-right green sweep (~500ms total, staggered per bar)
- `SORT_PREAMBLES` map injects a `sort_print(arr, compare=, swap=)` helper before user code for: Python, JS, TS, Ruby, C++, Rust, Lisp
- hljs highlight loop skips `language-mermaid` and `language-excalidraw` to avoid unknown-language warnings
- Homepage has examples: insertion sort (Python), merge sort (Python), quicksort with 1/3 pivot (Lisp, Rust)
- Run all / Visualize all buttons in preview, slides, and live view
- Master playback control that syncs all sort visualizers simultaneously

### Lit variable system

`data` fence blocks define variables (`name = value`, JSON-parsed). `{varname}` in a code block's info string injects the variable with language-specific serialization. `random(min, max, count)` metafunction generates arrays at runtime. Injection order: var injections → sort preamble → user code.

### Syntax highlighting

The `highlight.min.js` common bundle covers most languages. Languages missing from the bundle are loaded as standalone modules from cdnjs:

| Module | Languages highlighted |
|---|---|
| `lisp.min.js` | `lisp`, `common-lisp` |
| `haskell.min.js` | `haskell`, `hs` |
| `ocaml.min.js` | `ocaml` |
| `julia.min.js` | `julia`, `jl` |
| `nim.min.js` | `nim` |
| `d.min.js` | `d` |

**No hljs grammar available** (as of hljs 11.9.0): Zig, Pascal.

## Development

No build step. Edit `index.html` and refresh the browser. To serve locally:

```bash
python3 -m http.server 8080
# or
npx serve -p 8080 .
```

## Deployment

GitHub Pages at https://landinbik.github.io/litbin/ — currently serving from `feature/sort-visualizer` branch (not yet merged to `main`). Switch branch via `gh api repos/Landinbik/litbin/pages -X PUT -f build_type=legacy -f source[branch]=BRANCH -f source[path]=/`.
