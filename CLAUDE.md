# TRMNL LPP Timetable Plugin

E-ink display plugin for TRMNL showing LPP Ljubljana bus departure times for a station.

## Files

- `settings.yml` — plugin config: `polling` strategy, custom fields (`station_code`, `bus_lines`, `next_hours`, `previous_hours`), `polling_url` template.
- `full.liquid`, `half_horizontal.liquid`, `half_vertical.liquid`, `quadrant.liquid` — markup per TRMNL view size. Same structure in all four; only `data-overflow-max-cols` differs (2 for full/half_horizontal, 1 for half_vertical/quadrant).

There is no build step or transform — `settings.yml`'s `polling_url` hits the API directly and Liquid renders the raw JSON.

## Data source

`polling_url` calls `GET https://data.lpp.si/api/station/timetable` with `station-code`, one `route-group-number` per line in `bus_lines` (comma-split), `next-hours`, `previous-hours`. Full docs: https://data.lpp.si/doc/#api-Station-timetable

Response shape used by templates (`data.*`):
```
data.station.name
data.route_groups[].route_group_number
data.route_groups[].routes[].name
data.route_groups[].routes[].route_number_suffix
data.route_groups[].routes[].timetable[].hour
data.route_groups[].routes[].timetable[].minutes[]
data.route_groups[].routes[].timetable[].is_current
```
`route-group-number` is required by the API (no default) — `bus_lines` must be non-empty or the poll fails. API's own `next-hours`/`previous-hours` defaults (1h / 4h) differ from this plugin's `settings.yml` defaults (1 / 0); the plugin's defaults win since they're always sent.

## TRMNL markup rules (hard rules, apply to all `.liquid` edits)

- No inline `style="..."` or `<style>` blocks — framework classes only.
- No emojis in markup (no e-ink emoji glyphs).
- Don't wrap markup in `<div class="view view--*">` — platform adds it. Start with `layout`.
- Use `trmnl.com` for asset URLs, never `usetrmnl.com`.
- Content `<img>` tags need `image-dither`; small title_bar icons don't.
- Guard possibly-nil values (`{% if %}`, `default:` filter) — this repo already guards `route.timetable.size > 0` but not `data.station`/`group`/`route` themselves.
- Prefer one `.column` + `data-overflow-max-cols="N"` over manually splitting items into multiple `.column` divs (already followed here).

Full design system reference: https://github.com/usetrmnl/trmnl-agent-skills (see `skills/trmnl/references/template_guide.md` and `agent_prompt.md`).
