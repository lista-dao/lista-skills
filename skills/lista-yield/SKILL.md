---
name: lista-yield
description: Scan best yield opportunities across all Lista vaults
---

# Lista Lending — Yield Scanner

Scan all vaults and surface the best deposit opportunities. Optionally filter by asset symbol.

**API base:** `https://api.lista.org/api/moolah`

---

## BEFORE ANYTHING ELSE — Ask for language

Do NOT run any commands until the user has answered this question:

> Which language should I use for the output?
> 請問輸出以哪種語言生成？
>   1) English
>   2) 简体中文
>   3) 繁體中文
>   4) Other (specify)

**Language handling rules:**
- **1 / English** — use the English format template exactly.
- **2 / 简体中文** — use the 繁體中文 format template, then convert all Traditional Chinese characters to Simplified Chinese. Do NOT alter any numbers, symbols, separators, or field layout.
- **3 / 繁體中文** — use the 繁體中文 format template exactly.
- **4 / Other** — translate all label text into the requested language. Keep every separator line, number format, and indentation identical to the English template. Do NOT add bullet points or reformat rows.

Remember the answer and use it for all output generated below.

---

## Step 1 — Fetch all vaults

```bash
curl -s "https://api.lista.org/api/moolah/vault/list?pageSize=100"
```

Key fields per vault (`response.data.list`):

| Field | Description |
|---|---|
| `assetSymbol` | Token deposited (WBNB, USD1…) |
| `apy` | Base supply APY (decimal: 0.087 = 8.7%) |
| `emissionApy` | LISTA token bonus APY |
| `emissionEnabled` | Whether LISTA rewards are active |
| `depositsUsd` | Total TVL in USD |
| `utilization` | Current utilization |
| `zone` | 0=Classic, 1=Alpha, 4=Aster |

## Step 2 — Sort and filter

- Filter by `assetSymbol` if user specified an asset
- `totalApy = apy + (emissionApy if emissionEnabled else 0)`
- Sort by `totalApy` descending within each zone

## Step 3 — For top 5 vaults, fetch market allocation breakdown

```bash
curl -s "https://api.lista.org/api/moolah/vault/allocation?address=<VAULT_ADDRESS>&pageSize=100"
```

Identify from allocation (`response.data.list`):
- `smartCollateralConfig != null` → Smart Lending market (extra DEX fees)
- `termType == "fixed"` → Fixed Rate market
- `zone == 1` → Alpha (higher risk/reward)
- `zone == 4` → Aster (partner assets)

---

## Step 4 — Generate output

**STRICT FORMAT RULES — follow exactly, no exceptions:**
- Copy the template below character-for-character, including separator lines (━━━).
- Use the exact field labels shown. Do NOT rename, reorder, or omit any field.
- Plain text only — no markdown bold/italics. Intended for Telegram/Discord paste.
- Note: In the templates below, ＄ represents $. Use the regular $ in actual output.

### English format

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

━━━━━━━━━━━━━━━━━━━━━━━━━

💡 High utilization (>85%) = rates may rise. Smart Lending earns DEX trading fees.

Data: api.lista.org  |  BSC Mainnet
```

### 繁體中文格式

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

━━━━━━━━━━━━━━━━━━━━━━━━━

💡 高利用率（>85%）代表利率可能上升。Smart Lending 可賺取 DEX 交易手續費。

資料來源：api.lista.org  |  BSC 主網
```

APY values are decimal strings — multiply × 100 for display.
