# Borrowing-Asset Solver — Desk Simulation

Two settlement strategies for a Base-chain swap desk, replayed against the same real trade tape
and the same starting capital, so the results compare directly:

- **AaveSolver** (`aave-solver.html`) — every trade settles by borrowing from / repaying an Aave
  position on Base, hedged with a mirrored trade on KalqiX.
- **LiquiditySolver** (`liquidity-solver.html`) — every trade settles straight out of on-chain
  inventory, mirrored across Base and KalqiX. No lending market, no interest, plain transfers.

`index.html` is a one-page landing view with a switcher for **AaveSolver**, **LiquiditySolver**,
or **Both** (stacked, for side-by-side comparison).

## Running it

These are static HTML/JS files with no build step, but they load `data/trades.csv` via `fetch()`,
which browsers block on the `file://` scheme — serve the folder over HTTP:

```bash
git clone https://github.com/qkslvr/borrowingAssetSolver.git
cd borrowingAssetSolver
python3 -m http.server 8000
# or: npx serve .
```

Then open `http://localhost:8000/` (the landing page), or go straight to
`http://localhost:8000/aave-solver.html` / `liquidity-solver.html` for a single dashboard.

To publish it, push this repo and enable **GitHub Pages** (Settings → Pages → deploy from the
`main` branch, root folder) — everything here is static, no server-side code required.

## Data

`data/trades.csv` is a snapshot of the `trades` table (27,319 rows, 2026-06-02 through
2026-07-03), pre-filtered to positive volume/price and encoded compactly, one row per trade:

```
<unix_ts>,<chain_code>,<asset_group>,<side>,<volume_usd>,<price_usd>
```

- `chain_code`: `b`=base, `e`=ethereum, `a`=arbitrum, `o`=optimism, `s`=bsc, `p`=polygon
- `asset_group`: `E`=ETH/WETH/wstETH, `B`=WBTC/cbBTC
- `side`: `1`=BUY, `0`=SELL

Both dashboards load this same file, apply the same chain/size/date filters client-side, and run
their own settlement model over it — so any difference between the two views is the settlement
model, not the data. Replace the file with a fresher export (same columns, same encoding) to
update both dashboards at once; nothing else needs to change.

## What's editable

Every dashboard has a **Parameters** panel (top right) for capital, fees, settlement/rebalancing
costs, drift and utilization triggers, and (AaveSolver only) Aave supply/borrow rates — all
recompute instantly against the loaded tape, no rebuild needed.
