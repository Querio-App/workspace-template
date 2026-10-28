---
name: notebook-code-mode
description: Drive the live notebook runtime through querio code exec. Use for every request that inspects or changes notebook state when Bash is available.
---

# Notebook Code Mode

The active runtime is the source of truth. It holds the kernel namespace, cell state, outputs, and reactive dataflow graph. The user interacts with that same runtime through the notebook UI.

Only Code Mode changes notebook cells. Use it for every durable notebook change and read current cell code through `ctx.cells[...].code`.

Every call to `create_cell`, `edit_cell`, `delete_cell`, `move_cell`, `set_layout`, `run_cell`, or `set_ui_value` must use this exact top-level async transaction shape:

```bash
querio code exec <<'PY'
from code_execution_environment import code_mode

async with code_mode.get_context() as ctx:
    cell_id = ctx.create_cell("x = 1\nx", hide_code=False)
    ctx.run_cell(str(cell_id))

cell = ctx.cells[str(cell_id)]
print(cell.status, cell.errors, cell.output)
print(ctx.globals["x"])
PY
```

A context obtained without `async with` is read-only. A synchronous `with code_mode.get_context()` block raises `TypeError`; always write `async with`. Never use `asyncio.run(...)` or `run_until_complete(...)`.

## Execute code

Use `querio code exec` through Bash for every interaction with the selected runtime. Pass multiline Python over stdin with a single-quoted heredoc.

The command evaluates Python in a temporary scratchpad with a shallow copy of kernel globals. Notebook variables are available by name. New top-level bindings and rebindings are discarded after the call, while deliberate in-place mutations to notebook-owned objects can persist.

In an editable session, `querio code exec` waits for the resulting notebook save before reporting success. A save failure does not undo execution or notebook edits. Keep the session open, resolve the save error, and inspect the existing notebook instead of replaying authoring code. Cell `idle` status alone establishes execution, not a successful save.

Use ordinary Python for transient work:

```bash
querio code exec <<'PY'
print(df.head())
PY
```

The scratchpad supports top-level async code. Use `async with` directly.

Set the Bash timeout from your estimate of the work, with room for query and data-transfer variability. Bash defaults to 120 seconds; the notebook kernel and CLI do not impose a separate execution-duration limit. Keep `querio code exec` attached to its caller. A timed-out or disconnected execution requests notebook interruption and cancels scratchpad executions already waiting in that session. A timeout is feedback to reassess the workload or split it into smaller pieces, not a reason to submit repeated probes or duplicate jobs. Interruption is not rollback: completed file writes and other side effects can remain. Check existing results before retrying, and do not promise that cancellation stopped a remote warehouse job.

Mutation methods are synchronous. Call `create_cell`, `edit_cell`, `delete_cell`, `move_cell`, `set_layout`, `run_cell`, and `set_ui_value` directly inside the context without `await`. They queue operations that apply when the context exits normally. If the block raises, its queued operations are discarded.

The context remains usable after the block exits. Applied cell state is available through `ctx.cells`, and values produced by execution are available through `ctx.globals`. Inspection, mutation, execution, and verification may happen in one command or across several commands. Choose the structure that fits the task.

`create_cell` defaults to `hide_code=True`. Pass `hide_code=False` for normal visible notebook cells.

## Author notebook cells

Notebook cells are the durable deliverable.

Default to working rather than asking questions. When a request has a reasonable analytical interpretation, choose it and produce useful work first. Mention any chosen scope briefly after completing the work instead of blocking on clarification.

Match the notebook to the request instead of following a fixed sequence of steps. Keep direct requests lean. For analytical questions, start with a title markdown cell, build the analysis in ordered data and visualization cells, and end with a summary markdown cell containing the findings, interpretations, and recommendations. Make the result understandable to a stakeholder reading the notebook without the chat.

Compare equivalent timeframes for period comparisons. Use dynamic SQL date expressions when the user gives a relative period such as last month or current year. When the user asks for a query, execute it and show its results instead of returning raw SQL in chat.

Every rendered table already has copy and CSV download controls in its top-right corner. When the user asks to export an existing table, answer with that location immediately. Do not inspect or change the notebook, run the table, create files, or build custom download machinery.

