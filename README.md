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
applications: rendering modes, callback state, zoom-aware recomputation,
Datashader integration, Plotly legends and colorbars, and PDF export
validation.

It uses a static local dataset as the default dashboard starting point. For
explicit live-exploration requirements, it describes how to use credential-free
MCP query recipes as the starting point for a validated, read-only ODBC or IIS
data adapter, including secure first-start IIS credential provisioning through
the host keyring or an approved server-side secret manager. It does not grant
historian access or permit raw hosts, arbitrary SQL, browser-side credentials,
or persisted plaintext secrets.

It intentionally does not prescribe visual design. Use `/frontend-design` for
visual hierarchy, typography, spacing, responsive composition, palettes, and
UX polish.

## Contents

- `skills/dash-data-explorer/SKILL.md` — operational rules for agents.

## License

This project is proprietary. All rights are reserved; see [LICENSE](LICENSE).

