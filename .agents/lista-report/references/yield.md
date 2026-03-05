> Follow the FORMAT ENFORCEMENT rules from SKILL.md. Output must match templates character-for-character.

# Report C — Vault Yield

Query each Vault's current APY, TVL, and underlying market allocations.

## C.1 — Fetch data

```
lista_get_lending_vaults({ pageSize: 50 })
```

Returns per vault: `address`, `name`, `apy`, `emissionApy`, `emissionEnabled`, `depositsUsd`, `assetSymbol`, `collaterals` (market list), `utilization`, `zone`.

If user asks about a specific asset (e.g. "BNB yield", "USDT 收益"), pass `keyword` parameter to filter.

For each vault's "Top Markets" allocation weights, fetch:

```bash
curl -s "https://api.lista.org/api/moolah/vault/allocation?address=<VAULT>&pageSize=100"
```

Each allocation entry has: `collateralSymbol`, `loanSymbol`, `allocation` (decimal weight, 0.44 = 44%), `smartCollateralConfig`, `termType`.

## C.2 — Compute

- `totalApy = apy + (emissionApy if emissionEnabled else 0)`
- Sort by `totalApy` descending within each zone.
- Group by zone: 0=Classic, 1=Alpha, 4=Aster.
- For each vault, list top 3 underlying markets by `allocation` weight from the allocation API.

## C.3 — Output template

### English

```
💰 Lista Lending — Vault Yield
<YYYY-MM-DD HH:MM> UTC  |  BSC Mainnet
━━━━━━━━━━━━━━━━━━━━━━━━━

🏆 Classic (Audited)

| # | Vault | Base APY | LISTA | Total APY | TVL | Util | Top Markets |
|---|-------|----------|-------|-----------|-----|------|-------------|
| 1 | WBNB Vault (WBNB) | 4.2% | 2.1% | 6.3% | ＄18.2M | 52% | slisBNB/WBNB 39% ⚡, PT-slisBNBx/WBNB 21% 🔒 |
| 2 | USD1 Vault (USD1) | 3.1% | 1.8% | 4.9% | ＄8.3M | 61% | BTCB/USD1 44%, WBNB/USD1 32% |
| 3 | U Vault (U) | 2.5% | 0% | 2.5% | ＄5.1M | 48% | slisBNB/U 52%, BTCB/U 30% |

━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  Alpha (Higher Risk)

| # | Vault | Base APY | LISTA | Total APY | TVL | Util | Top Markets |
|---|-------|----------|-------|-----------|-----|------|-------------|
| 1 | WBTC Vault (WBTC) | 14.2% | 0% | 14.2% | ＄420K | 78% | WBTC/USD1 100% |

━━━━━━━━━━━━━━━━━━━━━━━━━

⚡ Smart Lending  |  🔒 Fixed Rate

Data: api.lista.org  |  BSC Mainnet
```

Notes:
- One table per zone. Omit a zone section entirely if no vaults exist in that zone.
- If user filtered by asset, show only matching vaults and replace title with: `💰 Lista Lending — <ASSET> Vault Yield`.
- If `emissionApy` is 0 or emission is disabled, show `0%` in LISTA column.

### 繁體中文

```
💰 Lista Lending — Vault 收益
<YYYY-MM-DD HH:MM> UTC  |  BSC 主網
━━━━━━━━━━━━━━━━━━━━━━━━━

🏆 Classic（已審計）

| # | 金庫 | 基礎年化 | LISTA | 總年化 | TVL | 利用率 | 底層市場 |
|---|------|----------|-------|--------|-----|--------|----------|
| 1 | WBNB 金庫（WBNB） | 4.2% | 2.1% | 6.3% | ＄18.2M | 52% | slisBNB/WBNB 39% ⚡、PT-slisBNBx/WBNB 21% 🔒 |
| 2 | USD1 金庫（USD1） | 3.1% | 1.8% | 4.9% | ＄8.3M | 61% | BTCB/USD1 44%、WBNB/USD1 32% |
| 3 | U 金庫（U） | 2.5% | 0% | 2.5% | ＄5.1M | 48% | slisBNB/U 52%、BTCB/U 30% |

━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  Alpha（較高風險）

| # | 金庫 | 基礎年化 | LISTA | 總年化 | TVL | 利用率 | 底層市場 |
|---|------|----------|-------|--------|-----|--------|----------|
| 1 | WBTC 金庫（WBTC） | 14.2% | 0% | 14.2% | ＄420K | 78% | WBTC/USD1 100% |

━━━━━━━━━━━━━━━━━━━━━━━━━

⚡ Smart Lending  |  🔒 固定利率

資料來源：api.lista.org  |  BSC 主網
```
