---
name: lista-report
description: Bilingual (EN/ZH) position report across one or more wallets — collateral, debt, LTV, liquidation price on Lista Lending
---

# Lista Lending — Position Report

Generate a structured position report across one or more wallet addresses on Moolah.

**RPC script:** `skills/scripts/moolah.js` (Node.js stdlib, no packages needed)

---

## BEFORE ANYTHING ELSE — Ask for language

Do NOT run any commands until the user has answered this question:

> Which language should I use for the report?
> 請問報告以哪種語言生成？
>   1) English
>   2) 简体中文
>   3) 繁體中文
>   4) Other (specify)

**Language handling rules:**
- **1 / English** — use the English format template exactly.
- **2 / 简体中文** — use the 繁體中文 format template, then convert all Traditional Chinese characters to Simplified Chinese. Do NOT alter any numbers, symbols, separators (━━━, ────), or field layout.
- **3 / 繁體中文** — use the 繁體中文 format template exactly.
- **4 / Other** — translate all label text (field names, headers, summary lines, recommendations) into the requested language. Keep every separator line, number format, indentation, and the #N position header format identical to the English template. Do NOT add bullet points or reformat the rows.

Remember the answer and use it for all output generated below.

---

## Step 1 — Collect addresses

Accept one or more wallet addresses from the user — comma-separated, space-separated, or line-by-line. Strip extra whitespace and deduplicate. Process them in the order received.

---

## Step 2 — Fetch positions for each address

Run once per address:

```bash
node skills/scripts/moolah.js user-positions <address>
```

Returns JSON with `positions[]`. Each entry has:

| Field | Description |
|---|---|
| `marketId` | 32-byte market ID |
| `collateralSymbol` / `loanSymbol` | Token symbols |
| `collateral` | Raw collateral amount (1e18 units) |
| `borrowShares` | User borrow shares (raw) |
| `supplyShares` | User supply shares (raw) |
| `currentDebt` | Current debt in loan token raw units (pre-computed) |
| `lastUpdateIso` | Last interest accrual timestamp |

If `positions` is empty → the address has no active positions on Moolah.

---

## Step 3 — Detect collateral type and fetch prices (per unique market)

Deduplicate marketIds across all addresses. For each unique active marketId, first fetch the market info from the Lista API — this tells you whether the collateral is a plain ERC20 token or a Curve LP token (Smart Lending market).

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

Output line 1: `ERC20` or `LP`
Output line 2: loan token USD price (float, e.g. `1.00005`)

---

### 3a — ERC20 collateral

**Step 1: Try oracle-price**

```bash
# Oracle price (1e36-scaled). May revert for some oracles — handle gracefully.
node skills/scripts/moolah.js oracle-price <marketId>
# Returns: { price (1e36-scaled), lltv, lltvPct }
```

If `oracle-price` succeeds → use path **4a** for metric computation.

**Step 2: Fallback to token-price (when oracle-price reverts)**

When `oracle-price` reverts, do NOT mark as N/A. Instead, fetch the collateral token address from market params, then query the Lista oracle directly:

```bash
# Get collateral token address and LLTV from market params
node skills/scripts/moolah.js params <marketId>
# Returns: { collateralToken, loanToken, lltv, lltvPct, ... }

# Get USD price of collateral token via Lista oracle (8 decimal places)
node skills/scripts/moolah.js token-price <collateralToken>
# Returns: { priceUSD }
```

If `token-price` returns 0 or errors → only then mark collateral USD and all USD-derived fields as `N/A`.

Use path **4c** for metric computation when the token-price fallback is used.

---

### 3b — LP collateral (Smart Lending markets)

Smart Lending markets use a Curve StableSwap LP token as collateral.
Calling `oracle-price` on these markets **always reverts** — the provider contract is not a standard Morpho oracle. Use `lp-price` instead:

```bash
node skills/scripts/moolah.js lp-price <marketId>
# Returns: { lpTokenPriceUSD, token0Symbol, token1Symbol, virtualPriceF, coin0PriceUSD }
```

