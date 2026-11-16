# Starknet BTC-staking α: reproduction script

`alpha_repro.py` is a single self-contained script that reproduces every backtest figure from the forum proposal *"Fixed α makes BTC staking yield a bet on STRK price: proposal for a floating α"* and the follow-up reply.

All inputs are embedded in the script:
- 21 on-chain snapshots;
- daily STRK and BTC prices from CoinGecko.

The script needs only the Python standard library.

## How to run

Requires Python 3.9 or newer. No dependencies.

```bash
python3 alpha_repro.py
```

This reruns all calculations: the fixed-α baseline, the original floating-α rule, the revised SNIP-35-style controller with its variants, the days on which BTC APR exceeded STRK APR, and the forward scenarios. The expected result is in [`output/backtests.txt`](output/backtests.txt).

### Verify the embedded on-chain data (optional)

You need a Starknet mainnet RPC endpoint that serves historical state (an archive node, JSON-RPC 0.7+):

```bash
RPC=https://your-starknet-node python3 alpha_repro.py --verify
```

For each of the 21 snapshots, the script calls `RewardSupplier.calculate_current_epoch_rewards()` and `Staking.get_total_stake_for_token()` at that snapshot's block and prints the maximum relative difference from the embedded values. Expected: about 1e-10, because values are stored to 10 significant digits. See [`output/verify.txt`](output/verify.txt).

## Expected output

```
window: 337 steps, 2025-11-14 00:00:00+00:00 .. 2026-10-15 08:10:40+00:00

fixed alpha 25% (status quo)        33.5M STRK (+0%)  $1.95M (+0%)  24.9 BTC (+0%)  BTC APR avg 3.38% min 1.73% max 10.37%  alpha [25.0-25.0%]  days BTC>STRK 2
-- original post: k=0.4, 7d TWAP, [10%,33%], no rate cap
original floating                   36.1M STRK (+8%)  $2.01M (+3%)  25.5 BTC (+2%)  BTC APR avg 3.31% min 1.57% max 5.33%  alpha [12.1-33.0%]  days BTC>STRK 0
-- reply to sluhai: SNIP-35-style controller, a0=0.25, spot, cap 33%
k=0.4 d=0.5%/epoch                  36.4M STRK (+9%)  $2.01M (+3%)  25.5 BTC (+2%)  BTC APR avg 3.30% min 1.49% max 4.70%  alpha [12.2-33.0%]  days BTC>STRK 0
k=0.4 d=0.2%/epoch                  36.4M STRK (+9%)  $2.03M (+4%)  25.7 BTC (+3%)  BTC APR avg 3.33% min 1.40% max 6.16%  alpha [13.6-33.0%]  days BTC>STRK 0
k=0.4 d=1%/epoch                    36.5M STRK (+9%)  $2.01M (+3%)  25.5 BTC (+2%)  BTC APR avg 3.30% min 1.72% max 4.35%  alpha [9.0-33.0%]  days BTC>STRK 0
k=0.4 d=0.5% + 7d TWAP              36.1M STRK (+8%)  $2.01M (+3%)  25.5 BTC (+2%)  BTC APR avg 3.31% min 1.57% max 5.33%  alpha [12.1-33.0%]  days BTC>STRK 0
k=0.4 d=0.5% + 30d TWAP             35.3M STRK (+5%)  $2.01M (+3%)  25.4 BTC (+2%)  BTC APR avg 3.32% min 1.61% max 7.89%  alpha [17.2-33.0%]  days BTC>STRK 1
k=0.4 d=0.5% cut-only (max 25%)     30.3M STRK (-9%)  $1.69M (-13%)  21.5 BTC (-13%)  BTC APR avg 2.86% min 1.49% max 4.50%  alpha [12.2-25.0%]  days BTC>STRK 0
k=0.5 d=0.5%/epoch                  42.5M STRK (+27%)  $2.34M (+20%)  29.8 BTC (+20%)  BTC APR avg 3.92% min 1.87% max 5.63%  alpha [14.8-33.0%]  days BTC>STRK 0
k=0.6 d=0.5%/epoch                  45.0M STRK (+34%)  $2.51M (+29%)  32.0 BTC (+28%)  BTC APR avg 4.24% min 2.25% max 6.41%  alpha [17.1-33.0%]  days BTC>STRK 0

-- days with BTC APR > STRK APR under fixed alpha
   2026-10-11  STRK $0.1032  BTC APR 8.75%  STRK APR 7.69%
   2026-10-12  STRK $0.1230  BTC APR 10.37%  STRK APR 7.69%

-- forward scenarios (12 months, geometric paths, S constant). STRK APR 7.71%, crossover price today $0.0941
   STRK->$0.15 BTC x1: fixed BTC APR at end  12.3%, paid $ 4.3M; floating APR 3.08%, paid $1.49M; BTC>STRK from day 115
   STRK->$0.15 BTC x2: fixed BTC APR at end   6.1%, paid $ 4.3M; floating APR 3.08%, paid $2.15M; BTC>STRK from day -
   STRK->$0.3  BTC x1: fixed BTC APR at end  24.6%, paid $ 6.5M; floating APR 3.08%, paid $1.49M; BTC>STRK from day 57
   STRK->$0.3  BTC x2: fixed BTC APR at end  12.3%, paid $ 6.5M; floating APR 3.08%, paid $2.15M; BTC>STRK from day 115
   STRK->$0.5  BTC x1: fixed BTC APR at end  41.0%, paid $ 8.9M; floating APR 3.08%, paid $1.49M; BTC>STRK from day 42
   STRK->$0.5  BTC x2: fixed BTC APR at end  20.5%, paid $ 8.9M; floating APR 3.08%, paid $2.15M; BTC>STRK from day 66
   STRK->$1.0  BTC x1: fixed BTC APR at end  81.9%, paid $14.2M; floating APR 3.08%, paid $1.49M; BTC>STRK from day 31
   STRK->$1.0  BTC x2: fixed BTC APR at end  41.0%, paid $14.2M; floating APR 3.08%, paid $2.15M; BTC>STRK from day 42
```

