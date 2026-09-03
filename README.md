# TradingView indicators

Pine Script v6. Paste a file into TradingView → Pine Editor → *Add to chart*.

| File | What it is |
|---|---|
| [`ict_toolkit.pine`](ict_toolkit.pine) | FVG / IFVG, CISD, SMT, sessions, PD arrays |
| [`chart_info_watermark.pine`](chart_info_watermark.pine) | Chart-info panels + watermark, in one indicator |

---

# ICT Toolkit — `ict_toolkit.pine`

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

---

# Chart Info + Watermark — `chart_info_watermark.pine`

One indicator that replaces both a watermark script and a chart-info / quote
script. Everything is drawn with tables, so nothing touches price data.

## Panels

| Panel | Contents |
|---|---|
| **Watermark** | Title + subtitle, each with its own colour and text size. |
| **Quote** | A free-text block (multi-line text area). |
| **Symbol info** | Any combination of: custom text, symbol, exchange, timeframe, date, time, last price, change %, bar countdown. |
| **Extra line 1 / 2** | Two spare free-text lines, independently positioned — handy for placeholder strings. |

Every panel has its own: show toggle, screen position (vertical × horizontal),
text colour, background colour, border (hide / colour / width), text size,
horizontal + vertical text alignment, cell width and height (`0` = auto), and a
stack order.

## Stacking instead of overlapping

TradingView allows one table per screen position, so two panels sent to, say,
*bottom · center* would normally sit on top of each other. This script collects
every enabled line first and builds **one table per position**, with each
panel's lines as rows, ordered by the panel's *Stack order* (lowest on top).
Put the watermark and the symbol info both at *top · center* and they stack
cleanly. *Global → Separator lines between stacked rows* draws the border colour
between those rows.

## Placeholders

Any text field (title, subtitle, quote, info text, extra lines) expands these:

`{ticker}` `{symbol}` `{exchange}` `{description}` `{currency}` `{tf}`
`{date}` `{time}` `{price}` `{change}` `{countdown}`

So a subtitle of `{ticker} · {tf}` renders as `NQ1! · 5m`, and an extra line of
`{countdown}` gives a live "time until this bar closes" clock.

## Symbol info details

- **Element layout** — *Stacked* puts each element on its own table row (so the
  symbol can be rendered larger); *Inline* joins them all into one row with a
  configurable separator.
- **Element order** — a comma-separated key list: `text, symbol, exchange, tf,
  date, time, price, change, countdown`. Reorder it or drop keys from it, e.g.
  `date,symbol,tf` puts the date on the top row.
- **Render the symbol one size larger** — the symbol gets bumped one step above
  the panel's text size (`small` → `normal`, and so on).
- **Timeframe suffix** — defaults to `" TIMEFRAME"`, giving `5m TIMEFRAME`.
  Empty for a bare `5m`.
- **Symbol format** — ticker, exchange + ticker, full ticker ID, or the
  instrument's description.
- Timeframes print as `30s` / `5m` / `4H` / `1D` / `1W` / `3M`.

## Reproducing the two originals

- *AG FX watermark* — title + subtitle at `top · center`, symbol info at
  `bottom · center` with **Element order** `date,text`, **Text?** on and
  **Info text** `{symbol} | {tf}`, date format `d/M/yyyy`.
- *toodegrees quote / info* — quote panel on, symbol info at `top · right`,
  size `small`, **Element order** `text,symbol,tf`, timeframe suffix
  `" TIMEFRAME"`, *Render the symbol one size larger* on.

## Notes

- Date and time use the *Global → Timezone* setting (exchange, UTC or a custom
  IANA name) and can be sourced from the last bar or from the wall clock.
- Change % is measured against the previous daily close by default (one
  `request.security` call on `D` with `close[1]`, so it doesn't repaint), or
  against the previous bar's close.
- Panels are rebuilt on the last bar only; toggling any input redraws
  everything immediately.
- Up to 16 rows can share a single screen position; extra rows are dropped.
