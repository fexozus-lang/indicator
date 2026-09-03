# ICT Toolkit — TradingView indicator

> **Licence: CC BY-NC-SA 4.0.** The Fair Value Gap module is adapted from *Fair Value Gap [LuxAlgo]*
> © LuxAlgo, published under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
> ShareAlike means this combined script inherits that licence: keep the attribution, keep the same
> licence on anything derived from it, and **do not use it commercially**. Removing the FVG module
> would lift that constraint.

Single Pine Script v6 indicator: **`ict_toolkit.pine`**.

Paste it into TradingView → Pine Editor → *Add to chart*. Clear the editor's default template first (Ctrl+A, delete) — leaving it in place gives you two `indicator()` calls and a compile error.

## Modules

| Module | Behaviour |
|---|---|
| **Sessions** | Asia / London / NY AM / NY PM, in a configurable timezone. Each session can show a running high–low box, horizontal high/low lines, and text labels — independently. |
| **Previous D/W/M** | Previous day, week and month high and low. Drawn with `plot()`, so they cannot be evicted by TradingView's drawing cap. |
| **PD arrays** | Premium / discount / equilibrium and OTE (0.705 / 0.295, plus 0.618 / 0.382) over a selectable range: previous day, week, month, or the current day. |
| **FVG** | Three-candle imbalance with a threshold filter (fixed % or auto, from the running average range). Boxes extend right until price closes through the far edge. Optional mitigation levels left behind on fill, levels on the N most recent unmitigated gaps, a higher-timeframe source, and a dynamic mode showing one running zone per direction that shrinks as price fills it. |
| **SMT divergence** | Swings on this chart are compared with up to two correlated symbols **at the same bars**. When one market makes a higher high and the other does not — or one makes a lower low and the other does not — the disagreement is drawn as a connector between the two swings and labelled. |
| **Status table** | Active session, which symbols SMT is comparing against, cumulative SMT signals, FVG counts and mitigation rate, PDH/PDL, PWH/PWL, premium-or-discount bias. |

Still to come: CISD.

## SMT options

- **Correlated symbol 1 / 2** — defaults are `CME_MINI:ES1!` and `CBOT_MINI:YM1!`. **Set symbol 1 to something other than your chart symbol** — comparing a market with itself can never diverge, so you would get no signals. The status table shows what it is comparing against, so this is easy to spot.
- **Swing pivot length** — a pivot only confirms this many bars after it prints; that lag is inherent to swing detection, not a flaw. Lower it for faster, noisier signals.
- **Max bars between swings** — ignores a divergence when the two swings are further apart than this (default 100, `0` = no limit). Stops distant, unrelated swings being paired up.
- **Connector** — style, width, label text and size. `Match chart text colour` draws in `chart.fg_color` so it reads on light and dark themes; turn it off to set explicit colours for highs and lows.

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
