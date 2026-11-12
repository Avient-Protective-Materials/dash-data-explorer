# Dash Data Explorer Skill

Technical guidance for building responsive, self-hosted Dash dashboards and
exportable Plotly figures from dense time-series and matrix data.

## Install

If you have received prior written authorization from the copyright holder,
install the `dash-data-explorer` skill from this repository with your agent's
skills installer. For example, with the `skills` CLI:

```bash
npx skills add <repository-url> --skill dash-data-explorer
```

## Scope

This skill covers technical implementation choices for Dash/Plotly
applications: historian-analysis provenance and state semantics, rendering
modes, callback state, UTC-aware zoom handling, zoom-aware recomputation,
Datashader integration, Plotly legends and colorbars, and PDF export
validation.

It uses a static local dataset as the default dashboard starting point. For
explicit live-exploration requirements, it describes how to use credential-free
MCP query recipes as the starting point for a validated, read-only ODBC or IIS
data adapter, including secure historian credential provisioning through
an approved OS secret store or encrypted vault, verified TLS/corporate trust,
incremental native-Parquet caching, and distinct dashboard versus historian
health states. It does not grant historian access or permit raw hosts,
arbitrary SQL, browser-side credential persistence, or plaintext secrets.

It intentionally does not prescribe visual design. Use `/frontend-design` for
visual hierarchy, typography, spacing, responsive composition, palettes, and
UX polish.

## Contents

- `skills/dash-data-explorer/SKILL.md` — operational rules for agents.

## License

This project is proprietary. All rights are reserved; see [LICENSE](LICENSE).

