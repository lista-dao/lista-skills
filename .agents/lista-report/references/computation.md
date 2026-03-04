> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Shared Computation

Referenced by Reports A, D, E for position discovery, price fetching, and metric computation.

## Step 1 — Discover active markets (holding API)

```bash
curl -s "https://api.lista.org/api/moolah/one/holding?userAddress=<address>&type=market"
```

Response: `{ code, msg, data: { objs: [...] } }`. Each entry provides:
- `marketId` — use for on-chain position queries
- `collateralSymbol`, `loanSymbol` — display labels
- `collateralPrice`, `loanPrice` — USD prices (decimal strings, full precision)
- `collateralToken`, `loanToken` — contract addresses
- `zone` — 0=Classic, 1=Alpha, 3=Smart Lending, 4=Aster

This replaces vault scanning. Only markets where the user has activity are returned.

Smart Lending detection: `collateralSymbol` contains `&` (e.g. "slisBNB & BNB"). Label as `slisBNB/BNB LP` in output.

## Step 2 — Fetch on-chain position data

For each active market, fetch the raw position:

```bash
node skills/scripts/moolah.js position <marketId> <address>
# Returns: { supplyShares, borrowShares, collateral }
```

For markets with `borrowShares > 0`, compute current debt:

```bash
node skills/scripts/moolah.js market <marketId>
# Returns: { totalBorrowAssets, totalBorrowShares }
# currentDebt = borrowShares × totalBorrowAssets / totalBorrowShares
```

Fetch LLTV:

```bash
node skills/scripts/moolah.js params <marketId>
# Returns: { lltv, lltvPct }
```

These three calls can be batched per market. If multiple markets are active, run them in parallel.

## Step 3 — Metric computation

**Precision rule:** All on-chain ERC20 and LP token quantities (collateral, currentDebt, supplyShares, borrowShares, lltv) are raw 1e18 integers. Always divide by 1e18 before display or USD conversion.

Prices come from the holding API (Step 1). No separate price-fetching calls needed.

```
collateral_f       = collateral / 1e18
currentDebt_f      = currentDebt / 1e18
collateralPriceUSD = collateralPrice (from holding API)
loanPriceUSD       = loanPrice (from holding API)
lltvF              = lltv / 1e18

collateralUSD      = collateral_f × collateralPriceUSD
debtUSD            = currentDebt_f × loanPriceUSD
netEquityUSD       = collateralUSD − debtUSD

LTV                = debtUSD / collateralUSD
healthFactor       = lltvF / LTV                       (when LTV > 0)
liqPriceUSD        = debtUSD / (collateral_f × lltvF)
buffer             = (collateralPriceUSD − liqPriceUSD) / collateralPriceUSD
```

This single path works for both ERC20 and LP collateral — the holding API provides the correct USD price for both types.

## Correlated asset pairs

Asset families:
- USD stable: USD1, U, USDT, USDC, DAI, FDUSD, BUSD
- BNB / BNB-LST: BNB, WBNB, slisBNB, wstBNB, ankrBNB, BNBx
- ETH / ETH-LST: ETH, WETH, wstETH, stETH, rETH
- BTC: BTCB, WBTC, BTC

A position is **correlated** when collateral and loan are in the same family. For LP: both component tokens AND loan must be in the same family.

## Risk level assignment

**Uncorrelated (standard):**
- 🟢 SAFE — LTV / lltvF < 80%
- 🟡 WARNING — 80% ≤ LTV / lltvF < 90%
- 🔴 DANGER — LTV / lltvF ≥ 90%

**Correlated (adjusted):**
- 🟢 SAFE — LTV / lltvF < 92%
- 🟡 WARNING — 92% ≤ LTV / lltvF < 97%
- 🔴 DANGER — LTV / lltvF ≥ 97%

Append "(correlated)" / "(相關對)" for correlated positions.

**Health factor display:** HF >= 1.2 → ✅, HF < 1.2 → ⚠️

**Dust filter:** Skip positions where both collateralUSD < $1 AND debtUSD < $1.

**Supply-only:** Skip debt, LTV, and liquidation rows.
