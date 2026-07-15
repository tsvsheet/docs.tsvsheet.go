---
title: Home
---

**A spreadsheet for plain text.** A `.tsvt` file _is_ the spreadsheet — a single TAB-separated grid whose cells are literal values or `=formulas` that address other cells in A1 notation (`B2`, `D2:D5`), computed in place, versioned as text, diffed line by line.

## Playground

[**Open the playground →**](playground/) — the real engine, compiled to WebAssembly, running entirely in your browser. No server, no upload: edit any cell (a value or an `=formula`), watch it recompute live, toggle the `.tsvt` source pane, and copy the result to your clipboard. `TODAY()`, `NOW()`, and `ISNOW()` tick against your own clock.

## Model

A `.tsvt` is the whole spreadsheet — one grid of cells, each either a literal value or an `=formula`. Formulas address other cells in A1 notation: a single cell (`B2`), a range (`D2:D5`), or a cross-sheet reference (`"prices"!A1`). Cells are computed in dependency order; a bad reference propagates `#REF!`, a cycle propagates `#CIRC!`. Because the file is plain TSV, it versions as text and diffs line by line.

The formula expression sublanguage is Excel-faithful: `^` (power), `&` (concat), postfix `%`, `TRUE`/`FALSE`, and error-value literals over A1 cell and range references. The grid itself is plain TSV split by the host — only the `=formula` cells are parsed.

## Command line

The `tsvsheet` CLI renders, parses, checks, and explains sheets, serves a browser editor, or opens a bubbletea TUI — one shared engine behind all of them, with unix stdin/stdout discipline:

```text
tsvsheet render sheet.tsvt          # compute and print the grid
tsvsheet check  sheet.tsvt          # validate formulas and references
tsvsheet explain D5 < sheet.tsvt    # trace how a cell was produced
tsvsheet serve  sheet.tsvt          # local browser editor (reads/writes the file)
tsvsheet tui    sheet.tsvt          # terminal editor
```

A `.tsvt` can carry a `#!` shebang and `#` comments, so a sheet is directly executable — `render` is the default command.
