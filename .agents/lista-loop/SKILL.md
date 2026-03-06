---
name: lista-loop
description: Calculate optimal leverage loop strategy and net APY for Lista Lending
---

# Lista Lending — Loop Strategy Calculator

Simulate a leverage loop strategy: deposit collateral → borrow → re-deposit → repeat.

**Input:** `<collateral_asset> <borrow_asset> <initial_amount> [target_loops]`

**MCP tools:** `lista_get_borrow_markets`, `lista_get_oracle_price`, `lista_get_staking_info`, `lista_get_position`

**First-run check:** If any `lista_*` MCP tool call fails with a connection error, the Lista MCP server is not configured. Guide the user:

> Lista MCP server is not connected. To set it up:
>
> **Claude Code:**
> ```
> claude mcp add lista --transport sse https://localhost:3001/mcp
> ```
>
> **OpenClaw** — add to `openclaw.json`:
> ```json
> { "mcpServers": { "lista": { "transport": "streamable-http", "url": "https://localhost:3001/mcp" } } }
> ```
>
> **Other MCP clients:** Add to your MCP config:
> ```json
> { "mcpServers": { "lista": { "url": "https://localhost:3001/mcp" } } }
> ```
>
> Then restart your session and try again.

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

## Step 1 — Find the relevant market

The `keyword` parameter of `lista_get_borrow_markets` filters by **loan token name**,
not collateral name. Search by the borrow asset, then filter by collateral:

```
lista_get_borrow_markets({ keyword: "<borrow_asset>", pageSize: 50 })
```

From the returned list, find the entry where `collateral == <collateral_asset>`. Collect:
- `rate` — borrow APY (decimal)
- `lltv` — liquidation LTV (decimal, e.g. "0.860000000000000000") — already in the response, no separate RPC call needed
- `id` — market identifier
- `collateralToken` — collateral token contract address
- `loanToken` — loan token contract address

If no matching market is found, paginate (`page: 2`, `page: 3`) or drop the keyword
filter and scan all markets. Inform the user if the market still cannot be found.

## Step 1.5 — Fetch token prices

**Stablecoins** (U, USD1, USDT, USDC, lisUSD): use `P = 1.00` directly — skip all calls.

**Collateral price — try in order until one succeeds:**

1. **If the user has an active position in this market**, call:
   ```
   lista_get_position({ wallet: "<address>" })
   ```
   Use `holdings[].collateralPrice` for the matching `marketId`. Most reliable.

2. **MCP oracle** (supports ERC20, LST, and Smart Lending LP tokens):
   ```
   lista_get_oracle_price({ tokenAddress: <collateralToken> })
   ```
   Use the returned `price` if `found: true`.

3. **moolah.js** (last resort — only if MCP is unavailable):
   ```bash
   node .agents/scripts/moolah.js token-price <collateralToken>
   node .agents/scripts/moolah.js lp-price <marketId>   # Smart Lending LP
   ```

## Step 2 — Get collateral native yield

For slisBNB, ankrBNB, wstBNB, BNBx:

```
lista_get_staking_info()
```

Use the returned staking APR/APY as native yield.

For other assets:
- PT tokens: use fixed rate from `terms.apy` in market response
- BTCB, stablecoins: 0% native yield

## Step 3 — Simulate loops

Variables:
- `P_c` = collateral price (USD), `P_b` = borrow price (USD)
- `r` = `rate` (annual), `y` = native yield (annual)
- `L` = LLTV, `targetLTV` = 0.70 (conservative default)

```
coll[0] = initial_amount
debt[0] = 0

for i in 1..N:
  borrowed_value = coll[i-1] × targetLTV × P_c / P_b   # in borrow token units
  coll[i] = coll[i-1] + borrowed_value × P_b / P_c       # convert back to collateral
  debt[i] = debt[i-1] + borrowed_value

  currentLTV = debt[i] × P_b / (coll[i] × P_c)
  stop recommending when currentLTV > 0.75 × L
```