How to read a line:
- **STRK / $ / BTC**: total rewards paid to BTC stakers over the window, in STRK, in USD at each day's price, and in BTC. The percentage is the change against fixed α = 25%.
- **BTC APR avg/min/max**: gross BTC staking APR at 0% commission.
- **alpha**: the range α took over the window.
- **days BTC>STRK**: the number of steps on which BTC APR exceeded STRK APR.

## Data

| | Source |
|---|---|
| Snapshots | `RewardSupplier` `0x009035556d1ee136e7722ae4e78f92828553a45eed3bc9b2aba90788ec2ca112`: per-epoch (STRK, BTC) rewards; `Staking` `0x00ca1702e64c81d9a07b86bd2c540188d92a2c73cf5cc0e508d949015e7e84a7`: total stake for STRK and for each BTC wrapper (WBTC, tBTC, SolvBTC, LBTC, strkBTC). `get_alpha()` = 25 at every snapshot. |
| Prices | CoinGecko `market_chart`, USD, daily, fetched 2026-10-15 ≈08:11 UTC. Rounded to 7 significant digits, which leaves the outputs unchanged at the printed precision. The last point is CoinGecko's intraday point. |

## Conventions

- **Window:** one step per STRK price point at or after the first snapshot (2025-11-13 19:42:42 UTC). That gives **337 steps, 2025-11-14 00:00 → 2026-10-15 08:10:40 UTC**. Each step accrues 1/365 of the annual amount.
- **Price alignment:** the BTC price is keyed by UTC day (`t // 86400`), and the later point of a day wins.
- **Between snapshots:** on-chain values are linearly interpolated in time and clamped at the ends.
- **Emission:** `M = (strk_rw + btc_rw) × 8760` per year. The STRK staker reward is fixed at `0.75·M`. The BTC reward is `α/(1−α) · 0.75·M`; at α = 25% this equals the status quo. Under a lower α, the excess is not minted.
- **Target:** `α* = k·V / (S + k·V)`, where `V = B / r` is the BTC stake valued in STRK and `r = P_STRK / P_BTC`. It sets BTC APR = k × STRK APR.
- **Original rule:** `r` is the 7-day mean of the daily ratio, `α = clamp(α*, 10%, 33%)`, recomputed independently each step.
- **Revised rule:** start at **α = 0.25**, use the spot ratio, and limit each 1-hour epoch to a step of δ = 0.5%. On daily data the target is held constant within the day, so the 24 epoch steps compound into one clamp `[α/1.005^24, α·1.005^24]`, followed by `min(α, 33%)`.
- **Forward scenarios:** constants from the 2026-10-16 snapshot. STRK price and BTC stake follow geometric paths over 365 days, and STRK stake is held constant.

## Errata vs the forum posts

| Published | This script |
|---|---|
| fixed α: 33.6M STRK | 33.5M |
| floating: +8% STRK, $2.02M | +9%, $2.01M |
| k = 0.6: +28% USD | +29% USD (+28% in BTC) |
| "7-day TWAP + step cap peaks at 7.9%" | 7.89% is with a **30-day** TWAP; with a 7-day TWAP the peak is 5.33% (spot + step cap: 4.70%) |
| original post: "α today ~13–14%, ~139M/yr" | spot values on the 2026-10-10 inputs, not the last backtest step |

## Limitations

- **Modelled, not realized.** These are modelled minted rewards, not realized or claimed rewards: attestation, commission and claim timing are not included.
- **Interpolated stake.** Stake is interpolated between snapshots 13–30 days apart, so short-lived inflows are smoothed.
- **Daily prices.** Intra-day moves are not captured, so true peak APRs under a fixed α were likely higher.
- **In-sample k.** k = 0.4 is the past average BTC/STRK APR ratio, so budget neutrality over the same window is partly by construction. The forward scenarios are the out-of-sample case.
