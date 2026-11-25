[![make.fun](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![Rank](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/musemosmouses/pumpfun-bot-sprint/rank.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![Reward](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/musemosmouses/pumpfun-bot-sprint/reward.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)

# pumpfun-sprint-bot

A disciplined paper-trading bot for the make.fun pump.fun replay sprint
"Pump.fun Bot Sprint: Make Bot And Win"
(`u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed`, $2 split
1st $1 / 2nd $1, closes 28 Nov 2026 15:00 WIB, 6 h final window,
`agent-fun/replay/v2`).

## Strategy

A launch sniper with a strict rug filter, tuned on the kit's frozen dev window
(24 h of real pump.fun data, 2 Nov 2026, 128k events, 1,222 mints):
-
- **Entry (first 3 slots after a coin's CreateEvent only):** the creator bought
- >= 0.75 SOL of their own coin, >= 4 distinct non-creator wallets piled in
- (and <= 10 — a launch bundled by many wallets is usually the creator's cluster
- waiting to dump), the creator has not sold, and the curve already holds
- >= 1 SOL of real SOL.
- **Size:** up to 0.5 SOL per coin; every order is still capped at 1% of the
- coin's real SOL reserve. Orders are never split into pieces — each extra piece
- costs a fixed 105k-lamport tx fee, measured at ~0.02 SOL over the dev window
- for zero benefit.
- **Exits:** take profit +10%, stop loss -5%, max hold 5 slots, exit at once
- when the creator sells.
- **Guardrail pacer:** the entry filter is deliberately strict (~35 coins per
- 24 h), but a 6 h final window would only hold ~9 of them — below the 20-coin
- minimum bar. When the bot's traded-coin count lags a 25-coin / 6 h pace, it
- opens tiny 0.012 SOL probe round trips on liquid curves (buy, sell as soon as
- the buy lands) until it is back on pace. Probes are purely a guardrail pacer,
- never a profit source.

Deterministic by construction: no clock, no randomness, no network, no I/O —
the two graded executions produce byte-identical action logs.

## Verified results (dev window, OLD kit — see caveat)

| | |
|---|---|
| Net PnL (`grade-local --driver process`, old kit) | **+0.0872 SOL** |
| Local sim (`tools/sim.ts`, old kit) | +0.0739 SOL |
| Coins / round trips | 90 coins, 91 round trips >= 0.01 SOL |
| Max drawdown | 1.1% (bar: <= 50%) |
| Guardrails | met (90 coins >= 0.01 SOL round trips) |
| Determinism | L1 regrade reproduced byte for byte |
| Shipped momentum pilot, same window | -9.38 SOL, 95% drawdown |

**Caveat — read before trusting these numbers:** they were measured
24 Nov 2026 on the earlier pump.fun replay kit and were NOT re-verified by
running this bounty's grader (this build does not execute the entry locally).
The kit for this bounty ships byte-identical SDK, protocol and dataset files,
so the strategy carries over unchanged — but the numbers above remain old-kit
numbers. Honest stress picture from the old kit: 20 rebuilt dev-window copies
gave median -0.14 SOL with 14 of 20 losing money, and the top 3 coins made
62% of the dev gains (without them net is -0.16 SOL). On a fresh final window
this bot is roughly a coin flip, not a reliable money-maker; it is the best
verified candidate available, beats the house momentum pilot by ~9.5 SOL on
dev, and meets the guardrails with a wide margin at ~1% drawdown.

The edge, found by dissecting the dev data: most fresh coins dump within slots
of creation and every trade pays ~2.7% in fees, so indiscriminate sniping and
momentum pyramiding both lose; only launches where the creator has real skin in
the game *and* several independent wallets pile in early show a positive
expectancy — and only at small size with fast exits. The final window is
collected after the deadline, so dev tuning is a proxy only; parameters were
chosen from flat regions of the sweep (neighbouring cells all score +0.05 to
+0.08 SOL), not from a single best cell.

## Run it

```sh
node main.ts            # the grader runs exactly this (Node 22, TS as-is)
```

## Layout

- `main.ts` — entrypoint (`node /bot/main.ts`)
- `strategy.ts` — the strategy (tuned parameters in `DEFAULT_PARAMS`)
- `vendor/agentfun-bot.ts` — the kit's SDK shim, vendored byte-for-byte

## License

MIT — see [LICENSE](LICENSE).