### Querio cell APIs

Every cell is Python. Import the notebook API with this canonical form:

```python
import code_execution_environment as cee
```

Use `cee.sql()` for datasource queries. Pass the datasource slug shown in the system context and assign the returned dataframe to a descriptive public name:

```python
import code_execution_environment as cee

revenue_by_region_df = cee.sql(
    """
    SELECT region, SUM(amount) AS total_revenue
    FROM orders
    GROUP BY region
    ORDER BY total_revenue DESC
    """,
    datasource="mydb",
)
```

`cee.sql(query, datasource=...)` returns a pandas dataframe and displays it as the cell output by default. Use `output=False` only when the table should not be displayed. Do not invent connection objects or alternate query APIs.

Use `cee.md()` for markdown:

```python
import code_execution_environment as cee

cee.md(f"## Revenue Summary\nTop region: **{revenue_by_region_df.iloc[0]['region']}**")
```

The last expression is the cell output. Keep widget definitions and code that reads their `.value` in separate cells so reactive dependencies work.

Available widgets are `checkbox`, `date_picker`, `dropdown`, `multiselect`, `number_input`, `radio`, `slider`, `table`, and `text_input` under `cee.ui`. Before authoring a widget-backed query, inspect the relevant widget and SQL documentation together in one scratchpad call:

```python
import code_execution_environment as cee

help(cee.ui.dropdown)
help(cee.sql)
```

The active runtime documentation is authoritative for signatures, values, reactive structure, and SQL construction. Use `cee.hstack([...])` and `cee.vstack([...])` when a cell needs to display multiple related elements.

### Variables and cell structure

Each public variable has one owning cell. Preserve its authoritative definition and edit that cell when changing it. Use a new descriptive public name for a downstream transformation. Names with one leading underscore are cell-local.

Use names such as `customer_orders_df`, `revenue_by_region`, and `top_products_df`. Avoid names such as `df1`, `data`, `temp`, and `result`. Keep extraction, calculation, and presentation concerns in separate cells when they produce reusable values.

Fix failed cells in place. Do not abandon an error cell and create a duplicate replacement.

### Visualizations

Use Altair for notebook charts unless the user explicitly requests another library. Do not call `plt.show()` or use any plotting API that opens a desktop GUI.

Build charts from an upstream dataframe and make the chart object the last expression:

```python
import altair as alt

revenue_chart = (
    alt.Chart(revenue_by_region_df)
    .mark_bar()
    .encode(
        x=alt.X("region:N"),
        y=alt.Y("total_revenue:Q"),
    )
    .properties(title="Revenue by Region")
)
revenue_chart
```

Put titles on charts with `.properties(title=...)`, not in separate title cells. Produce one focused chart per cell. Reuse one upstream query across charts at the same grain instead of duplicating SQL. For a large dashboard, fetch at most five broad base dataframes and derive the charts from them with pandas instead of issuing a query per chart. Keep chart-specific filtering and reshaping inside the chart cell so users do not see duplicate intermediate tables.

The Querio theme handles responsive sizing. Do not set width or height unless the user requests dimensions. For semantic colors, use encoding values such as `color=alt.value("red")`; mark-level colors may be overridden by the theme.

Do not use layered, concatenated, repeated, or faceted chart containers unless the user explicitly requests a combined view. Prefer color and `xOffset` encodings for grouped comparisons. Use tooltips instead of a second text layer for labels.

Use temporal axes for line and area charts. Never use a bare temporal encoding such as `alt.X("date:T")` or `alt.X("month:T")` with `mark_bar()`. For monthly bars, use `alt.X("yearmonth(date):O")` so every month occupies a full discrete band. Convert numeric category values to clean strings before matching explicit nominal scale domains. Rotate long or dense category labels and format large quantitative axes compactly.

## Reactive graph rules

The notebook is a directed acyclic graph:

- Cells cannot form dependency cycles.
- Each public name has one owning cell.
- Wildcard imports are not allowed because they prevent static analysis.
- Names beginning with a single underscore are local to their cell and unavailable to other cells.
- Running a cell reactively executes its affected descendants. Do not rerun descendants individually unless the task requires an explicit rerun.

