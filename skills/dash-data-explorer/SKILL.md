---
name: dash-data-explorer
description: Build technically correct, responsive self-hosted Dash dashboards and Plotly reports for dense time-series and matrix data. Use for Dash callbacks, Plotly Heatmap/Scattergl behavior, Datashader integration, zoom-aware recomputation, legends, colorbars, and Kaleido PDF export.
---

# Dash Data Explorer

Use this skill when building a self-hosted, open-source Dash application or a
dense Plotly report. It covers technical rendering and interaction
behavior, not data-source semantics or visual design.

## Scope and dependencies

- Use `/frontend-design` for visual hierarchy, typography, spacing, responsive
  composition, palette selection, and UX polish.
- Consume upstream data provenance; do not infer native cadence, source
  semantics, or completeness when upstream metadata provides them.
- Preserve upstream distinctions between native, aggregate, interpolated, and
  derived data. Render those distinctions faithfully.
- Target open-source, self-hosted Dash. Do not assume Dash Enterprise services
  or components are installed.
- Do not recommend `dash-design-kit`, Snapshot Engine, `dash-enterprise-auth`,
  `EnterpriseDash`, or Dash Enterprise job services unless the project
  explicitly confirms them.

## Standalone application data adapters

### Start static

- Start with a static local dataset unless the user explicitly requests live
  historian exploration. Prefer a bounded MCP Parquet artifact or a supplied
  local file.
- Build zoom, hover, derived views, Datashader images, heatmap scaling, and PDF
  export so they operate entirely from the local dataset. This makes the first
  dashboard reproducible, portable, and usable without historian credentials.
- Preserve upstream provenance with the dataset: selected site, tags, time
  range, retrieval mode, data kind, row count, cadence, completeness, and any
  source-specific metadata.

### Add a live adapter only on request

- Add a live data pipeline only when the user explicitly requests arbitrary
  time-span exploration, data refresh, or deployment as a historian browser.
- Obtain a credential-free template through the MCP query-recipe capability
  after site and tags are verified. Treat its ODBC/IIS templates as the
  approved starting point for the app's data adapter, not as permission to
  expose arbitrary query inputs.
- Choose ODBC or IIS from verified retrieval semantics and deployment support:
  ODBC requires the approved SQLplus driver on the host; IIS is appropriate for
  supported ProcessData retrievals and portable Linux/WSL/server deployments.
  Do not silently switch transport; show the selected backend and provenance.
- Keep the standalone app's historian boundary at least as strict as the MCP:
  fixed approved site profiles, vetted tags, timezone-aware validated start/end
  timestamps, bounded duration and tag counts, explicit row limits, query
  timeouts, and read-only retrievals.
- A date-range control may change a validated start/end interval. It must not
  accept caller-provided historian hosts, arbitrary SQL, credentials, or
  unrestricted query text.
- Do not query a historian on every mouse move. Fetch only after a deliberate
  range/filter refresh, then use local slicing and rendering for hover, pan,
  and zoom interaction.

### Secure IIS credential provisioning

- Keep IIS authentication in the host OS keyring or an approved server-side
  secret manager. Never put passwords, authorization headers, or
  credential-bearing URLs in Dash layout, browser stores, callbacks, source
  control, logs, exports, or environment files.
- On first startup, detect missing IIS credentials and present a setup-needed
  state. The host administrator or operating-system user must complete an
  interactive provisioning step outside browser state, then store the
  credential through an approved keyring integration.
- On Windows, use Windows Credential Manager. On Linux, use an approved Secret
  Service/KWallet backend. For server deployment, the service account must own
  the keyring/secret access; browser users must never supply durable historian
  credentials to the app.
- Keep credential provisioning separate from normal dashboard callbacks. A
  moved or newly deployed app must require its own host/service-account secret
  setup rather than copying a user's credentials with the source tree.

## Choose the application architecture

- Use standalone Plotly HTML only for static snapshots that do not need
  range-dependent computation.
- Use a Dash app when zoom/pan must recompute Datashader images, heatmap
  colorscales, aggregations, derivatives, filters, or exports.
- Build the visible figure and exported figure through the same figure-building
  function. Browser, download, PDF, and interpretation must use the same
  selected range.
- Keep raw source data immutable. Derive visible subsets, aggregates, images,
  and figures from explicit UI state.
- Use `dcc.Store` only for small selection/filter state; do not place large raw
  datasets in browser component properties.

## Callback and range state

