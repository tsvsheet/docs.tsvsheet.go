---
title: Home
---

**A spreadsheet for plain text.** A `.tsvt` file _is_ the spreadsheet — a single TAB-separated grid whose cells are literal values or `=formulas` that address other cells in A1 notation (`B2`, `D2:D5`), computed in place, versioned as text, diffed line by line.

## Playground

[**Open the playground →**](playground/) — the real engine, compiled to WebAssembly, running entirely in your browser. No server, no upload: edit any cell (a value or an `=formula`), watch it recompute live, toggle the `.tsvt` source pane, and copy the result to your clipboard. `TODAY()`, `NOW()`, and `ISNOW()` tick against your own clock.

## Model

A `.tsvt` is the whole spreadsheet — one grid of cells, each either a literal value or an `=formula`. Formulas address other cells in A1 notation: a single cell (`B2`), a range (`D2:D5`), or a cross-sheet reference (`"prices"!A1`). Cells are computed in dependency order; a bad reference propagates `#REF!`, a cycle propagates `#CIRC!`. Because the file is plain TSV, it versions as text and diffs line by line.

The formula expression sublanguage is Excel-faithful: `^` (power), `&` (concat), postfix `%`, `TRUE`/`FALSE`, and error-value literals over A1 cell and range references — plus one unix-native addition, the [pipe operator](#pipes). The grid itself is plain TSV split by the host — only the `=formula` cells are parsed.

## Pipes

Formulas compose like a shell pipeline. `expr | fn(arg, …)` feeds the expression in as the function's **first argument**, so a multi-stage transformation reads left to right in execution order instead of inside-out:

```text
=A2:A10 | sort() | unique() | count()     ≡  =count(unique(sort(A2:A10)))
=A1 | round(2)                            ≡  =round(A1, 2)
=A1 & B1 | len()                          ≡  =len(A1 & B1)    (pipe binds loosest)
```

The pipe is **pure sugar over function calls** — the two spellings are the same formula, chains fold left, and error values flow through stages exactly as they do through arguments. There is no new execution model: the processor normalizes a pipe to its composed call when it parses the formula, and everything downstream (dependency order, memoization, `explain`, structural edits) sees an ordinary call — while your sheet keeps the spelling you wrote. Specified in [SPECIFICATION.md §5.4](https://github.com/tsvsheet/tsvsheet/blob/main/SPECIFICATION.md).

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
