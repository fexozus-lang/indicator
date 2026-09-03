# ICT Toolkit — TradingView indicator

Single Pine Script v6 indicator: **`ict_toolkit.pine`**.

Paste it into TradingView → Pine Editor → *Add to chart*.

## What it draws

| Module | Behaviour |
|---|---|
| **FVG** | 3-candle imbalance (`low > high[2]` / `high < low[2]`). Optional displacement filter (middle candle must close in the gap's direction) and a minimum size filter in ATR(200) multiples. Box extends right until mitigated. Optional CE (consequent encroachment) midline. |
| **IFVG** | When a bar *closes* through a gap's far edge, the zone flips polarity and is recoloured as an inverted FVG instead of being deleted. It is removed once price closes back through it. |
| **CISD** | Tracks the last consecutive run of same-direction *closes*. A close back through the **open of the first candle of that run** marks the change in state of delivery; the level is drawn from the run's origin. |
| **SMT** | Pivot highs/lows on the chart symbol are compared against up to 2 correlated symbols at the *same bars*. If one makes a higher high and the other doesn't (or one makes a lower low and the other doesn't), the divergence is drawn and labelled. |
| **Sessions** | Asia / London / NY AM / NY PM boxes with running high–low, in a configurable timezone. Session high/low lines keep extending after the session closes. |
| **PD levels** | Previous day / week / month high and low, anchored to the start of the current period. |
| **PD arrays** | Premium / discount / equilibrium and OTE (0.705, 0.618, 0.79 and mirrors) over a selectable range: previous day, week, month, or the current day. |

## Alerts

`Bullish FVG`, `Bearish FVG`, `IFVG`, `Bullish CISD`, `Bearish CISD`, `Bullish SMT`, `Bearish SMT`.

## Styling

Every drawing exposes **colour + line style (Solid / Dashed / Dotted) + width** in the settings panel:

- **FVG / IFVG** — separate fills for bullish/bearish FVG and bullish/bearish IFVG (each colour input has TradingView's opacity slider), zone border style & width, CE line style & width, and a zone-label mode: `None` / `IFVG only` / `All`.
- **CISD** — line colour per direction, style, width, extension length, and an optional floating `CISD` label at the right end of the level.
- **SMT** — connector style and width. `Match chart text colour` draws it in `chart.fg_color` so it reads on light and dark themes; turn it off to set explicit bullish/bearish colours.
- **Sessions** — per-session colour, box border style & width, fill transparency, and high/low line style & width.
- **Previous D/W/M** — colour, style and width per timeframe.
- **PD arrays** — four independent sets: boundary (0.0 / 1.0), equilibrium (0.5), OTE (0.705 / 0.295), and the remaining fib levels, each with its own colour, style and width.

Labels are drawn as floating text (`label.style_none`) rather than bubbles, so they sit beside the level instead of covering price.

## Repainting

Everything that changes structure is gated on bar close:

- FVGs are created and mitigated/inverted on **confirmed bars only**.
- CISD defaults to `Confirm on bar close = true`. Turning it off marks the break intrabar — faster, but the mark can disappear if the bar closes back inside.
- SMT pivots confirm `Swing pivot length` bars after the swing prints. That lag is inherent to pivots — lower the length for faster (noisier) signals.
- Previous D/W/M values use `high[1]`/`low[1]` with `lookahead_on`, which reads only *closed* higher-timeframe bars.

## Notes / limits

- `request.security` is called for 2 SMT symbols + 3 higher timeframes. Bad symbols are ignored rather than erroring (`ignore_invalid_symbol`), and SMT is simply skipped when the correlated feed has no data at a pivot.
- Drawing budget is capped at 500 boxes/lines/labels. `Max live gaps kept` controls how many FVGs stay on the chart.
- SMT compares the correlated symbol's high/low **at the chart symbol's pivot bars**, which is the usual convention. If both feeds don't trade the same hours, gaps are carried forward (`gaps_off`).