This command internally:
1. Reads `smartCollateralConfig.swapPool` and `token0` from the Lista API
2. Calls `get_virtual_price()` on the Curve pool (1e18-scaled, LP value in coin0 units)
3. Calls `oracle.peek(token0)` for the coin0 USD price (8 decimal places)
4. Returns `lpTokenPriceUSD = (virtualPrice / 1e18) × (coin0PriceRaw / 1e8)`

Also fetch LLTV for LP markets (oracle-price is unavailable to provide it):

```bash
node skills/scripts/moolah.js params <marketId>
# Returns: { lltv, lltvPct, ... }
```

In the report, label the collateral as `<token0Symbol>/<token1Symbol> LP` (e.g. `slisBNB/BNB LP`).

---

## Step 4 — Compute metrics per position

All raw position values are 1e18 integers. Use floating point for display only.

### 4a — ERC20 collateral (oracle-price succeeded, path 4a)

```
collateral_f       = collateral / 1e18
currentDebt_f      = currentDebt / 1e18
oraclePrice_f      = oraclePrice / 1e36          (from oracle-price)
loanTokenUSD       = loanTokenPrice              (float, from API step 3)
lltvF              = lltv / 1e18                 (from oracle-price result)

collateral_in_loan = collateral_f × oraclePrice_f    (collateral in loan token units)
collateralPriceUSD = oraclePrice_f × loanTokenUSD    (USD per 1 collateral token)
collateralUSD      = collateral_f × collateralPriceUSD
debtUSD            = currentDebt_f × loanTokenUSD
netEquityUSD       = collateralUSD − debtUSD

LTV                = currentDebt_f / collateral_in_loan
liqPriceUSD        = debtUSD / (collateral_f × lltvF)
buffer             = (collateralPriceUSD − liqPriceUSD) / collateralPriceUSD
```

### 4b — LP collateral (Smart Lending markets)

```
collateral_f      = collateral / 1e18
currentDebt_f     = currentDebt / 1e18
lpTokenPriceUSD   = lpTokenPriceUSD             (from lp-price result)
loanTokenUSD      = loanTokenPrice              (float, from API step 3)
lltvF             = lltv / 1e18                 (from params result)

collateralUSD     = collateral_f × lpTokenPriceUSD
debtUSD           = currentDebt_f × loanTokenUSD
netEquityUSD      = collateralUSD − debtUSD

LTV               = debtUSD / collateralUSD      (USD-based, matches on-chain LTV closely)
liqPriceUSD       = debtUSD / (collateral_f × lltvF)   (LP token price that triggers liquidation)
buffer            = (lpTokenPriceUSD − liqPriceUSD) / lpTokenPriceUSD
```

### 4c — ERC20 collateral (oracle-price reverted, token-price fallback, path 4c)

```
collateral_f       = collateral / 1e18
currentDebt_f      = currentDebt / 1e18
collateralPriceUSD = priceUSD              (from token-price result, 8dp already divided)
loanTokenUSD       = loanTokenPrice        (float, from API step 3)
lltvF              = lltv / 1e18           (from params result)

collateralUSD      = collateral_f × collateralPriceUSD
debtUSD            = currentDebt_f × loanTokenUSD
netEquityUSD       = collateralUSD − debtUSD

LTV                = debtUSD / collateralUSD           (USD-based)
liqPriceUSD        = debtUSD / (collateral_f × lltvF)  (collateral price that triggers liquidation)
buffer             = (collateralPriceUSD − liqPriceUSD) / collateralPriceUSD
```

In the report, show `Collateral price: $X.XX/collateralSymbol` on a separate line (same position as the LP price line).

---

**Correlated asset check — do this BEFORE assigning risk level:**

When collateral and loan token are in the same asset family, both assets move together in price. Liquidation can only happen if the peg or ratio between them breaks — not from broad market moves. Apply more lenient risk thresholds for correlated pairs.

Asset families:
- USD stable: USD1, U, USDT, USDC, DAI, FDUSD, BUSD (any USD-pegged stablecoin)
- BNB / BNB-LST: BNB, slisBNB, wstBNB, ankrBNB, BNBx
- ETH / ETH-LST: ETH, WETH, wstETH, stETH, rETH
- BTC: BTCB, WBTC, BTC

