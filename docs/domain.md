# Lista Lending — Domain Logic Reference

Business logic definitions for Lista Lending (Moolah) on BSC.

---

## Core Metrics

### Loan-to-Value (LTV)

```
LTV = debtUSD / collateralUSD
```

- Expressed as a percentage (e.g. 0.45 = 45%)
- Higher LTV = more leverage = higher risk
- LTV = 0 for supply-only positions (no borrow)

### Liquidation Loan-to-Value (LLTV)

The maximum LTV before a position becomes eligible for liquidation. Set per market.

- Retrieved from on-chain `params` (raw 1e18 integer) or API `lltv` field
- Typical range: 70%–96% depending on asset pair correlation

### LTV Gap

```
LTV gap = LLTV - current LTV
```

- Measures distance from liquidation threshold
- Smaller gap = closer to liquidation
- Used for alert threshold logic

### Health Factor (HF)

```
HF = LLTV / LTV
```

- HF > 1: position is above water (safe)
- HF = 1: position is at liquidation threshold
- HF < 1: position is underwater (should not occur — liquidation triggers at HF = 1)

Display indicators:
- HF >= 1.2 → ✅
- HF < 1.2 → ⚠️ (yellow warning in reports)

### Liquidation Price

The collateral price at which LTV reaches LLTV:

```
liqPriceUSD = debtUSD / (collateral_f × LLTV)
```

- For ERC20 collateral: price per 1 unit of collateral token
- For LP collateral: price per 1 LP token

### Buffer

```
buffer = (currentPrice - liqPriceUSD) / currentPrice × 100%
```

- Percentage price drop required to trigger liquidation
- Higher buffer = safer position

---

## Price Sources

### ERC20 Collateral — Primary (oracle-price)

Morpho oracle price scaled to 1e36:

```
collateralPriceUSD = (oraclePrice / 1e36) × loanTokenPriceUSD
```

### ERC20 Collateral — Fallback (token-price)

When `oracle-price` reverts, use the Lista oracle directly:

```bash
node skills/scripts/moolah.js token-price <collateralTokenAddress>
```

Returns USD price with 8 decimal places.

### LP Collateral — Smart Lending (lp-price)

For markets where `smartCollateralConfig != null`:

```
lpPriceUSD = (virtualPrice / 1e18) × (coin0PriceUSD)
```

- `virtualPrice`: Curve StableSwap `get_virtual_price()` (1e18-scaled)
- `coin0PriceUSD`: First coin in pool, priced via Lista oracle

`oracle-price` always reverts on Smart Lending markets — use `lp-price` exclusively.

---

## Correlated Asset Pairs

When collateral and loan token belong to the same asset family, both move together. Liquidation risk comes from peg/ratio deviation, not broad market moves.

### Asset Families

| Family | Tokens |
|---|---|
| USD stable | USD1, U, USDT, USDC, DAI, FDUSD, BUSD |
| BNB / BNB-LST | BNB, WBNB, slisBNB, wstBNB, ankrBNB, BNBx |
| ETH / ETH-LST | ETH, WETH, wstETH, stETH, rETH |
| BTC | BTCB, WBTC, BTC |

### Correlation Rules

- A position is **correlated** when collateral and loan token are in the same family
- For LP collateral: both LP component tokens AND the loan token must all be in the same family
- Example: slisBNB/BNB LP + BNB loan → correlated (all BNB family)
- Example: slisBNB/BNB LP + USD1 loan → uncorrelated (different families)

### Risk Thresholds

**Uncorrelated pairs (standard):**

| Risk Level | Condition |
|---|---|
| 🟢 SAFE | LTV / LLTV < 80% |
| 🟡 WARNING | 80% <= LTV / LLTV < 90% |
| 🔴 DANGER | LTV / LLTV >= 90% |

**Correlated pairs (adjusted):**

| Risk Level | Condition |
|---|---|
| 🟢 SAFE | LTV / LLTV < 92% |
| 🟡 WARNING | 92% <= LTV / LLTV < 97% |
| 🔴 DANGER | LTV / LLTV >= 97% |

---

## Vault Zones

| Zone | Name | Description |
|---|---|---|
| 0 | Classic | Audited vaults with established track record |
| 1 | Alpha | Higher risk / higher reward, newer strategies |
| 4 | Aster | Partner-originated assets |

---

## Position Types

### Supply-only Position
- `supplyShares > 0`, `borrowShares = 0`
- No debt, LTV, or liquidation price applicable
- Earns supply APY + optional LISTA emission

### Borrow Position
- `borrowShares > 0`, `collateral > 0`
- Subject to LTV monitoring and potential liquidation
- Current debt: `borrowShares × totalBorrowAssets / totalBorrowShares` (from on-chain market state)

### Dust Position
- Both `collateralUSD < $1` AND `debtUSD < $1`
- Skipped in reports (not worth displaying)
