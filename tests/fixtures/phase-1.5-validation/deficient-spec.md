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

## Testing strategy

- Unit test in jsdom: mount the chart component, assert `canvas` element is
  present in the DOM.
- Unit test in jsdom: dispatch `CHART_MODE_CHANGED` with `mode: 'bar'`, assert
  the store's `chartMode` field updates to `'bar'`.
- Unit test in jsdom: render a single-series dataset, verify no console errors
  are thrown.
- Unit test in jsdom: call `chart-renderer.js` draw method, assert the function
  completes without throwing.

## Documentation impact

- Add "Chart Panel" entry to the Dashboard Configuration Guide.
- Update the component inventory with `chart-renderer` and `chart-controller`.

## Acceptance criteria

- **AC-01**: Chart renders to Canvas for both line and bar modes.
- **AC-02**: Mode toggle switches the chart type without a page reload.
- **AC-03**: Performance holds at 200 data points without frame drops.
- **AC-04**: Axis labels and legend render correctly from series metadata.

## Dependencies

- Application state store (already in bundle).
- Design team color palette tokens for series colors.

## Open questions

- Should the chart support zooming or panning in the initial release?
- How should the renderer handle empty or null data series?
