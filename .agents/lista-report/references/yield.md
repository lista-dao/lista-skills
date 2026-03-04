> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Report C — Vault Yield

Query each Vault's current APY, TVL, and underlying market allocations.

## C.1 — Fetch data

```bash
# All vaults
curl -s "https://api.lista.org/api/moolah/vault/list?pageSize=100"

# For each vault, get underlying market allocations
curl -s "https://api.lista.org/api/moolah/vault/allocation?address=<VAULT>&pageSize=100"
```

- If user asks about a specific asset (e.g. "BNB yield", "USDT 收益"), filter by `assetSymbol`.
- `totalApy = apy + (emissionApy if emissionEnabled else 0)`
- Sort by `totalApy` descending within each zone.
- Group by zone: 0=Classic, 1=Alpha, 4=Aster.
- For each vault, list top 3 underlying markets by allocation weight.

## C.2 — Output template

### English

```
💰 Lista Lending — Vault Yield
<YYYY-MM-DD HH:MM> UTC  |  BSC Mainnet
━━━━━━━━━━━━━━━━━━━━━━━━━

🏆 Classic (Audited)

1. WBNB Vault  (WBNB)
   APY: 4.2% base + 2.1% LISTA = 6.3% total
   TVL: ＄18.2M  |  Utilization: 52%
   Markets: slisBNB/WBNB 39% ⚡, PT-slisBNBx/WBNB 21% 🔒, BTCB/WBNB 18%

- - - - -

2. USD1 Vault  (USD1)
   APY: 3.1% base + 1.8% LISTA = 4.9% total
   TVL: ＄8.3M  |  Utilization: 61%
   Markets: BTCB/USD1 44%, WBNB/USD1 32%

- - - - -

3. U Vault  (U)
   APY: 2.5% base + 0% LISTA = 2.5% total
   TVL: ＄5.1M  |  Utilization: 48%
   Markets: slisBNB/U 52%, BTCB/U 30%

━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  Alpha (Higher Risk)

1. WBTC Vault  (WBTC)
   APY: 14.2% base + 0% LISTA = 14.2% total
   TVL: ＄420K  |  Utilization: 78%
   Markets: WBTC/USD1 100%

━━━━━━━━━━━━━━━━━━━━━━━━━

🤝 Aster (Partner Assets)

1. <vault name>  (<asset>)
   APY: <base>% base + <emission>% LISTA = <total>% total
   TVL: ＄<tvl>  |  Utilization: <util>%
   Markets: <top allocations>

━━━━━━━━━━━━━━━━━━━━━━━━━

⚡ Smart Lending  |  🔒 Fixed Rate

Data: api.lista.org  |  BSC Mainnet
```

Notes:
- Show all vaults per zone, ranked by totalApy.
- Omit a zone section entirely if no vaults exist in that zone.
- If user filtered by asset, show only matching vaults and replace title with: `💰 Lista Lending — <ASSET> Vault Yield`.
- If `emissionApy` is 0 or emission is disabled, show `0% LISTA`.

### 繁體中文

```
💰 Lista Lending — Vault 收益
<YYYY-MM-DD HH:MM> UTC  |  BSC 主網
━━━━━━━━━━━━━━━━━━━━━━━━━

🏆 Classic（已審計）

1. WBNB 金庫（WBNB）
   年化：4.2% 基礎 + 2.1% LISTA = 6.3% 總計
   TVL：＄18.2M  |  利用率：52%
   底層市場：slisBNB/WBNB 39% ⚡、PT-slisBNBx/WBNB 21% 🔒、BTCB/WBNB 18%

- - - - -

2. USD1 金庫（USD1）
   年化：3.1% 基礎 + 1.8% LISTA = 4.9% 總計
   TVL：＄8.3M  |  利用率：61%
   底層市場：BTCB/USD1 44%、WBNB/USD1 32%

- - - - -

3. U 金庫（U）
   年化：2.5% 基礎 + 0% LISTA = 2.5% 總計
   TVL：＄5.1M  |  利用率：48%
   底層市場：slisBNB/U 52%、BTCB/U 30%

━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  Alpha（較高風險）

1. WBTC 金庫（WBTC）
   年化：14.2% 基礎 + 0% LISTA = 14.2% 總計
   TVL：＄420K  |  利用率：78%
   底層市場：WBTC/USD1 100%

━━━━━━━━━━━━━━━━━━━━━━━━━

🤝 Aster（合作夥伴資產）

1. <金庫名稱>（<資產>）
   年化：<基礎>% 基礎 + <獎勵>% LISTA = <總計>% 總計
   TVL：＄<tvl>  |  利用率：<util>%
   底層市場：<主要配置>

━━━━━━━━━━━━━━━━━━━━━━━━━

⚡ Smart Lending  |  🔒 固定利率

資料來源：api.lista.org  |  BSC 主網
```
