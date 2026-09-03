# ICT Toolkit — TradingView indicator

Single Pine Script v6 indicator: **`ict_toolkit.pine`**.

Paste it into TradingView → Pine Editor → *Add to chart*. Clear the editor's default template first (Ctrl+A, delete) — leaving it in place gives you two `indicator()` calls and a compile error.

## Modules

| Module | Behaviour |
|---|---|
| **Sessions** | Asia / London / NY AM / NY PM, in a configurable timezone. Each session can show a running high–low box, horizontal high/low lines, and text labels — independently. |
| **Previous D/W/M** | Previous day, week and month high and low. Drawn with `plot()`, so they cannot be evicted by TradingView's drawing cap. |
| **PD arrays** | Premium / discount / equilibrium and OTE (0.705 / 0.295, plus 0.618 / 0.382) over a selectable range: previous day, week, month, or the current day. |
| **Status table** | Active session, PDH/PDL, PWH/PWL, premium-or-discount bias, bar count. |

Still to come: CISD, SMT divergence.

## Session options

- **Boxes** — colour per session, fill transparency, on/off.
- **High/low lines** — style (solid / dashed / dotted), width, and an extend mode: `While session is open`, `Until next session`, or `To right edge`.
- **Labels** — `Asia H` / `Asia L` and so on, with colour, text size, optional price, and a choice of labelling only the newest session of each type or every session kept in history.

Lines, boxes and labels are independent — you can run any combination, including labels with no lines.

## Styling

Colour, line style and width are exposed for every drawn element. Session and level colours use TradingView's colour picker, so opacity is adjustable on each.

## Repainting

- Session flags and levels are computed on confirmed data; nothing re-anchors after the fact.
- Previous D/W/M values use `high[1]`/`low[1]` with `lookahead_on`, which reads only *closed* higher-timeframe bars.

## Drawing budget

TradingView keeps only the newest 500 boxes / 500 lines / 500 labels per script and silently discards the oldest. Session drawings are therefore capped by `Sessions kept per type` (default 10, so 40 sessions total), and the D/W/M and PD levels are `plot()` calls rather than drawing objects, so they are exempt from the cap entirely.

`max_bars_back = 1000` is declared on the indicator. Without it, history references Pine cannot size on its own abort the script at runtime and the chart renders nothing at all.
