---
name: lista-report
description: Lista Lending report hub — position status, market overview, yield scan, liquidation risk check, and daily digest on BSC
---

# Lista Lending — Report Hub

Generate reports for Lista Lending (Moolah) on BSC. Choose a report type below.

**RPC script:** `skills/scripts/moolah.js` (Node.js stdlib, no packages needed)
**API base:** `https://api.lista.org/api/moolah`

---

## Step 0 — Ask for language

Do NOT run any commands until the user has answered this question:

> Which language should I use for the output?
> 請問輸出以哪種語言生成？
>   1) English
>   2) 简体中文
>   3) 繁體中文
>   4) Other (specify)

**Language handling rules:**
- **1 / English** — use the English template exactly.
- **2 / 简体中文** — use the 繁體中文 template, then convert all Traditional Chinese characters to Simplified Chinese. Do NOT alter numbers, symbols, separators, or field layout.
- **3 / 繁體中文** — use the 繁體中文 template exactly.
- **4 / Other** — translate all label text into the user's language. Use natural, idiomatic phrasing (not word-for-word). Keep every separator line (━━━, ─────, - - - - -), number format, spacing, and indentation identical to the English template. Do NOT add bullet points, reformat rows, or change the structural layout.

Remember the answer and use it for all output below.

---

## FORMAT ENFORCEMENT — applies to every report below

**You MUST follow these rules strictly for ALL output. No exceptions.**

1. **Plain text only.** No markdown bold (`**`), italics (`_`), headers (`#`), or links. Output is intended for Telegram/Discord paste.
2. **Copy template structure character-for-character.** Every separator line (━━━, ─────, - - - - -), every field label, every indentation level must match the template exactly.
3. **Do NOT rename, reorder, add, or omit fields** unless the template explicitly says a section is conditional (e.g. "if risk alerts exist").
4. **＄ in templates = $ in output.** Templates use fullwidth ＄ for escaping; replace with normal $ when generating.
5. **Numbers stay as-is.** Do not round, reformat, or change decimal places beyond what the data provides.
6. **Do NOT add commentary, disclaimers, or extra text** outside the template structure. The report IS the output.

---

## Step 1 — Report type

Ask the user (or infer from their request):

> Which report would you like?
> 你需要哪種報告？
>   1) Position Report — collateral, debt, health factor, LTV, liquidation price
>   2) Market Overview — TVL, lending rates, top vaults
>   3) Yield Opportunities — best deposit APY across vaults
>   4) Risk Check — liquidation risk alerts with thresholds
>   5) Daily Digest — positions + yield + market snapshot

If the user's original message already implies a type (e.g. "check my positions" → 1, "what rates are available" → 2, "best yield" → 3, "am I safe" → 4, "daily report" → 5), skip the question and proceed directly.

---

## Step 2 — Wallet address (for types 1, 4, 5)

Reports 1, 4, 5 require a wallet address. Reports 2, 3 do not — skip this step for them.

### Load saved address

```bash
cat ~/.lista/wallet.txt 2>/dev/null
```

If the file exists and contains a valid address, use it. Inform the user:

> **EN:** Using saved wallet: 0xAbCd...5678. Say "change address" to update.
> **中文：** 使用已儲存的錢包：0xAbCd...5678。輸入「換個地址」可更新。

### Ask if no saved address

> **EN:** What is your wallet address? I will save it locally so you don't need to enter it again.
> **中文：** 請問你的錢包地址是什麼？我會儲存到本地，下次不用再輸入。

### Save address

```bash
mkdir -p ~/.lista && echo "<ADDRESS>" > ~/.lista/wallet.txt
```

### Change address

When the user says "change address" / "换个地址" / "換個地址", ask for the new address and save it:

```bash
echo "<NEW_ADDRESS>" > ~/.lista/wallet.txt
```

### Multiple addresses

The user may provide multiple addresses (comma/space/line separated). Save all to the file (one per line) and process each.

---

# REPORT A — Position Report

Generates a full position report with collateral, debt, health factor, LTV, liquidation price, and strategy recommendations.

## A.1 — Fetch positions

Run once per address:

```bash
node skills/scripts/moolah.js user-positions <address>
```

Returns JSON with `positions[]`. Each entry has: `marketId`, `collateralSymbol`, `loanSymbol`, `collateral`, `borrowShares`, `supplyShares`, `currentDebt`, `lastUpdateIso`.

If `positions` is empty → "No active positions."

## A.2 — Fetch prices and compute metrics

