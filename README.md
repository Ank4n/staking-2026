# Polkadot Staking — retreat presentation

Static site. Two surfaces:

- `presentation.html` — slide deck. **Why → What → When**, keyboard nav.
- `expenses.html` — protocol-expenses simulator. Drag DAP split, see costs / APYs / flow update live.
- `index.html` — landing page linking the two.

## Running

Needs to be served over HTTP — `fetch('config.yaml')` won't work from `file://`.

```sh
cd /path/to/2026-05-retreat
python3 -m http.server 8765
```

Then open http://localhost:8765/

Any other static-file server works (`npx serve`, `caddy file-server`, etc.). No build step.

## Files

| File | Purpose |
|---|---|
| `index.html` | Landing — links to the deck + the simulator |
| `presentation.html` | The deck. Single-page, 2D slide grid (sections × subslides) |
| `expenses.html` | Protocol-expenses simulator (linked from landing and embedded in the deck's last What slide) |
| `lib.js` | Shared helpers: config loader, formula functions (`tiNew`, `yearlyEmission`, `incentiveWeight`), slider/checkbox builders |
| `style.css` | All styles. Dark theme, accent pink |
| `config.yaml` | All defaults, ranges, presets, timeline phases. **Edit here, refresh — no code changes needed** |
| `staking.md` | Source script (presenterm format) the deck is shaped from |
| `dump.md` | External-source dump (HackMD spec, forum posts, Ref 1827 — used to ground numbers) |
| `sources.md` | Quick-reference URL list + revenue/outflow numbers used in the leaky bucket |
| `ui_spec.md` | Earlier design spec for the simulator. Historical |
| `images/` | Slide image assets |

## Keyboard

In `presentation.html`:

| Key | Action |
|---|---|
| `→` / `←` | next / previous sub-slide. Crosses sections at boundaries. |
| `↓` / `↑` | next / previous within current section only |
| `Home` | jump to first slide |

The When section's phase navigator intercepts `←` / `→` to step through phases until the boundary, then falls through to slide nav.

URL hash tracks position (`#sectionIdx,slideIdx`) — refresh stays on the current slide.

## Editing content

**Slide copy:** edit `presentation.html` directly.

**Numbers, defaults, phases, presets:** edit `config.yaml`. The simulator and the When phase navigator both read from it.

**Adding a phase:** append to `timeline.phases` in `config.yaml`. Pills + nav update automatically.

**Adding a simulator preset:** append to `presets:` in `config.yaml` with the same shape as the existing entries.

## Dependencies

Loaded from CDN at runtime, no install:

- `js-yaml` — YAML config parser
- `chart.js` — used by the simulator

That's it. No bundler, no Node deps.
