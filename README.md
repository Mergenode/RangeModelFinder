# ICT Range Deviation / PO3 Engine

A TradingView Pine Script v6 indicator that models Range Deviation as a causal lifecycle rather than a candle pattern:

`SEARCHING → ACCUMULATION → MANIPULATION → CONFIRMATION → EXPANSION → COMPLETE / INVALID`

The deployable indicator is [ict_range_deviation_po3.pine](./ict_range_deviation_po3.pine).

## What is implemented

- Rolling accumulation search across the configured minimum/maximum duration.
- Distinct RH/RL touch-run counting with range-, ATR-, or tick-based tolerance.
- Compression, boundary stability, duration, and boundary-expansion penalties.
- Bullish deviations below RL and bearish deviations above RH.
- Normal deviation depth plus fast-reclaim, strong-reaction, and wick-extension exceptions.
- Modular FVG, rejection, and volume-impulse liquidity evidence.
- Reclaim plus scored candle displacement confirmation; no signal at manipulation alone.
- Unique, non-overlapping setup lifecycle and one entry signal per setup.
- Rolling expansion scoring and `STRONG`, `HEALTHY`, `WEAKENING`, and `FAILED` states.
- Weighted 0–100 setup scoring and adjustable quality gates.
- Range, equilibrium, manipulation, entry, expansion, compact panel, debug labels, and alert conditions.
- Configurable historical-object retention.
- Data Window telemetry for staged validation and TradingView CSV export.

## Install

1. Open TradingView's Pine Editor.
2. Create a blank indicator and paste the full contents of `ict_range_deviation_po3.pine`.
3. Save it, choose **Add to chart**, and tune the inputs for the instrument and timeframe.
4. For alerts, create a TradingView alert and select one of the indicator's named conditions. Use **Once Per Bar Close** with the default non-repainting mode.

## Deterministic proxies

Several ICT terms do not have a single mathematical definition. This implementation isolates them as configurable proxies:

| Concept | Deterministic proxy |
|---|---|
| Boundary interaction | A distinct visit to the RH/RL tolerance zone; consecutive bars in-zone count once |
| Range stability | Drift between the oldest/newest halves, penalized for repeated boundary expansion |
| Compression | Range size divided by ATR |
| Strong reaction | Rejection wick/body, close location, directional close, and body/ATR score |
| Displacement | Body/ATR, body/recent-body, close location, opposite wick, directional sequence, FVG |
| Liquidity evidence | Independently switchable FVG, reaction, and volume-impulse modules |
| Expansion quality | Directional progress, directional bar ratio, body momentum, overlap, weak candles, opposing pressure |

These proxies are intentionally centralized in helper functions and inputs so they can be calibrated from historical results rather than hidden as magic numbers.

## Repainting behavior

Default behavior processes state transitions only when `barstate.isconfirmed` is true. Pine commits closed-bar state, so confirmed entry markers do not disappear because of later price action. The engine does not use future bars, negative offsets, pivots that require right-side bars, or higher-timeframe requests.

`Developing/repainting mode` permits intrabar previews. TradingView rolls unconfirmed realtime executions back between ticks, so an intrabar preview can disappear before the bar closes. Confirmed historical signals remain causal; expansion status is expected to evolve because it is a live trade-management observation.

## Calibration guidance

Start with the defaults, then tune one stage at a time in the order used by the engine: accumulation, manipulation, exceptions, confirmation, expansion, scoring. Enable debug mode to see transition reasons. The script is an indicator, not an order-executing strategy; its structural labels are not financial advice.

The `Range-length scan step` trades search precision for runtime. A value of `1` evaluates every length; the default `2` is lighter, while larger values are useful on very long chart histories.

## Staged validation

The indicator exposes state, setup ID, direction, range levels, deviation percentage, provisional/confirmation/expansion scores, confirmed-signal events, and invalidation events in TradingView's Data Window. This makes each development stage independently inspectable and allows chart data to be exported for offline analysis.

Suggested validation order:

1. Disable strict confirmation filters and inspect only accepted accumulation boxes, touch runs, and debug rejection reasons.
2. Verify manipulation direction, extreme, ratio, and duration against the locked range.
3. Test each extended-deviation exception separately around the normal depth limit.
4. Increase confirmation strictness and compare signal bars with the displacement telemetry.
5. Review expansion classifications over the configured rolling window.
6. Export chart data, group rows by setup ID, and calibrate thresholds on separate development and out-of-sample periods.

## Current-timeframe architecture

The initial version deliberately uses only chart-timeframe data. Detection, manipulation, evidence, confirmation, expansion, scoring, rendering, and alerts are separated into functions/sections so an HTF/LTF adapter can be added later without changing the lifecycle semantics. Any future HTF implementation should consume confirmed HTF values to preserve the default non-repainting contract.