For each position, follow **APPENDIX: Shared Computation** to get prices and compute: collateralUSD, debtUSD, netEquityUSD, LTV, healthFactor, liqPriceUSD, buffer, riskLevel.

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

---

# REPORT B — Market Overview

Protocol-wide stats and per-market lending rates.

## B.1 — Fetch data

```bash
# All vaults
curl -s "https://api.lista.org/api/moolah/vault/list?pageSize=100"

# Allocations for top vaults
curl -s "https://api.lista.org/api/moolah/vault/allocation?address=<VAULT>&pageSize=100"
```

API shape: `response.data.list`. APY values are decimals (0.087 = 8.7%).

## B.2 — Compute

- **Total TVL** = sum of `depositsUsd`
- **Est. Borrows** = sum of `depositsUsd × utilization`
- **Overall Utilization** = Est. Borrows / Total TVL
- Group by zone: 0=Classic, 1=Alpha, 4=Aster
- Per-market: `supplyApy`, `borrowRate`, `liquidity`
- High-utilization: `utilization > 0.85`
- Near-cap: `totalSupply / cap > 0.90`
- Smart Lending: `smartCollateralConfig != null`
- Fixed Rate: `termType == "fixed"`

If user asks about a specific asset (e.g. "USDT rate"), filter to markets with that asset.

## B.3 — Output template

### English

```
📊 Lista Lending — Market Overview
<DATE> UTC
━━━━━━━━━━━━━━━━━━━━━━━━━

🌐 Protocol Overview
   Total TVL:        ＄42.1M
   Est. Borrows:     ＄18.9M  |  Overall Util: 44.9%
   Active Vaults:    12 Classic  |  4 Alpha  |  2 Aster

- - - - -

💰 Top Vaults by TVL
1. WBNB Vault  (WBNB)
   TVL: ＄18.2M  |  APY: 4.2% + 2.1% LISTA = 6.3%  |  Util: 52%

- - - - -

🔥 High-Utilization Markets (>85%)
   slisBNB/WBNB — 92%  |  Borrow rate: 8.4%  [rate rising]

⚠️  Near Supply Cap
   PT-slisBNBx/WBNB — 94% of cap used  (＄240K remaining)

📋 Lending Rates by Market
   slisBNB/WBNB  — Supply: 3.2%  |  Borrow: 5.8%  |  Liquidity: ＄4.1M
   BTCB/U        — Supply: 2.8%  |  Borrow: 4.6%  |  Liquidity: ＄12.3M
   WBNB/USD1     — Supply: 4.1%  |  Borrow: 7.2%  |  Liquidity: ＄2.8M

⚡ Smart Lending  |  🔒 Fixed Rate
   slisBNB/WBNB — DEX fees active
   PT-slisBNBx/WBNB — 5.8% fixed

━━━━━━━━━━━━━━━━━━━━━━━━━

💡 <1–2 insight sentences>

Data: api.lista.org  |  BSC Mainnet
```

### 繁體中文

```
📊 Lista Lending — 市場總覽
<DATE> UTC
━━━━━━━━━━━━━━━━━━━━━━━━━

🌐 協議總覽
   總 TVL：      ＄42.1M
   預估借款：    ＄18.9M  |  整體利用率：44.9%
   活躍金庫：    12 Classic  |  4 Alpha  |  2 Aster

- - - - -

💰 TVL 最高金庫
1. WBNB 金庫（WBNB）
   TVL：＄18.2M  |  年化：4.2% + 2.1% LISTA = 6.3%  |  利用率：52%

- - - - -

🔥 高利用率市場（>85%）
   slisBNB/WBNB — 92%  |  借款利率：8.4%  [利率上升中]

⚠️  接近供給上限
   PT-slisBNBx/WBNB — 已用 94%（剩餘 ＄240K）

📋 各市場借貸利率
   slisBNB/WBNB  — 供款：3.2%  |  借款：5.8%  |  流動性：＄4.1M
   BTCB/U        — 供款：2.8%  |  借款：4.6%  |  流動性：＄12.3M
   WBNB/USD1     — 供款：4.1%  |  借款：7.2%  |  流動性：＄2.8M

⚡ Smart Lending  |  🔒 固定利率
   slisBNB/WBNB — DEX 手續費啟用
   PT-slisBNBx/WBNB — 5.8% 固定

━━━━━━━━━━━━━━━━━━━━━━━━━

💡 <1-2 句市場見解>

資料來源：api.lista.org  |  BSC 主網
```

---

# REPORT C — Yield Opportunities