Net APY at N loops:
```
grossYield  = coll[N] × y × P_c
borrowCost  = debt[N] × r × P_b
netAPY      = (grossYield − borrowCost) / (initial_amount × P_c)

liqPrice    = debt[N] × P_b / (coll[N] × L)   # in collateral asset USD
buffer      = (P_c − liqPrice) / P_c × 100%
```

Recommend the loop count that maximises net APY while keeping buffer ≥ 20%.

---

## Step 4 — Generate output

**STRICT FORMAT RULES — follow exactly, no exceptions:**
- Copy the template below character-for-character, including separator lines (━━━, ────, │).
- Use the exact column headers shown. Do NOT rename, reorder, or omit any column.
- Use `- - - - -` between the table and the recommendation block.
- Plain text only — no markdown bold/italics. Intended for Telegram/Discord paste.
- Note: In the templates below, ＄ represents $. Use the regular $ in actual output.

### English format

```
Lista Lending — Loop Strategy: slisBNB/WBNB
━━━━━━━━━━━━━━━━━━━━━━━━━
LLTV: 86%  |  Borrow Rate: 2.6% APY  |  slisBNB Native Yield: 4.2%

Loops │ Collateral      │ Debt          │ Leverage │ Net APY │ Liq Price │ Buffer
──────┼─────────────────┼───────────────┼──────────┼─────────┼───────────┼───────
  0   │ 10.0 slisBNB    │ 0             │  1.00×   │  4.20%  │     —     │   —
  1   │ 17.0 slisBNB    │  7.0 WBNB    │  1.70×   │  5.80%  │  ＄195    │  28%
  2   │ 21.9 slisBNB    │ 11.9 WBNB    │  2.19×   │  6.40%  │  ＄210    │  23%
  3   │ 25.3 slisBNB    │ 15.3 WBNB    │  2.53×   │  6.70%  │  ＄221    │  19%  <- Optimal
  4   │ 27.7 slisBNB    │ 17.7 WBNB    │  2.77×   │  6.60%  │  ＄230    │  15%  ⚠️

- - - - -

✅ Recommended: 3 loops
   Net position: 25.3 slisBNB / 15.3 WBNB debt  |  Leverage: 2.53×
   Net APY: ~6.70% (vs 4.20% unlooped)
   Liquidation price: ＄221  (current ~＄272, 19% buffer)

⚠️  Risk: Borrow rate is variable. If it rises above ~5.8%, strategy turns net negative.

━━━━━━━━━━━━━━━━━━━━━━━━━
Data: api.lista.org  |  BSC Mainnet
```

### 繁體中文格式

```
Lista Lending — 槓桿策略：slisBNB/WBNB
━━━━━━━━━━━━━━━━━━━━━━━━━
LLTV：86%  |  借款利率：2.6% APY  |  slisBNB 原生收益：4.2%

循環 │ 抵押品            │ 負債          │ 槓桿   │ 淨年化 │ 清算價格  │ 緩衝
─────┼───────────────────┼───────────────┼────────┼────────┼───────────┼──────
  0  │ 10.0 slisBNB      │ 0             │ 1.00×  │ 4.20%  │     —     │   —
  1  │ 17.0 slisBNB      │  7.0 WBNB    │ 1.70×  │ 5.80%  │ ＄195     │ 28%
  2  │ 21.9 slisBNB      │ 11.9 WBNB    │ 2.19×  │ 6.40%  │ ＄210     │ 23%
  3  │ 25.3 slisBNB      │ 15.3 WBNB    │ 2.53×  │ 6.70%  │ ＄221     │ 19%  <- 最佳
  4  │ 27.7 slisBNB      │ 17.7 WBNB    │ 2.77×  │ 6.60%  │ ＄230     │ 15%  ⚠️

- - - - -

✅ 建議：3 次循環
   淨部位：25.3 slisBNB / 負債 15.3 WBNB  |  槓桿：2.53×
   淨年化：約 6.70%（未槓桿 4.20%）
   清算價格：＄221（當前約 ＄272，緩衝 19%）

⚠️  風險：借款利率為浮動利率。若利率上升至約 5.8% 以上，策略將轉為淨虧損。

━━━━━━━━━━━━━━━━━━━━━━━━━
資料來源：api.lista.org  |  BSC 主網
```
