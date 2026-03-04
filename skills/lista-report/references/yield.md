> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Report C — Yield Opportunities

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