Scan vaults and surface top deposit APY.

## C.1 — Fetch and sort

```bash
curl -s "https://api.lista.org/api/moolah/vault/list?pageSize=100"
```

- Filter by `assetSymbol` if user specified an asset
- `totalApy = apy + (emissionApy if emissionEnabled else 0)`
- Sort by `totalApy` descending within each zone

For top 5 vaults, fetch allocations:

```bash
curl -s "https://api.lista.org/api/moolah/vault/allocation?address=<VAULT>&pageSize=100"
```

Identify Smart Lending (`smartCollateralConfig != null`) and Fixed Rate (`termType == "fixed"`) markets.

## C.2 — Output template

### English

```
Lista Lending — Top Yield Opportunities
━━━━━━━━━━━━━━━━━━━━━━━━━
<YYYY-MM-DD> UTC  |  BSC Mainnet

🏆 Classic Zone (Audited)

🥇 WBNB Vault
   APY: 4.2% base  +  2.1% LISTA  =  6.3% total
   TVL: ＄42.1M  |  Utilization: 52%
   Top markets: slisBNB/WBNB 39%,  PT-slisBNBx/WBNB 21%

- - - - -

🥈 USD1 Vault
   APY: 3.1% base  +  1.8% LISTA  =  4.9% total
   TVL: ＄18.3M  |  Utilization: 61%
   Top markets: BTCB/USD1 44%

⚡ Smart Lending — slisBNB/WBNB earns extra ~1.2% from DEX fees
📌 Fixed Rate   — PT-slisBNBx at 5.8% fixed

- - - - -

⚠️  Alpha Zone (Higher Risk)
   WBTC/USD1 — 14.2%  |  Emerging market, less liquidity

🤝 Aster Zone (Partner Assets)
   <vault name> — <APY>%  |  Partner-originated

━━━━━━━━━━━━━━━━━━━━━━━━━

💡 High utilization (>85%) = rates may rise. Smart Lending earns DEX trading fees.

Data: api.lista.org  |  BSC Mainnet
```

### 繁體中文

```
Lista Lending — 最佳存款收益
━━━━━━━━━━━━━━━━━━━━━━━━━
<YYYY-MM-DD> UTC  |  BSC 主網

🏆 Classic 區（已審計）

🥇 WBNB 金庫
   年化：4.2% 基礎  +  2.1% LISTA  =  6.3% 總計
   TVL：＄42.1M  |  利用率：52%
   主要市場：slisBNB/WBNB 39%，PT-slisBNBx/WBNB 21%

- - - - -

🥈 USD1 金庫
   年化：3.1% 基礎  +  1.8% LISTA  =  4.9% 總計
   TVL：＄18.3M  |  利用率：61%
   主要市場：BTCB/USD1 44%

⚡ Smart Lending — slisBNB/WBNB 額外賺取約 1.2% DEX 手續費
📌 固定利率   — PT-slisBNBx 5.8% 固定

- - - - -

⚠️  Alpha 區（較高風險）
   WBTC/USD1 — 14.2%  |  新興市場，流動性較低

🤝 Aster 區（合作夥伴資產）
   <金庫名稱> — <APY>%  |  合作方發起

━━━━━━━━━━━━━━━━━━━━━━━━━

💡 高利用率（>85%）代表利率可能上升。Smart Lending 可賺取 DEX 交易手續費。

資料來源：api.lista.org  |  BSC 主網
```

---

# REPORT D — Risk Check

Check all positions for liquidation risk and flag those approaching thresholds.

## D.1 — Fetch positions and compute metrics

Same as Report A steps A.1–A.2. Use **APPENDIX: Shared Computation** for price fetching and metric computation.

## D.2 — Apply alert thresholds

For each position with `borrowShares > 0`:

```
ltvGap = lltvF - LTV    # decimal; multiply by 100 for display %

if lltvF >= 0.90:
    defaultThreshold = 0.005   # 0.5%
else:
    defaultThreshold = 0.05    # 5%

threshold = userCustomThreshold or defaultThreshold
isAlert = ltvGap <= threshold
```

## D.3 — Output template

Show alert-flagged positions first, then safe positions in compact format.

### English — alert triggered

