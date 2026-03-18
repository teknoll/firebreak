# Canvas Chart Renderer Feature

## Problem

The dashboard needs a dynamic chart renderer that draws line graphs and bar
charts directly to a Canvas element. Currently all chart data is displayed in
HTML tables only, which cannot represent trends visually. User research shows
68% of users want visual charting. Requests account for 3 open support tickets
and two roadmap items held for Q2.

## Goals

- Render line and bar charts to an HTML Canvas element from structured data.
- Support at least 200 data points per series without visible frame drops.
- Allow the user to toggle between line and bar views without a page reload.
- Display axis labels and a legend derived from series metadata.

## User-facing behavior

When the user opens a dashboard panel configured for charting, a Canvas element
renders the chart. A toggle button above the chart switches between "Line" and
"Bar" modes. Switching mode re-renders immediately with a 150 ms CSS transition.

Axis labels are drawn inside the canvas using the series `label` property.
A legend block beneath the canvas lists series names with their color swatches.
Hovering a legend item highlights the corresponding series in the chart.

## Technical approach

The feature is implemented across two modules:

- `chart-renderer.js`: Reads series data from the application state store and
  calls Canvas 2D context methods to draw the chart.
- `chart-controller.js`: Handles the toggle button DOM event and dispatches a
  `CHART_MODE_CHANGED` action to the state store. `chart-renderer.js` subscribes
  to the store and re-renders on state change.

The two modules share an interface through the application state store: the
controller writes to `store.chartMode` and the renderer reads from it. There is
no direct import between the modules.

### Integration seam declaration

- [ ] **Seam 1 — state store interface**: `chart-controller.js` dispatches
  `{ type: 'CHART_MODE_CHANGED', payload: { mode: 'line' | 'bar' } }`. The
  `mode` field must use the exact string literals `'line'` and `'bar'`
  (not `"Line"`, not `"BAR"`). `chart-renderer.js` reads `store.getState().chartMode`
  and must handle an undefined initial value by defaulting to `'line'`.
- [ ] **Seam 2 — Canvas context injection**: `chart-renderer.js` receives the
  Canvas 2D context via a constructor parameter `ctx` typed as
  `CanvasRenderingContext2D`. Tests must supply a real context from a
  `<canvas>` element (or Playwright's Canvas API) rather than a plain object
  mock, so that `ctx.fillRect`, `ctx.strokeStyle`, and `ctx.measureText`
  resolve to real implementations.

## Testing strategy

Tests are mapped to user verification steps (UV) as noted below.

- **Unit test** — `chart-controller.test.js`: Dispatch `{ type: 'CHART_MODE_CHANGED', payload: { mode: 'bar' } }` and assert `store.getState().chartMode === 'bar'`. Covers store write side of Seam 1.
- **Unit test** — `chart-renderer.test.js` (jsdom with real Canvas context via `OffscreenCanvas`): Call `renderer.draw(singleSeriesDataset)` where `singleSeriesDataset` is `[{ label: 'Revenue', values: [10, 20, 30], color: '#4A90D9' }]`. Assert `ctx.beginPath` was called at least once. Covers renderer draw path — not a real-browser Canvas API test.
- **E2E test (Playwright, Chromium)** — `chart-render.e2e.spec.js`: Navigate to the dashboard chart panel, assert a `<canvas>` element is visible, call `page.evaluate(() => { const ctx = document.querySelector('canvas').getContext('2d'); return ctx !== null; })` and assert `true`. Then trigger mode toggle and assert the canvas pixel at coordinate (10, 10) differs from the pre-toggle snapshot (validates real Canvas API execution path, not jsdom stub). Covers UV-1, UV-2.
- **E2E test (Playwright, Chromium)** — `chart-legend-hover.e2e.spec.js`: Hover a legend item and assert the corresponding canvas series stroke width increases from `1` to `3` pixels as read via `getImageData`. Covers UV-3.
- **Performance test** — `chart-perf.bench.js`: Render a 200-point single-series dataset in a real browser environment, assert total render time reported by `performance.now()` delta is under 50 ms. Covers AC-03.

## User verification steps

- **UV-1**: Open the dashboard chart panel in a browser. Action: observe the canvas area. Expected outcome: a line chart is visible with axis labels along the x and y edges and a color swatch legend beneath the chart.
- **UV-2**: Click the "Bar" toggle button. Action: watch the canvas. Expected outcome: the chart redraws as a bar chart within 150 ms; no page reload occurs; the toggle label reads "Bar" in an active/selected state.
- **UV-3**: Hover over a legend item for one data series. Action: move the cursor over the legend label. Expected outcome: the corresponding series line or bars in the canvas are visually highlighted (thicker stroke or brighter fill) while the cursor remains over the legend item.

## Documentation impact

- Add "Chart Panel" entry to the Dashboard Configuration Guide.
- Update the component inventory with `chart-renderer` and `chart-controller`.
- Add a "Seam contracts" note to the developer guide describing the
  `CHART_MODE_CHANGED` payload shape and the `OffscreenCanvas` test pattern.

## Acceptance criteria

- **AC-01**: Chart renders to Canvas for both line and bar modes.
- **AC-02**: Mode toggle switches the chart type without a page reload.
- **AC-03**: Performance holds at 200 data points without frame drops.
- **AC-04**: Axis labels and legend render correctly from series metadata.

## Dependencies

- Application state store (already in bundle).
- Playwright (already in devDependencies).
- Design team color palette tokens for series colors.

## Open questions

- Should the chart support zooming or panning in the initial release?
- How should the renderer handle empty or null data series?
