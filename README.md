# SPC + SPC Studio

**SPC** is a programming language for building apps and UIs. **SPC Studio** is a browser IDE for writing and previewing SPC.
The language engine (`src/core`, `src/features`) is independent of the Studio UI (`src/studio`, `src/app.js`).

## Start
ES modules do not load from `file://`, so serve the folder over HTTP:
- Desktop / Termux: `python3 -m http.server 8000` then open `http://localhost:8000` (or `npm start`)
- Phone without a terminal: upload the folder to GitHub Pages or Netlify (drag-and-drop) and open the link

## Test
`npm test` (Node 20+, no dependencies) runs the unit tests in `tests/`.
Browser test (needs `npm i playwright` and the folder being served on port 8765): `node tests/e2e/browser.mjs`.

## Modules
| Path | Purpose |
|---|---|
| `src/core/errors.js` | `SpcError` (type, message, file, line, column, hint) |
| `src/core/lexer.js` | source text → tokens with line/column |
| `src/core/parser.js` | tokens → AST (plain objects: `k` = node kind, `line`, `col`) |
| `src/core/interpreter.js` | executes the AST; variables, functions, control flow, UI commands, scopes |
| `src/features/routing.js` | `route()`, matching (`:params`, query, `*`), page loading, history, 404 |
| `src/features/styling.js` | `style { }` and `ui.theme { }` → real CSS / CSS variables |
| `src/features/events.js` | `on.click/input/change/submit/hover`, `value()` |
| `src/features/components.js` | table of `ui.*` components |
| `src/spc.js` | public entry: `lex, parse, interpret, SpcError` (use the engine outside Studio) |
| `src/studio/storage.js` | persistence (localStorage behind a small interface) |
| `src/studio/file-manager.js` | add / rename / delete / open files |
| `src/studio/editor.js` | textarea editor, line numbers, error-line marking, Tab and auto-indent |
| `src/studio/terminal.js` | terminal output and error formatting |
| `src/studio/preview.js` | DOM host for the engine + preview back/forward bar |
| `src/app.js`, `src/main.js` | wiring and startup |

## How the engine works
`lex(source)` → `parse(tokens)` → `interpret(ast, host, files, entryFile, startFile)`.
The interpreter never touches the page: it calls the **host** object (`make, add, push, pop, top, css, root, into, print, nav, fail`),
which `preview.js` implements with the real DOM (and `tests/helpers.js` with a fake one).
Every error is an `SpcError` carrying the file it happened in. Pages and imports are parsed on demand from the `files` map.

## How modules communicate
`app.js` owns the project state and passes it down; Studio modules talk to `app.js` through callbacks.
Dependency direction: `app → studio → core/features`. `features → core` only. The engine never imports Studio code or uses `document`/`window`
(both rules are enforced by `tests/structure.test.js`, which also rejects circular imports).

## Add a new language feature
1. New token or operator? Edit `lexer.js`. New syntax? Add a node in `parser.js` (`stmt()` or the expression functions).
2. Execute it: add a `case` in `block()` / `ev()` in `interpreter.js`, or a new factory in `src/features/` (see `styling.js`) that receives the context `X`.
3. A new `ui.*` component: add it to `features/components.js`.
4. Errors: use `err(message, node, hint)` so the file and line are attached.
5. Add tests in `tests/` using `mk()` from `tests/helpers.js`.
