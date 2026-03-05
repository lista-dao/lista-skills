> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Shared Computation

Referenced by Reports A, D, E for position discovery, price fetching, and metric computation.

## Step 1 — Fetch position data (MCP)

Call two MCP tools:

```
lista_get_position({ wallet: "<address>" })
```

Returns three sections:

- **holdings.objs[]** — active markets with metadata:
  - `marketId`, `collateralSymbol`, `loanSymbol`, `collateralPrice`, `loanPrice` (USD), `zone`, `termType`
- **collaterals[]** — per-market collateral:
  - `address` (marketId), `amount` (human-readable), `usdValue`
- **borrows[]** — per-market debt (pre-computed, not raw shares):
  - `address` (marketId), `assetSymbol`, `collateralSymbol`, `amount` (human-readable), `usdValue`

Then fetch LLTV for each active market. Markets are paginated (max 50/page), so paginate until all needed IDs are found:

```
lista_get_borrow_markets({ pageSize: 50, page: 1 })
lista_get_borrow_markets({ pageSize: 50, page: 2 })   # if needed
lista_get_borrow_markets({ pageSize: 50, page: 3 })   # if needed
```

Each market has `id` and `lltv` (decimal string, e.g. "0.860000000000000000"). Match by `id` to the user's active market IDs from holdings. Stop paginating once all active market LLTVs are found.

Smart Lending detection: `collateralSymbol` contains `&` (e.g. "slisBNB & BNB"). Label as `slisBNB/BNB LP` in output.

## Step 2 — Metric computation

All amounts from MCP are human-readable (not raw 1e18). No precision conversion needed.

For each market, join data by `marketId`:

```
collateralAmount   = collaterals[].amount
collateralUSD      = collaterals[].usdValue
debtAmount         = borrows[].amount          (0 if no borrow entry)
debtUSD            = borrows[].usdValue        (0 if no borrow entry)
collateralPrice    = holdings[].collateralPrice
loanPrice          = holdings[].loanPrice
lltv               = borrow_markets[].lltv     (match by market id)

netEquityUSD       = collateralUSD − debtUSD

LTV                = debtUSD / collateralUSD
healthFactor       = lltv / LTV                       (when LTV > 0)
liqPriceUSD        = debtUSD / (collateralAmount × lltv)
buffer             = (collateralPrice − liqPriceUSD) / collateralPrice
```

## Correlated asset pairs

Asset families:
- USD stable: USD1, U, USDT, USDC, DAI, FDUSD, BUSD
- BNB / BNB-LST: BNB, WBNB, slisBNB, wstBNB, ankrBNB, BNBx
- ETH / ETH-LST: ETH, WETH, wstETH, stETH, rETH
- BTC: BTCB, WBTC, BTC

A position is **correlated** when collateral and loan are in the same family. For LP: both component tokens AND loan must be in the same family.

## Risk level assignment

**Uncorrelated (standard):**
- 🟢 SAFE — LTV / lltv < 80%
- 🟡 WARNING — 80% ≤ LTV / lltv < 90%
- 🔴 DANGER — LTV / lltv ≥ 90%

**Correlated (adjusted):**
- 🟢 SAFE — LTV / lltv < 92%
- 🟡 WARNING — 92% ≤ LTV / lltv < 97%
- 🔴 DANGER — LTV / lltv ≥ 97%

Append "(correlated)" / "(相關對)" for correlated positions.

**Health factor display:** HF >= 1.2 → ✅, HF < 1.2 → ⚠️

**Dust filter:** Skip positions where both collateralUSD < $1 AND debtUSD < $1.

**Supply-only:** Skip debt, LTV, and liquidation rows.
