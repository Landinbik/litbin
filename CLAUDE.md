# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file (`index.html`) markdown pastebin. Zero backend. The entire document is stored in the URL hash as LZ-compressed, URI-encoded text — sharing a URL shares the content.

## URL storage model

```
https://example.com/#<LZString.compressToEncodedURIComponent(markdown)>
```

- The hash fragment is never sent to the server, so there is no server-side involvement
- Practical limit: ~200 KB markdown fits comfortably; warning appears at 500 KB encoded, error at 1.5 MB
- Uses [lz-string](https://github.com/pieroxy/lz-string) via CDN

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

## Development

No build step. Edit `index.html` and refresh the browser. To serve locally:

```bash
python3 -m http.server 8080
# or
npx serve .
```