```
🚨 Lista Liquidation Alert
━━━━━━━━━━━━━━━━━━━━━━━━━
Wallet: 0xAbCd...5678

#1  slisBNB / WBNB
─────────────────────
Collateral:   2.50 slisBNB  (~＄1,450)
Debt:         1.89 WBNB  (~＄1,300)
Health factor: 1.004 🔴
Current LTV:  89.6%  /  LLTV: 90.0%
LTV gap:      0.4% — below threshold (0.5%)

Liq. price:   ＄532.40 / slisBNB  (current: ＄580.00)
Distance:     ＄47.60  (8.2%)

Action: Add collateral or repay debt immediately.
        Repay ~0.15 WBNB to bring LTV to 80%.
━━━━━━━━━━━━━━━━━━━━━━━━━
```

### English — all clear

```
✅ Lista Position Check — All Clear
━━━━━━━━━━━━━━━━━━━━━━━━━
Wallet: 0xAbCd...5678
<YYYY-MM-DD HH:MM> UTC

No positions approaching liquidation.

#1  BTCB / U  |  HF: 1.83 ✅  |  LTV: 47.1% / 86.0%  |  gap: 38.9%
#2  ETH / USD1  |  HF: 2.14 ✅  |  LTV: 37.4% / 80.0%  |  gap: 42.6%

━━━━━━━━━━━━━━━━━━━━━━━━━
Threshold: LLTV >= 90% → gap 0.5%  |  LLTV < 90% → gap 5%

Data: api.lista.org  |  BSC Mainnet
```

### 繁體中文 — 觸發預警

```
🚨 Lista 清算預警
━━━━━━━━━━━━━━━━━━━━━━━━━
錢包：0xAbCd...5678

#1  slisBNB / WBNB
─────────────────────
抵押品：  2.50 slisBNB（約 ＄1,450）
負債：    1.89 WBNB（約 ＄1,300）
健康係數：1.004 🔴
當前 LTV：89.6%  /  清算線：90.0%
LTV 差距：0.4% ⚠️ 低於預警閾值（0.5%）

清算觸發價格：＄532.40 / slisBNB（當前價：＄580.00）
距離清算：    ＄47.60（8.2%）

操作建議：立即補充抵押品或償還部分借款。
          償還約 0.15 WBNB 可將 LTV 降至 80%。
━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 繁體中文 — 全部正常

```
✅ Lista 持倉檢查 — 全部正常
━━━━━━━━━━━━━━━━━━━━━━━━━
錢包：0xAbCd...5678
<YYYY-MM-DD HH:MM> UTC

沒有持倉接近清算線。

#1  BTCB / U  |  健康係數：1.83 ✅  |  LTV：47.1% / 86.0%  |  差距：38.9%
#2  ETH / USD1  |  健康係數：2.14 ✅  |  LTV：37.4% / 80.0%  |  差距：42.6%

━━━━━━━━━━━━━━━━━━━━━━━━━
預警閾值：LLTV >= 90% → 差距 0.5%  |  LLTV < 90% → 差距 5%

資料來源：api.lista.org  |  BSC 主網
```

## D.4 — Threshold customization

- "change threshold to X%" / "把閾值改成 X%" → update immediately, no confirmation.
- "restore default" / "恢復預設" → reset to system defaults.

## D.5 — Push notification setup

If user says "enable alerts" / "開啟告警" / "开启告警":

> **EN:** Would you like to set a custom alert threshold first? Default: LLTV >= 90% → gap 0.5%, LLTV < 90% → gap 5%.
> **中文：** 是否需要先設定自訂預警閾值？預設：LLTV >= 90% → 差距 0.5%，LLTV < 90% → 差距 5%。

After threshold is confirmed (or user accepts default):

> **EN:** Which channel? 1) Telegram  2) Discord
> **中文：** 推送渠道？1) Telegram  2) Discord

After channel is selected:

> **EN:** Done. Alerts will be sent via [channel] when any position crosses the threshold.
> **中文：** 已設置。當任何持倉觸及預警閾值時，系統將透過 [渠道] 通知你。

If user says "cancel alerts" / "disable alerts" / "取消推送" / "關閉告警" / "关闭告警":

> **EN:** Alerts have been disabled. You can re-enable anytime with "enable alerts".
> **中文：** 已關閉推送告警。隨時可輸入「開啟告警」重新啟用。

---

# REPORT E — Daily Digest

Position overview + estimated yield + market snapshot in one report.

## E.1 — Fetch data

Positions: same as Report A step A.1, metrics via **APPENDIX: Shared Computation**.

Vaults:
```bash
curl -s "https://api.lista.org/api/moolah/vault/list?pageSize=100"
```

Yield estimation (no historical snapshot needed):
- `estimatedDailyYield = supplyUSD × vaultAPY / 365` per vault
- `estimatedWeeklyYield = estimatedDailyYield × 7`

Market snapshot rate deltas:
- If previous rate data is available (e.g. from a prior run), show change indicators: ↑, ↓, or "unchanged" / "持平"
- If no previous data, omit the delta parenthetical

## E.2 — Output template

### English

```
📊 Lista Daily Digest · <YYYY-MM-DD>
Wallet: 0xAbCd...5678
━━━━━━━━━━━━━━━━━━━━━━━━━

