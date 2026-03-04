> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Report A — Position Report

Generates a full position report with collateral, debt, health factor, LTV, liquidation price, and strategy recommendations.

## A.1 — Fetch positions

Run once per address:

```bash
node skills/scripts/moolah.js user-positions <address>
```

Returns JSON with `positions[]`. Each entry has: `marketId`, `collateralSymbol`, `loanSymbol`, `collateral`, `borrowShares`, `supplyShares`, `currentDebt`, `lastUpdateIso`.

If `positions` is empty → "No active positions."

## A.2 — Fetch prices and compute metrics

For each position, follow `references/computation.md` to get prices and compute: collateralUSD, debtUSD, netEquityUSD, LTV, healthFactor, liqPriceUSD, buffer, riskLevel.

## A.3 — Recommendations

Generate 1–3 concise suggestions per address based on actual numbers:

**Risk reduction (uncorrelated):**
- LTV/LLTV >= 90% (DANGER): Repay debt or add collateral. Show amounts to reach 75% LTV/LLTV.
- LTV/LLTV 80–90% (WARNING): Partial repayment or top-up. Amounts to reach 70% LTV/LLTV.

**Risk reduction (correlated):**
- LTV/LLTV >= 97% (DANGER): Immediate debt reduction. Amounts to reach 85% LTV/LLTV.
- LTV/LLTV 92–97% (WARNING): Monitor collateral/loan price ratio. Real risk = depeg event.

**Yield enhancement (low LTV):**
- LTV/LLTV < 50% (uncorrelated) or < 75% (correlated): Suggest leveraging via /lista-loop.
- Supply-only: Mention borrowing to amplify yield.

## A.4 — Output template

### English

```
Lista Lending — Position Report
Generated: <YYYY-MM-DD HH:MM> UTC  |  BSC Mainnet
━━━━━━━━━━━━━━━━━━━━━━━━━

Address 1: 0xAbCd…5678
────────────────────────
#1  BTCB / U  🟢 SAFE
  Collateral:     398.85 BTCB  (~＄38,250,000)
  Debt:           18,020,988.00 U  (~＄18,020,988)
  Net equity:                       ~＄20,229,012
  Health factor:  1.83 ✅
  LTV:            47.1%  /  LLTV 86.0%
  Liq. price:     BTCB < ＄52,500  (current: ＄96,000)
  Last accrual:   2026-03-01 03:12 UTC

- - - - -

#2  slisBNB/BNB LP / BNB  🟢 SAFE (correlated)
  Collateral:     120.00 slisBNB/BNB LP  (~＄78,143)
  Debt:           50.00 BNB  (~＄34,550)
  Net equity:                  ~＄43,593
  Health factor:  1.95 ✅
  LP price:       ＄651.19/LP
  LTV:            44.2%  /  LLTV 86.0%
  Liq. price:     LP < ＄335  (current: ＄651.19)
  Last accrual:   2026-03-01 03:12 UTC

Address 1 summary: 2 active positions  |  Net equity ~＄20.2M

Recommendations:
  1. LTV is comfortable — consider /lista-loop to amplify yield.

━━━━━━━━━━━━━━━━━━━━━━━━━

Total: <N> addresses  |  <M> active positions  |  Combined net equity ~＄X

Data: api.lista.org  |  BSC Mainnet
```

### 繁體中文

```
Lista Lending — 持倉報告
產生時間：<YYYY-MM-DD HH:MM> UTC  |  BSC 主網
━━━━━━━━━━━━━━━━━━━━━━━━━

地址 1：0xAbCd…5678
────────────────────────
#1  BTCB / U  🟢 安全
  抵押品：    398.85 BTCB（約 ＄38,250,000）
  負債：      18,020,988.00 U（約 ＄18,020,988）
  淨資產：                      約 ＄20,229,012
  健康係數：  1.83 ✅
  LTV：      47.1%  /  清算線 86.0%
  清算價格：  BTCB < ＄52,500（當前價：＄96,000）
  最後結算：  2026-03-01 03:12 UTC

- - - - -

#2  slisBNB/BNB LP / BNB  🟢 安全（相關對）
  抵押品：    120.00 slisBNB/BNB LP（約 ＄78,143）
  負債：      50.00 BNB（約 ＄34,550）
  淨資產：                約 ＄43,593
  健康係數：  1.95 ✅
  LP 價格：   ＄651.19/LP
  LTV：      44.2%  /  清算線 86.0%
  清算價格：  LP < ＄335（當前價：＄651.19）
  最後結算：  2026-03-01 03:12 UTC

地址 1 小結：2 個活躍持倉  |  淨資產約 ＄20.2M

持倉建議：
  1. LTV 尚在安全範圍，可考慮使用 /lista-loop 提高槓桿收益。

━━━━━━━━━━━━━━━━━━━━━━━━━

總計：<N> 個地址  |  <M> 個活躍持倉  |  合計淨資產約 ＄X

資料來源：api.lista.org  |  BSC 主網
```