A position is **correlated** when collateral and loan belong to the same family. For LP collateral, check whether both LP component tokens AND the loan token are all in the same family (e.g. slisBNB/BNB LP + BNB loan → BNB family → correlated; slisBNB/BNB LP + USD1 loan → different families → not correlated).

**Risk level — standard (uncorrelated pairs):**
- 🟢 SAFE     — LTV / lltvF < 80%
- 🟡 WARNING  — 80% ≤ LTV / lltvF < 90%
- 🔴 DANGER   — LTV / lltvF ≥ 90%

**Risk level — correlated pairs (adjusted):**
- 🟢 SAFE     — LTV / lltvF < 92%
- 🟡 WARNING  — 92% ≤ LTV / lltvF < 97%
- 🔴 DANGER   — LTV / lltvF ≥ 97%

For correlated positions, append "(相關對)" after the risk label in the position header (English: "(correlated)").

**Dust filter:** After computing collateralUSD and debtUSD for a position, if BOTH values are less than USD 1, skip the position entirely — do not include it in the report or count it in the position total.

**Supply-only position** (supplyShares > 0, borrowShares = 0): skip debt, LTV, and liquidation price rows.

---

## Step 5 — Position recommendations

After computing metrics for each active position, generate 1–3 concise strategy suggestions tailored to the actual numbers. Use the rules below as triggers.

**Risk reduction — uncorrelated positions:**
- LTV/LLTV ≥ 90% (DANGER): Strongly recommend repaying debt or adding collateral immediately. Show exact amounts needed to reach 75% LTV/LLTV.
- LTV/LLTV 80–90% (WARNING): Suggest partial repayment or collateral top-up. Show amounts needed to reach 70% LTV/LLTV.
- Buffer < 10%: Flag that a small price drop could trigger liquidation; recommend increasing buffer.

**Risk reduction — correlated positions:**
- LTV/LLTV ≥ 97% (DANGER): Even for correlated pairs, recommend immediate debt reduction — peg stability cannot be guaranteed at extreme leverage. Show amounts to reach 85% LTV/LLTV.
- LTV/LLTV 92–97% (WARNING): Note that broad market moves do not affect this position significantly. The real risk is a depeg event (e.g. LST smart contract exploit, stablecoin depeg). Recommend monitoring the collateral/loan price ratio and setting an alert if the ratio moves more than 3%.
- LTV/LLTV < 92% (SAFE for correlated): No action needed. Optionally suggest looping for yield.

**Yield enhancement (low LTV):**
- LTV/LLTV < 50% (uncorrelated) or < 75% (correlated): Collateral is under-utilized. Suggest borrowing more to deploy into Lista yield vaults (`/lista-yield`) or looping (`/lista-loop`).
- Supply-only position (no borrow): Mention that the user could borrow against their supply to amplify yield.

**General:**
- Always show the current borrow rate context (from `oracle-price` lltv info or market utilization if available).
- If no positions exist for an address, no recommendations needed.
- Keep recommendations factual and numeric — avoid vague language.

### English recommendation format

```
Recommendations for 0xAbCd…5678:
  1. [DANGER] Repay ~5,000 U to bring LTV to 60% and restore a safe buffer.
  2. Use /lista-yield to find the best yield for idle USDT if you reduce debt.
```

### 中文建議格式

```
地址 0xAbCd…5678 的持倉建議：
  1. 【高風險】建議償還約 5,000 U，將 LTV 降至 60%，恢復安全緩衝。
  2. 若有閒置 USDT，可使用 /lista-yield 查看最佳存款收益。
```

---

## Step 6 — Generate report

