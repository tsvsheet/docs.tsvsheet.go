[![actions](https://github.com/uplang/docs.tsvsheet.go/actions/workflows/actions.yml/badge.svg)](https://github.com/uplang/docs.tsvsheet.go/actions/workflows/actions.yml) [![docs](https://github.com/uplang/docs.tsvsheet.go/actions/workflows/docs.yml/badge.svg)](https://github.com/uplang/docs.tsvsheet.go/actions/workflows/docs.yml) [![pages](https://github.com/uplang/docs.tsvsheet.go/actions/workflows/pages.yml/badge.svg)](https://github.com/uplang/docs.tsvsheet.go/actions/workflows/pages.yml)

# docs.tsvsheet.go

Public documentation for [`uplang/tsvsheet.go`](https://github.com/uplang/tsvsheet.go) — the Go implementation of [tsvsheet](https://github.com/uplang/tsvsheet), *a spreadsheet for plain text*. Published as a [Hugo](https://gohugo.io) site via GitHub Pages at [uplang.github.io/docs.tsvsheet.go](https://uplang.github.io/docs.tsvsheet.go/).

## Playground

[`static/playground/`](static/playground/) hosts the browser playground: the `tsvsheet` engine compiled to WebAssembly (`main.wasm` + `wasm_exec.js`) plus a single-page editor (`index.html`). It runs entirely client-side — no server — and is served at [`/playground/`](https://uplang.github.io/docs.tsvsheet.go/playground/). Rebuild and redeploy it from the code repo with `make wasm` (staging into `dist/playground/`), then copy `dist/playground/*` here.

## Layout

| Path | Purpose |
| --- | --- |
| [`content/`](content/) | Markdown documentation — the Hugo site content. |
| [`static/playground/`](static/playground/) | The WebAssembly browser playground, served at `/playground/`. |
| [`layouts/`](layouts/) | Hugo templates (managed fleet style — distributed from the docs template). |
| [`assets/`](assets/) | Hugo assets (managed fleet style — distributed from the docs template). |
| [`hugo.json`](hugo.json) | Hugo configuration. |
| [`Makefile`](Makefile) | Local preview and build. Run `make` for help. |

Private material (ideas, tasks, specs) lives in the project's hub repo, never here.
