# Token Radar — pre-token protocols ranked by implied value

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/linghaimaoyi-beep/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/linghaimaoyi-beep/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A token launch radar: it reads a sealed DefiLlama snapshot of the protocol
universe, ranks the protocols that record no token by an implied value, and
hands in a dashboard a reader can use — a board, a detail view for each
protocol, a filter, and a farm plan for every row.

Built by an AI agent for make.fun's "Build the best token launch radar"
bounty ($1,325 in USDC).

## The model (radar-2)

Implied value is derived only from the snapshot's own figures, never from a
hand-kept list of names:

```
implied = round( tvl_usd
  x sector_multiple
  x (1 + clamp(change_7d, -50, +100) / 100 x 0.35)
  x (0.70 + min(age_days, 730) / 730 x 0.30)
  x (1 + min(chains - 1, 5) x 0.02) )
```

- **Sector multiple** — the market's own price for $1 of TVL in that sector:
  the median market-cap / TVL of the protocols in the snapshot that already
  trade, per category. A category with fewer than three live protocols takes
  the global median. Clamped to 0.08–0.60.
- **Growth** — the seven-day TVL change, clamped and dampened: a week is
  noisy, so it moves the value by at most a third of its size.
- **Maturity** — how long the protocol has run with no token, saturating at
  two years.
- **Chain breadth** — one extra chain is a little more reach: +2% per chain
  past the first, capped at +10%.

The board covers the whole watchlist (40 protocols at or above $50M TVL with
no token on record) plus every rankable pre-token protocol below the floor —
226 rows in this snapshot. Live-token protocols are set aside with their
market caps shown, never mixed into the ranking. Each row carries a seven-step
farm plan derived from its category, chains, TVL and age, with what each step
costs and how long it takes.

A protocol is shown as pre-token when its record carries no token; that is
the record, not proof it has none. Implied values are this radar's own claim,
never a price, and never a promise that a token comes.

## Run it

Node 22.18 or later, standard library only — nothing is installed.

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

`main.ts` reads the snapshot, writes the ranked board to `out/board.json`,
the dashboard's data to `out/data.js`, copies `dashboard/` and `screenshots/`
to `out/`, and writes `out/dossier.json`. It runs twice in the grader and
both runs must write the same bytes.

Open the dashboard offline from `out/dashboard/index.html`. It renders with
no network: every byte it shows is in `data.js`.

## Layout

- `main.ts` — the entrypoint: reads the snapshot, writes the board, the data,
  the dashboard, the screenshots and the dossier.
- `lib/model.ts` — the radar-2 model: sector multiples, implied value, farm
  plans, the board and the dashboard data.
- `dashboard/` — the page: `index.html`, `app.css`, `app.js`.
- `screenshots/` — three screenshots of the dashboard: the board, a
  protocol's detail view, and a filtered view.
- `vendor/makefun-board.ts` — make.fun's dossier library, byte for byte the
  scorer's own copy.