- Read `relayoutData` from any matched x-axis (`xaxis`, `xaxis2`, and so on),
  not only `xaxis`.
- Normalize range state to timezone-aware timestamps before slicing data.
- Preserve the selected range across stacked panels, derived calculations, and
  export callbacks.
- Keep callbacks deterministic. Inputs are UI state; outputs are derived solely
  from that state and stable source data.
- Avoid callback cycles. Do not write back a triggering property unless the
  state transition is explicit and guarded.
- Precompute static transforms at application startup. Cache deterministic
  expensive results only when keys include the selected range, filters, and
  rendering mode.
- Use `dcc.Loading` or an equivalent visible loading state around expensive
  panels. Show an explicit empty/error state instead of an ambiguous blank plot.

## Rendering policy

- Use native Plotly `Heatmap` for matrix/state views when hover and zoom-aware
  colorscales matter. Do not rasterize a heatmap whose colorscale must adapt to
  the visible range.
- Use `Scattergl` for dense but hoverable time-series lines, event traces, and
  derivative detail views.
- Use Datashader for dense overview lines when point-level hover is not useful.
- Use a hybrid strategy for dense derivatives: Datashader at wide ranges, then
  switch to colored `Scattergl` traces when visible points per series are below
  a practical threshold. Choose and document the threshold for the application.
- Prefer reducing trace count before optimizing point count. Hundreds of Plotly
  traces can be slower than a moderately dense set of traces.
- Avoid per-point marker styling, excessive annotations, and browser-side
  transfer of raw high-density arrays.

## Datashader embedded in Plotly

- Rasterize with deterministic viewport bounds derived from the current visible
  range.
- Recompute the image only when its range, filters, aggregation, or dimensions
  change.
- Embed the raster as a Plotly layout image anchored to valid date-axis
  coordinates.
- Add an invisible axis-anchor trace when needed so Plotly retains the image
  coordinate system.
- Verify that the image is actually visible in the rendered browser figure;
  accepted layout JSON alone is not proof.
- Add `legendonly` dummy Plotly traces for rasterized series when the legend is
  needed in the browser or exported PDF.

## Zoom and synchronized inspection

- On x-range zoom/pan, recompute heatmap `zmin` and `zmax` from visible columns
  only.
- Use diverging colorscales centered at zero for signed values such as
  deviations, residuals, and offsets.
- Use sequential colorscales for non-negative magnitudes such as force,
  counts, duration, or coverage.
- Use `matches="x"`, `hovermode="x unified"`, `hoversubplots="axis"`, and
  spike lines for synchronized inspection of stacked time panels.
- If built-in spikes do not span the required stacked panels, add an explicit
  hover-driven vertical shape using paper-referenced coordinates.
- Use `uirevision` deliberately to preserve user range state across callbacks;
  do not use it when the callback intentionally resets or recomputes the range.

## Layout, legends, and colorbars

- Position each heatmap colorbar from its actual subplot `yaxis*.domain`; do
  not use unrelated fixed paper coordinates.
- Keep legends as native Plotly legend entries when PDF or image export must
  include them.
- Reserve right-side space for legends and colorbars. Increase layout margins
  only as required; do not overlap controls, scale bars, and legends.
- Use stable series identifiers and color assignments across rendering modes,
  zoom levels, and exports.
- Use server-side filtering, sorting, and pagination for large Dash tables. Do
  not place full datasets in `dash_table.DataTable`.

## PDF and image export

- Provide a Dash download control that rebuilds the figure from current range
  and filter state before exporting with Kaleido.
- Ensure Chrome/Chromium is available to Kaleido before claiming PDF export is
  operational.
- Include active range/filter context in the exported title, subtitle, or
  annotation when it affects interpretation.
- Validate an actual zoomed PDF: verify PDF signature, expected page count,
  selected interval, legends, colorbars, and every required panel.
- Treat successful `figure.to_image()` execution as insufficient; inspect the
  rendered export for missing layout images, clipped legends, or blank panels.

## Minimum technical validation

1. Initial, wide-range, and zoomed views all render.
2. Every range-dependent callback uses the same normalized visible range.
3. Datashader and `Scattergl` branches both execute and preserve series labels.
4. Heatmap colorscales recompute from visible data and use an appropriate
   signed/magnitude scale policy.
5. Legends and colorbars do not overlap and are present in exported output.
6. The exported PDF/image uses the same active selection as the browser figure.
7. Empty, loading, and error states are visible and unambiguous.

