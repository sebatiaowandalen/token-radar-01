# Methodology: radar-2

This entry's model is one function of the sealed snapshot's figures, the same
for every protocol. No protocol name is special-cased and no row is hand-picked.

## Implied value

    implied = round(tvl_usd x category_multiple
                    x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
                    x (0.6 + min(age_days, 1095) / 1095 x 0.4)
                    x min(1.10, 1 + 0.02 x (chains - 1)))

- **TVL** is the protocol's total value locked, in whole US dollars.
- **Category multiple** is the median market-cap-to-TVL ratio of the
  live-token protocols in the same category, computed from the snapshot
  itself at run time (see below). It replaces the house model's flat 0.25.
- **Growth** is the 7-day TVL change, clamped to [-60%, +120%], at half weight.
  The clamp matters: one early protocol's 7-day change is +4448%, and without
  the clamp it would outrank the whole watchlist on one week.
- **Maturity** rewards time spent with no token: 0.6 for a brand-new protocol,
  rising to 1.0 at three years. Age is days since DefiLlama first listed the
  protocol; an unlisted age uses 365 days.
- **Chain factor** adds 2% per extra chain, capped at 1.10: a protocol live on
  more chains reaches more users for the same TVL, but chains never dominate
  the ranking.

## Category multiples, from the 2026-11-14 snapshot

Each multiple is the median mcap/TVL of the live-token protocols in that
category (129 live ratios in total). A category needs at least 3 live
comparables; anything thinner falls back to the global median, 0.223.

| Category | Multiple | Live comparables |
| --- | --- | --- |
| Canonical Bridge | 1.000 | 8 |
| Cross Chain Bridge | 1.000 | 4 |
| Bridge | 0.988 | 6 |
| Privacy | 0.969 | 3 |
| Basis Trading | 0.654 | 7 |
| Dexs | 0.342 | 11 |
| RWA | 0.275 | 9 |
| Yield Aggregator | 0.252 | 4 |
| Yield | 0.191 | 9 |
| CDP | 0.169 | 3 |
| Indexes | 0.148 | 3 |
| Lending | 0.132 | 15 |
| CEX | 0.075 | 9 |
| Liquid Staking | 0.050 | 9 |

The spread is the point: a bridge's TVL is valued near 1.0x, a liquid-staking
protocol's near 0.05x, because that is how the market values them. The
clamp to [0.05, 1.0] binds at both ends: Liquid Staking's raw median is
0.041, Canonical Bridge's is 1.126, Cross Chain Bridge's 2.205.

The fallback is deliberate, not a gap. Risk Curators — 12 of the 40
watchlist protocols — has exactly one live comparable, at a ratio of 2.97,
an outlier no honest median can rest on. Onchain Capital Allocator has two,
at 0.02 and 0.01. Trusting either would value the biggest watchlist names
off one or two trades, so both fall back to the global median of 0.223 and
are ranked on TVL, growth, maturity and chains instead.

## Worked example: Steakhouse Financial (rank 1)

- TVL $2.75B, Risk Curators, 11 chains, listed 2025-05-20 (age 543 days),
  7-day change -2.19%.
- Category multiple 0.223 (global median; Risk Curators has one live comp).
- Growth multiplier 1 + (-2.19 / 100 x 0.5) = 0.989.
- Maturity multiplier 0.6 + (543 / 1095 x 0.4) = 0.798.
- Chain multiplier min(1.10, 1 + 0.02 x 10) = 1.10.
- Implied: $2.75B x 0.223 x 0.989 x 0.798 x 1.10 = $532.7M.

The dashboard recomputes every row this way from the snapshot figures and
checks the result against `board.json`.

## Farm plans

Each row carries a 7-step plan, derived from the protocol's own category,
chains, TVL and age — never from a hand-written list of names. Capital
scales with TVL ($50 at the bottom, $250 above $5B); the hold is longer for
younger protocols (60 days under 6 months, 45 under a year, 30 after that).
The action fits the category: supply for lending, provide liquidity for
DEXs, bridge for bridges, stake for staking, deposit into a vault for
curators and allocators, mint for RWA and stable assets, buy cover for
insurance. The extra step against the house plan is the weekly points
check: a position that stops earning points is a position the next snapshot
will not count.

## What the board covers

- All 40 watchlist protocols: TVL at or above the $50M floor, no token on
  record, not a versioned pool, not a wrapped/staked/bridged product.
- All 186 rankable pre-token protocols below the floor, by TVL.
- The 24 largest live-token protocols are set aside in their own tab with
  their market caps shown, never mixed into the ranking.
