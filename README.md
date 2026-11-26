# token-radar-01

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/sebatiaowandalen/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/sebatiaowandalen/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A token launch radar: it reads a sealed DefiLlama snapshot of every protocol
tracked, finds the ones that still record no token, and ranks them by what
their launch would be worth, with a farm plan for each and a static dashboard
that shows the board, a detail view per protocol and the model's own arithmetic.

## The model (radar-2)

Implied value is one function of the snapshot's figures, the same for every row:

    implied = round(tvl_usd x category_multiple
                    x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
                    x (0.6 + min(age_days, 1095) / 1095 x 0.4)
                    x min(1.10, 1 + 0.02 x (chains - 1)))

The category multiple is the point of this entry. Instead of the house
model's flat 0.25 for every protocol, each category is valued at the median
market-cap-to-TVL ratio of the live-token protocols in that category,
computed from the snapshot itself at run time, clamped to 0.05–1.0. A lending
protocol is valued like lending protocols trade, a bridge like bridges trade:
Liquid Staking 0.05 and Lending 0.13 at the bottom, RWA 0.28 and Dexs 0.34 in
the middle, Basis Trading 0.65 and Bridge 0.99 at the top. A category with
fewer than 3 live comparables falls back to the global median (0.22) rather
than trusting one or two trades. No protocol name is ever special-cased.

The board covers all 40 watchlist protocols (TVL at or above the $50M floor,
no token on record) plus every rankable pre-token protocol below the floor,
226 rows in all. The 24 largest live-token protocols are set aside in their
own tab with their market caps shown, never mixed into the ranking. Each row
carries a 7-step farm plan with a cost and a time per step, derived from the
protocol's own category, chains, TVL and age.

## Run it

Node 22.18 or later. The snapshot is the sealed input; point `--input` at the
kit's `input/` directory (or the mounted `/input` in the grader):

    mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <kit>/input

That writes `out/board.json`, `out/data.js`, `out/dashboard/`,
`out/screenshots/` and `out/dossier.json`. Open `out/dashboard/index.html`
in a browser to use the radar: search, filter by category, chain or minimum
TVL, watchlist only, sort by implied value, TVL, growth or age, and open any
row for its per-factor breakdown and farm plan. The page recomputes each
row's implied value from the snapshot figures and checks it against
`board.json`.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
