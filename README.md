# Tiny Bot — Beat Buy-and-Hold on Solana

[![make.fun · u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/zavonlame/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/rank.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/zavonlame/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/reward.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

A tiny paper-trading bot for hourly Solana prices, entered in the
[make.fun bounty](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1).
It trades FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH and WIF on the
`agent-fun/bars-replay/v1` harness: hold only what is actually rising,
otherwise sit in cash.

## Strategy

Per symbol, each bar:

- fast EMA (12h) above slow EMA (48h) **and** a positive 24h return of at
  least +1.00% → long signal;
- the signal must persist for 2 consecutive bars before entry, so a
  one-bar flicker in a choppy market does not buy the top of a bounce;
- positions are equal-weighted at 20% of equity (harness cap 25%), topped
  up only when off-target by more than 2% of equity;
- exit when the signal turns off, or on a 6% trailing stop under the peak
  close since entry.

Everything is a pure function of the bars seen so far: no clock, no
randomness, no network — the two grading runs reproduce byte for byte.

## Backtest (dev window, `grade-local --driver process`)

The dev window (136 hourly bars, 4–9 Nov 2026) falls the whole way down:
every asset ends 3–14% lower, so buy-and-hold loses heavily. The bot
loses far less by staying mostly in cash and only riding real bounces.

| Variant | Net PnL (USDC) | Trades | Max DD | Fees |
|---|---:|---:|---:|---:|
| Buy-and-hold (house baseline) | −869.34 | 7 | 10.9% | 19.14 |
| House EMA-cross baseline | −173.04 | 28 | 2.5% | 48.18 |
| House momentum baseline | −268.23 | 19 | 5.7% | 54.59 |
| V1 EMA 6/24, mom 12h +0.5% | −525.30 | 57 | 5.9% | 120.41 |
| V2 EMA 12/48, mom 24h +1.0% | −170.35 | 26 | 1.7% | 51.54 |
| **FINAL (V2 + 2-bar streak)** | **−100.14** | **18** | **1.1%** | **35.81** |
| V4 FINAL + 25% size, max 4 | −135.65 | 18 | 1.5% | 44.70 |
| V5 FINAL + mom +1.5% | −55.18 | 10 | 0.7% | 19.92 |
| V10 EMA 8/24, mom 12h +1.0% | −242.93 | 27 | 3.0% | 59.11 |
| V12 EMA 10/36, mom 18h +1.0% | −284.34 | 27 | 2.8% | 54.96 |

Honest notes: **no variant has a positive edge on the dev window** — the
market only falls there, so the whole game is losing less than
buy-and-hold. V5/V6 lose least (−$55/−$58) but make only 10 trades on 136
bars, too few to safely clear the 8-trade guardrail on the 72-bar final
window. The shipped FINAL keeps 18 trades, a 1.1% max drawdown, and still
beats both the buy-and-hold baseline (−$869.34) and the house EMA-cross
baseline (−$173.04). The final window (27–30 Nov) is unseen data; nothing
here promises a profit there.

## Layout

- `main.ts` — the bot (strategy + replay loop, own design)
- `vendor/makefun-bars.ts` — the kit's bars-replay shim, copied as its
  header instructs
- `.makefun/owner.json` — claim file naming the payout wallet

## License

MIT — see [LICENSE](LICENSE).
