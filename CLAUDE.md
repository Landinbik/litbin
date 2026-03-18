# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file (`index.html`) markdown pastebin. Zero backend. The entire document is stored in the URL hash as LZ-compressed, URI-encoded text — sharing a URL shares the content.

## URL storage model

```
https://example.com/#<deflate-raw compressed, CJK-encoded markdown>
```

- The hash fragment is never sent to the server, so there is no server-side involvement
- Practical limit: ~200 KB markdown fits comfortably; warning appears at 500 KB encoded chars, error at 1.5 MB
- **Codec**: `CompressionStream('deflate-raw')` → 2048-char CJK alphabet (U+4E00–U+55FF), 11 bits/char
- **Legacy**: URLs encoded with [lz-string](https://github.com/pieroxy/lz-string) (base64url, 6 bits/char) still decode correctly; lz-string is kept on the CDN for this fallback
- **Detection**: if the first hash character is `< U+4E00` → legacy LZ-string; `≥ U+4E00` → new format

## Stack (all CDN, no build step)

| Library | Purpose |
|---|---|
| `marked@9` | Markdown → HTML |
| `lz-string@1` | URL compression |
| `highlight.js@11` (common bundle) | Code block syntax highlighting |

No Node.js, no npm, no build. Open `index.html` directly in a browser or serve with any static host.

## Architecture

Everything lives in `index.html`:

1. **CSS** — custom properties for light/dark theming, responsive split-pane layout
2. **HTML** — header bar, mobile tab bar, split `main` with editor pane + divider + preview pane
3. **JS** (one IIFE at bottom):
   - On load: decode hash → populate `<textarea>`
   - On `input`: debounce 250ms → encode → `history.replaceState` → re-render preview
   - Marked renderer override: pipes code blocks through highlight.js
   - Resizable divider: `mousedown/mousemove/mouseup` on `#divider`
   - Mobile: tab buttons toggle `.mobile-active` on the two panes

## Key behaviors

- **Theme**: reads `prefers-color-scheme` on load; toggle button swaps CSS custom props + hljs stylesheet
- **Share button**: `navigator.clipboard.writeText(location.href)`
- **URL size indicator**: color-coded (normal / yellow warn / red over) in the header
- **Default content**: shown only when the hash is empty (new visit with no shared URL)

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
- **Nim** — uses `rawOptions: '--hints:off'` to suppress verbose build chatter (CC/Hint lines) that Wandbox returns as `compiler_error`
- **Java** — `transform` strips `public` from `class` declarations (Wandbox requires a plain class name matching the filename)
- **Swift, OCaml, Erlang, Elixir** — disabled (commented out); broken server-side on Wandbox

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
npx serve .
```
