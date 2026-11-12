---
name: dash-data-explorer
description: Build technically correct, responsive self-hosted Dash dashboards and Plotly reports for dense time-series and matrix data. Use for Dash callbacks, Plotly Heatmap/Scattergl behavior, Datashader integration, zoom-aware recomputation, legends, colorbars, and Kaleido PDF export.
---

# Dash Data Explorer

Use this skill when building a self-hosted, open-source Dash application or a
dense Plotly report. It covers technical rendering, interaction behavior, and
the safe dashboard boundary for validated historian analyses; it does not
replace process expertise or visual design guidance.

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

## Preserve the validated analysis contract

- Resolve and document each approved tag's process meaning and signal polarity
  before adding controls, labels, derived metrics, or state summaries. Never
  infer that a value of `1` means active without confirmed process knowledge.
- Measure and retain observed native historian cadence per tag. A requested
  retrieval or display resolution is not evidence of stored sample frequency.
- Preserve and visibly label native/actual, aggregate, interpolated, and
  derived/corrected data in figures, tooltips, status text, and exports.
- Keep raw and derived/corrected signals separate. Make baseline, offset, and
  other correction assumptions inspectable rather than silently overwriting the
  source signal.
- Carry provenance through every data artifact and displayed result: source and
  approved site, selected tags and coverage, missing tags, UTC query range,
  retrieval mode/data kind, row count, and observed cadence.

## Historian data model

- Resample regular analog data only onto an explicit display grid, while
  retaining native records and their retrieval mode in the cache.
- Reconstruct sparse or discrete state history as state: combine event times
  with the display grid, forward-fill the last known state, and document a
  defined initial-state policy when earlier history is unavailable.
- Derive active-state durations and counts only from the confirmed polarity and
  state-reconstruction policy. Do not treat aggregate samples as event history.
- Clearly distinguish missing/unknown initial state from an inactive state in
  the rendered data and in aggregate calculations.

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
  range navigation, refresh, or scheduled live poll, then use local slicing and
  rendering for hover, pan, and zoom interaction.

### Cache native records incrementally

- Store retrieved native records locally in Parquet, deduplicated by approved
  tag and timestamp. Keep aggregate/interpolated retrievals separate; never
  silently overwrite native history with them.
- Cache completed historical windows for repeat browsing, with adjacent
  provenance that identifies the site, retrieval mode, query interval, tag
  coverage, observed cadence, and archive update time.
- For live data, fetch only the new tail plus a short documented overlap to
  capture late historian writes. Merge it into an append-only native archive,
  deduplicate, and render only the requested rolling window.
- Bound cache keys and file paths to approved profiles and server-controlled
  values. Do not derive filesystem paths from untrusted browser input.

### Secure credential provisioning

- Keep historian credentials server-side. Never put passwords, authorization
  headers, credential-bearing URLs, or secrets in Dash layout, browser stores,
  callbacks, logs, source control, exports, caches, or environment files.
- Persist credentials only in an approved secure OS secret store or encrypted
  vault; reject plaintext keyring and file fallbacks. Keep a vault passphrase
  only in memory for the running server session.
- A credential modal may submit credentials only to a trusted local Dash host.
  Remote use requires HTTPS and authenticated access. For server deployment,
  the service account must own the secret-store access; browser users must not
  supply durable historian credentials.
- After an approved vault is unlocked, reuse its encrypted stored historian
  credentials rather than requesting a password on every refresh.
- Open the credential dialog only for vault-lock, missing-credential, or
  historian `401`/`403` failures. Present unrelated retrieval and rendering
  failures as data/application errors, not credential prompts.

### TLS and corporate trust

- Keep TLS certificate verification enabled; never use `verify=False` to work
  around a trust failure.
- Prefer the host-system certificate store. Install required enterprise root
  and issuing CAs there; remember that WSL/Linux does not automatically inherit
  Windows trust roots.
- Support an explicit approved CA-bundle override only when necessary and only
  from server-side configuration.
- Verify the server's full certificate chain with OpenSSL and a real HTTPS
  request before diagnosing authentication. A `401` response can demonstrate
  that TLS succeeded.

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
- Store, query, validate, cache, and slice historian timestamps in UTC. Convert
  only at the presentation boundary to the operational timezone for axes,
  hover labels, titles, status text, and exports.
- Use `dcc.Store` only for small selection/filter state; do not place large raw
  datasets in browser component properties.

## Callback and range state

- Read `relayoutData` from any matched x-axis (`xaxis`, `xaxis2`, and so on),
  not only `xaxis`.
- Convert browser zoom ranges to timezone-aware UTC timestamps before slicing,
  caching, or historian retrieval. Use timezone-aware conversion so DST changes
  are correct.
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
- Report dashboard-server health separately from historian-data health. Start a
  live-refresh countdown only after a successful historian retrieval, and reset
  or mark it stale when the historian becomes unavailable.

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
- Include the active provenance needed to interpret the export: site, tags,
  operational timezone and displayed interval, retrieval mode/data kind, and
  any native-versus-aggregate distinction.
- Validate an actual zoomed PDF: verify PDF signature, expected page count,
  selected interval, legends, colorbars, and every required panel.
- Treat successful `figure.to_image()` execution as insufficient; inspect the
  rendered export for missing layout images, clipped legends, or blank panels.

## Minimum technical validation

1. Initial, wide-range, zoomed, and empty views all render with unambiguous
   loading/error status.
2. Every range-dependent callback uses the same normalized visible range.
3. Datashader and `Scattergl` branches both execute and preserve series labels.
4. Heatmap colorscales recompute from visible data and use an appropriate
   signed/magnitude scale policy.
5. Legends and colorbars do not overlap and are present in exported output.
6. A real zoomed PDF/image uses the same active selection as the browser figure
   and contains all panels, legends, colorbars, and data-kind provenance.
7. Native cadence/provenance, state polarity, initial-state behavior, and any
   derived-signal assumptions remain visible and correctly applied.
8. Credential, vault-lock, TLS, historian, and successful live-refresh states
   are distinct; only credential-specific failures open the credential dialog.
9. Live tail refreshes deduplicate overlapping native records and do not alter
   already archived native history with aggregate or interpolated values.