🏦 Lending Positions
─────────────────────
#1  BTCB / U
   Collateral: 2.50 BTCB (~＄250,000)
   Debt:       100,000.00 U (~＄100,000)
   Health factor: 2.15 ✅
   LTV: 40.0% / LLTV: 86.0%
   Liq. price: ＄46,511 / BTCB (current: ＄100,000)

─────────────────────

💰 Estimated Yield (at current APY)
   WBNB Vault: ~＄2.31/day  (12.4% APY on ＄6,800 supply)
   USD1 Vault: ~＄0.87/day  (5.6% APY on ＄5,670 supply)
   Daily total: ~＄3.18  |  Weekly: ~＄22.26

─────────────────────

📈 Market Snapshot
   USDT borrow rate: 8.2% (↑0.3%)
   WBNB Vault APY: 12.4% (unchanged)
   USD1 Vault APY: 5.6% (↓0.2%)

━━━━━━━━━━━━━━━━━━━━━━━━━

Data: api.lista.org  |  BSC Mainnet
```

If risk alerts exist, add before closing ━━━. Use two tiers:
- HF < 1.2 but above alert threshold → ⚠️ yellow warning
- At or below alert threshold → 🔴 red alert

```
─────────────────────

⚠️  Risk Alerts
   #2 slisBNB / WBNB — HF: 1.15 ⚠️  LTV gap: 4.2%
   Consider adding collateral or repaying ~0.5 WBNB.

🔴 Critical
   #3 ETH / USDT — HF: 1.004 🔴  LTV gap: 0.4%
   Repay debt or add collateral immediately.
```

### 繁體中文

```
📊 Lista 每日報告 · <YYYY-MM-DD>
錢包：0xAbCd...5678
━━━━━━━━━━━━━━━━━━━━━━━━━

🏦 借貸持倉
─────────────────────
#1  BTCB / U
   抵押品：2.50 BTCB（約 ＄250,000）
   負債：  100,000.00 U（約 ＄100,000）
   健康係數：2.15 ✅
   LTV：40.0% / 清算線：86.0%
   清算價格：＄46,511 / BTCB（當前價：＄100,000）

─────────────────────

💰 預估收益（按當前年化）
   WBNB 金庫：約 ＄2.31/日（12.4% 年化，供款 ＄6,800）
   USD1 金庫：約 ＄0.87/日（5.6% 年化，供款 ＄5,670）
   每日合計：約 ＄3.18  |  每週：約 ＄22.26

─────────────────────

📈 市場快訊
   USDT 借款利率：8.2%（↑0.3%）
   WBNB 金庫年化：12.4%（持平）
   USD1 金庫年化：5.6%（↓0.2%）

━━━━━━━━━━━━━━━━━━━━━━━━━

資料來源：api.lista.org  |  BSC 主網
```

For weekly: replace "Daily Digest" / "每日報告" with "Weekly Digest" / "每週報告". Show weekly yield estimates instead of daily.

## E.3 — Subscription setup

If user says "subscribe to daily" / "訂閱日報" / "订阅日报" or "subscribe to weekly" / "訂閱週報" / "订阅周报":

> **EN:** Which channel? 1) Telegram  2) Discord
> **中文：** 推送渠道？1) Telegram  2) Discord

After channel is selected:

> **EN (daily):** Done. Daily reports will be delivered at 08:00 UTC via [channel].
> **EN (weekly):** Done. Weekly reports will be delivered every Monday at 08:00 UTC via [channel].
> **中文（日報）：** 已設置。每日報告將於 08:00 UTC 透過 [渠道] 發送。
> **中文（週報）：** 已設置。每週報告將於每週一 08:00 UTC 透過 [渠道] 發送。

If user says "cancel subscription" / "unsubscribe" / "取消推送" / "取消訂閱" / "取消订阅":

> **EN:** Subscription cancelled. You can re-subscribe anytime.
> **中文：** 已取消訂閱。隨時可重新訂閱。

---

# APPENDIX — Shared Computation

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

All raw values are 1e18 integers. Use float for display.

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