**STRICT FORMAT RULES — follow exactly, no exceptions:**
- Copy the template below character-for-character, including the separator lines (━━━ and ────).
- Use the exact field labels shown (Collateral:, Debt:, Net equity:, LTV:, Liq. price:, Last accrual:). Do NOT rename, reorder, or omit any field.
- Separate each position block from the next with a single `- - - - -` line.
- Do NOT use bullet points (•, -) inside position block rows. Use the indented plain-text rows from the template.
- Do NOT add extra sections, headings, or emoji that are not in the template.
- Number each position sequentially within an address block (#1, #2, …). Replace the "Market:" label with the position number.
- Plain text only — no markdown bold/italics. Intended for Telegram/Discord paste.
- Numbers: comma thousands separator, 2 decimal places for token amounts, rounded to nearest dollar for USD.

Note: In the templates below, ＄ (fullwidth dollar sign) represents the US dollar sign. Use the regular $ in your actual output.

### English format

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
  LTV:            47.1%  /  LLTV 86.0%
  Liq. price:     BTCB < ＄45,200  (8.2% buffer)
  Last accrual:   2026-03-01 03:12 UTC

- - - - -

#2  slisBNB/BNB LP / BNB  🟡 WARNING
  Collateral:     120.00 slisBNB/BNB LP  (~＄78,143)
  Debt:           50.00 BNB  (~＄34,550)
  Net equity:                  ~＄43,593
  LP price:       ＄651.19/LP  (virtual price 1.000110 × slisBNB ＄651.12)
  LTV:            44.2%  /  LLTV 86.0%
  Liq. price:     LP < ＄484.05  (25.7% buffer)
  Last accrual:   2026-03-01 03:12 UTC

[If no active positions:]
  No active positions.

Address 1 summary: 2 active positions  |  Net equity ~＄20.2M

Recommendations for Address 1:
  1. LTV is comfortable. Collateral is under-utilized — consider /lista-loop to amplify yield.

━━━━━━━━━━━━━━━━━━━━━━━━━
[If multiple addresses, repeat the block above for each, then:]

Total: <N> addresses  |  <M> active positions  |  Combined net equity ~＄X

Data: api.lista.org  |  BSC Mainnet
```

### 中文格式

```
Lista Lending — 持倉報告
產生時間：<YYYY-MM-DD HH:MM> UTC  |  BSC 主網
━━━━━━━━━━━━━━━━━━━━━━━━━

地址 1：0xAbCd…5678
────────────────────────
#1  BTCB / U  🟢 安全
  抵押品：    398.85 BTCB  (約 ＄38,250,000)
  負債：      18,020,988.00 U  (約 ＄18,020,988)
  淨資產：                      約 ＄20,229,012
  LTV：      47.1%  /  清算線 86.0%
  清算價格：  BTCB < ＄45,200  (緩衝 8.2%)
  最後結算：  2026-03-01 03:12 UTC

- - - - -

#2  slisBNB/BNB LP / BNB  🟡 警告
  抵押品：    120.00 slisBNB/BNB LP  (約 ＄78,143)
  負債：      50.00 BNB  (約 ＄34,550)
  淨資產：                約 ＄43,593
  LP 價格：   ＄651.19/LP  (虛擬價格 1.000110 × slisBNB ＄651.12)
  LTV：      44.2%  /  清算線 86.0%
  清算價格：  LP < ＄484.05  (緩衝 25.7%)
  最後結算：  2026-03-01 03:12 UTC

[若無活躍持倉：]
  無活躍持倉。

地址 1 小結：2 個活躍持倉  |  淨資產約 ＄20.2M

地址 1 的持倉建議：
  1. LTV 尚在安全範圍，抵押品尚有餘裕，可考慮使用 /lista-loop 提高槓桿收益。

━━━━━━━━━━━━━━━━━━━━━━━━━
[若有多個地址，重複以上區塊，最後加總：]

總計：<N> 個地址  |  <M> 個活躍持倉  |  合計淨資產約 ＄X

資料來源：api.lista.org  |  BSC 主網
```

---

## After the report — offer lista-yield

Once the report has been delivered to the user, ask:

**English:**
> Would you like me to scan for the best yield opportunities on Lista Lending right now? (runs /lista-yield)

**中文：**
> 需要我幫你掃描 Lista Lending 目前最佳的存款收益機會嗎？（執行 /lista-yield）

If the user says yes, run `/lista-yield`.
