> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Shared Computation

Referenced by Reports A, D, E for price fetching and metric computation.

## Detect collateral type

```bash
curl -s "https://api.lista.org/api/moolah/market/<marketId>" \
  | python3 -c "
import sys, json
d = json.load(sys.stdin)['data']
cfg = d.get('smartCollateralConfig') or {}
print('LP' if cfg.get('swapPool') else 'ERC20')
print(d.get('loanTokenPrice', ''))
"
```

## ERC20 collateral — price fetching

**Primary — oracle-price:**
```bash
node skills/scripts/moolah.js oracle-price <marketId>
# Returns: { price (1e36-scaled), lltv, lltvPct }
```

**Fallback — token-price (when oracle-price reverts):**
```bash
node skills/scripts/moolah.js params <marketId>
# Returns: { collateralToken, loanToken, lltv, lltvPct }

node skills/scripts/moolah.js token-price <collateralToken>
# Returns: { priceUSD }
```

## LP collateral (Smart Lending) — price fetching

`oracle-price` always reverts on Smart Lending markets.

```bash
node skills/scripts/moolah.js lp-price <marketId>
# Returns: { lpTokenPriceUSD, token0Symbol, token1Symbol, virtualPriceF, coin0PriceUSD }

node skills/scripts/moolah.js params <marketId>
# Returns: { lltv, lltvPct }
```

Label collateral as `<token0>/<token1> LP`.

## Metric computation

**Precision rule:** All on-chain ERC20 and LP token quantities (collateral, currentDebt, supplyShares, borrowShares, lltv) are raw 1e18 integers. Always divide by 1e18 before display or USD conversion. Oracle prices from `oracle-price` are 1e36-scaled.

### Path A — ERC20 with oracle-price

```
collateral_f       = collateral / 1e18
currentDebt_f      = currentDebt / 1e18
oraclePrice_f      = oraclePrice / 1e36
loanTokenUSD       = loanTokenPrice (from API)
lltvF              = lltv / 1e18

collateral_in_loan = collateral_f × oraclePrice_f
collateralPriceUSD = oraclePrice_f × loanTokenUSD
collateralUSD      = collateral_f × collateralPriceUSD
debtUSD            = currentDebt_f × loanTokenUSD
netEquityUSD       = collateralUSD − debtUSD

LTV                = currentDebt_f / collateral_in_loan
healthFactor       = lltvF / LTV                     (when LTV > 0)
liqPriceUSD        = debtUSD / (collateral_f × lltvF)
buffer             = (collateralPriceUSD − liqPriceUSD) / collateralPriceUSD
```

### Path B — LP collateral

```
collateral_f      = collateral / 1e18
currentDebt_f     = currentDebt / 1e18
lpTokenPriceUSD   = from lp-price result
loanTokenUSD      = loanTokenPrice (from API)
lltvF             = lltv / 1e18

collateralUSD     = collateral_f × lpTokenPriceUSD
debtUSD           = currentDebt_f × loanTokenUSD
netEquityUSD      = collateralUSD − debtUSD

LTV               = debtUSD / collateralUSD
healthFactor      = lltvF / LTV                  (when LTV > 0)
liqPriceUSD       = debtUSD / (collateral_f × lltvF)
buffer            = (lpTokenPriceUSD − liqPriceUSD) / lpTokenPriceUSD
```

### Path C — ERC20 with token-price fallback

```
collateral_f       = collateral / 1e18
currentDebt_f      = currentDebt / 1e18
collateralPriceUSD = priceUSD (from token-price)
loanTokenUSD       = loanTokenPrice (from API)
lltvF              = lltv / 1e18

collateralUSD      = collateral_f × collateralPriceUSD
debtUSD            = currentDebt_f × loanTokenUSD
netEquityUSD       = collateralUSD − debtUSD

LTV                = debtUSD / collateralUSD
healthFactor       = lltvF / LTV                       (when LTV > 0)
liqPriceUSD        = debtUSD / (collateral_f × lltvF)
buffer             = (collateralPriceUSD − liqPriceUSD) / collateralPriceUSD
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
