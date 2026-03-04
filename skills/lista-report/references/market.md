> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Report B — Market Overview

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
