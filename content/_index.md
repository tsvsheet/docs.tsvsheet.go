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

## Examples

Everything below runs today, with the shipped engine and nothing else — each example was computed with `tsvsheet render` exactly as shown.

### Bars in the grid

`rept` scaled by `max` turns a column into a chart you can `cat`, diff, and review — the poor man's sparkline, entirely inside the language:

```text
Month	Revenue	Chart
Jan	1200	=rept("█", round(B2/max($B$2:$B$4)*20, 0))
Feb	1350	=rept("█", round(B3/max($B$2:$B$4)*20, 0))
Mar	1810	=rept("█", round(B4/max($B$2:$B$4)*20, 0))
```

```text
Month	Revenue	Chart
Jan	1200	█████████████
Feb	1350	███████████████
Mar	1810	████████████████████
```

### Shares that reflow

A percentage column over `sum` recomputes every share when any one number changes — the grid stays honest without a spreadsheet application in sight:

```text
Category	Monthly	Share %
Compute	4200	=round(B2/sum($B$2:$B$4)*100, 1)
Storage	1900	=round(B3/sum($B$2:$B$4)*100, 1)
Network	1100	=round(B4/sum($B$2:$B$4)*100, 1)
```

### A chart that is a sheet

Cells compute text — including SVG. A sheet with `input`/`output` is a function ([SPECIFICATION.md §8](https://github.com/tsvsheet/tsvsheet/blob/main/SPECIFICATION.md)), so a bar chart can be *written in tsvsheet* and embedded from any other sheet. `bar-chart.tsvt` scales its bars with `max`, assembles `<rect>` fragments with `concat`, and `output`s one complete SVG document:

```text
label	value	h	svg fragment
=input(1)	=input(2)	=round(B2/max($B$2:$B$5)*100, 0)	=concat("<rect x='20' y='", 120-C2, "' width='60' height='", C2, "' fill='#4C78A8'/>")
…	…	…	…
=output(concat("<svg xmlns='http://www.w3.org/2000/svg' width='325' height='145'>", D2, D3, D4, D5, "</svg>"))
```

Any report embeds it as an ordinary formula, and extracting the computed cell yields a rendered chart:

```text
Quarter	Revenue
Q1	120
Q2	180
Chart	=sheet("bar-chart.tsvt", A2, B2, A3, B3, …)
```

```text
tsvsheet render report.tsvt | awk -F'\t' '$1=="Chart"{print $2}' > chart.svg
```

The chart is a sheet: parameterized, versioned, diffable, forkable. A chart library is a directory of `.tsvt` files.

### Emitting other text languages

The same trick targets any text-based tool. An edge-list sheet whose third column assembles [Graphviz](https://graphviz.org/) statements becomes a rendered dependency diagram in one pipeline:

```text
service	depends-on	dot
api	db	=concat("  ", A2, " -> ", B2, ";")
api	cache	=concat("  ", A3, " -> ", B3, ";")
web	api	=concat("  ", A4, " -> ", B4, ";")
```

```text
{ echo 'digraph deps {'; tsvsheet render edges.tsvt | cut -f3 | tail -n +2; echo '}'; } | dot -Tsvg -o deps.svg
```

(String literals have no escape sequences — for a literal `"` inside emitted text, use `char(34)`.)

### Plotting computed output

The computed grid is plain TSV — the native input of the standard plotting toolchain. Feed it to [gnuplot](http://gnuplot.info/) from a pipeline, or to [d3](https://d3js.org/) directly — `d3.tsv()` parses `render` output as-is:

```text
tsvsheet render sales.tsvt > sales.tsv         # then: plot 'sales.tsv' with linespoints
const rows = await d3.tsv("sales.tsv", d3.autoType);
```

Formulas stay in the source, computation stays in the engine, and every downstream tool sees exactly what a spreadsheet export would have given it — minus the spreadsheet.