Code Mode validates structural changes before committing them. A structural validation failure rejects the transaction. Cell execution can still fail after structural validation.

## Inspect notebook state

Use the live document and graph views whenever current notebook state matters:

```bash
querio code exec <<'PY'
from code_execution_environment import code_mode
ctx = code_mode.get_context()
[(cell.id, cell.code, cell.status, cell.errors, cell.output) for cell in ctx.cells]
PY
```

`ctx.cells` is the ordered document view and `ctx.graph` is the dataflow view:

```python
for cell in ctx.cells:
    cell.id
    cell.code
    cell.config
    cell.status
    cell.errors
    cell.output
    cell.console_outputs

ctx.cells[0]
ctx.cells["cell-id"]
ctx.cells.find("substring")
ctx.cells.grep(r"pattern")
list(ctx.cells.keys())

for cell_id, implementation in ctx.graph.cells.items():
    implementation.defs
    implementation.refs

ctx.graph.descendants(cell_id)
ctx.graph.ancestors(cell_id)
```

Cell IDs are opaque strings. Query them from `ctx.cells` or retain the ID returned by `create_cell`.

### Cell execution lifecycle

Inspect status after the async transaction exits, not while execution is still queued. Creation and edits can execute affected cells at commit; omitting an explicit `run_cell` does not imply that a newly created cell is unexecuted.

- `stale`: nonempty code has not run, has changed since execution, or depends on changed state. It is not a successful current result.
- `queued` / `running`: execution is pending / in progress.
- `idle`: the current cell executed successfully. It does not mean never executed. A successful cell need not produce a displayed output.
- `exception`: the cell raised an exception. Inspect `cell.errors`, repair the owning cell, and rerun it.
- `querio-error`: execution was prevented by a notebook/runtime error; inspect `cell.errors` for the cause.
- `cancelled`: an ancestor failed, so this cell did not execute successfully. Fix the ancestor rather than rerunning each cancelled descendant.
- `interrupted`: execution was interrupted; completed side effects are not rolled back.
- `disabled`: execution is disabled directly or through an ancestor.
- `None`: an empty cell without an execution result.

After a run, inspect `ctx.result.succeeded`, `ctx.result.errors`, and the affected cells' statuses/errors before reading their globals. A failed ancestor can leave a dataframe undefined; blindly printing it adds a secondary scratchpad error. CLI execution errors lead with notebook failures and identify subsequent scratchpad errors separately. Detailed per-cell tracebacks remain in `ctx.result.errors`; print those only when the concise error is insufficient.

## Write notebook changes

Submit Python cell source to `create_cell` and `edit_cell`.

Read before replacing. The user can edit concurrently, and Code Mode protects against stale writes. Before calling `edit_cell`, read the current body from `ctx.cells[target].code` and submit the full replacement. If the notebook changes between inspection and commit, re-inspect and retry from current state rather than bypassing revision checks.

Reuse existing imports and public names. If a dependency is already available from another cell, reference it instead of adding a private duplicate import to bypass the graph. Edit the owning cell when changing a public definition, or choose a genuinely new descriptive name.

Queue `run_cell` for cells that should execute. Related changes may share one transaction, and applied state may be inspected after the block in the same command.

A cell is also a rerun boundary. Put expensive reusable computation upstream of presentation so presentation changes stay cheap.

## Prefer Code Mode-managed changes

Use Code Mode whenever an API exists:

- Use `ctx.edit_cell(...)` even for a one-character live change.
- Use `ctx.move_cell(...)` for ordering.
- Use `ctx.set_layout(...)` for notebook layout. The grid is 24 columns wide, with 40px rows and 10px gaps. Positions and sizes are in grid units; vertical space is unbounded.
- Use `ctx.set_ui_value(...)` for supported UI values.
- Delete deliberately because deletion removes the cell's values from kernel state.

Before deleting a cell, inspect its descendants when the impact is not obvious. Never delete a valid cell because it seems redundant, old, or unnecessary, including when the user vaguely asks to clean up or tidy the notebook. Ask what they want removed unless they identify a specific target. Preserve existing work unless deletion is explicitly requested or required to remove an unrecoverable broken cell.
