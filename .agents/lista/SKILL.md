---
name: lista
description: Lista Lending assistant — position report, market overview, yield scan, liquidation risk check, daily digest, and loop strategy on BSC
---

# Lista Lending

Your Lista Lending (Moolah) assistant on BSC. Choose a function below.

**MCP tools:** `lista_get_position`, `lista_get_borrow_markets`, `lista_get_lending_vaults`, `lista_get_oracle_price`, `lista_get_staking_info`, `lista_get_dashboard`, `lista_get_rewards`

**First-run check:** If any `lista_*` MCP tool call fails with a connection error, the Lista MCP server is not configured. Guide the user:

> Lista MCP server is not connected. To set it up:
>
> **Claude Code:**
> ```
> claude mcp add lista --transport sse https://mcp.ltqa.io/mcp
> ```
>
> **OpenClaw** — add to `openclaw.json`:
> ```json
> { "mcpServers": { "lista": { "transport": "streamable-http", "url": "https://mcp.ltqa.io/mcp" } } }
> ```
>
> **Other MCP clients:** Add to your MCP config:
> ```json
> { "mcpServers": { "lista": { "url": "https://mcp.ltqa.io/mcp" } } }
> ```
>
> Then restart your session and try again.

---

## Step 0 — Language

### Load saved language

```bash
cat ~/.lista/language.txt 2>/dev/null
```

If the file exists and contains a valid choice (en, zh-CN, zh-TW, or a custom language name), use it silently — do NOT ask again.

### Ask if no saved language

Do NOT run any commands until the user has answered this question:

> Which language should I use for the output?
> 請問輸出以哪種語言生成？
>   1) English
>   2) 简体中文
>   3) 繁體中文
>   4) Other (specify)

Save the choice:

```bash
mkdir -p ~/.lista && echo "<CHOICE>" > ~/.lista/language.txt
```

Where `<CHOICE>` is one of: `en`, `zh-CN`, `zh-TW`, or the user's custom language name.

### Change language

When the user says "change language" / "換語言" / "换语言", ask again and overwrite `~/.lista/language.txt`.

**Language handling rules:**
- **en / English** — use the English template exactly.
- **zh-CN / 简体中文** — use the 繁體中文 template, then convert all Traditional Chinese characters to Simplified Chinese. Do NOT alter numbers, symbols, separators, or field layout.
- **zh-TW / 繁體中文** — use the 繁體中文 template exactly.
- **Other** — translate all label text into the user's language. Use natural, idiomatic phrasing (not word-for-word). Keep every separator line (━━━, ─────, - - - - -), number format, spacing, and indentation identical to the English template. Do NOT add bullet points, reformat rows, or change the structural layout.

Use the selected language for all output below.

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
>   2) Market Lending Rates — Supply APY, Borrow APY, liquidity per market
>   3) Vault Yield — APY, TVL, underlying assets per vault
>   4) Risk Check — liquidation risk alerts with thresholds
>   5) Daily Digest — positions + yield + market snapshot
>   6) Loop Strategy — leverage loop simulation, net APY, liquidation risk

If the user's original message already implies a type (e.g. "check my positions" → 1, "USDT borrow rate" → 2, "vault APY" → 3, "am I safe" → 4, "daily report" → 5, "loop slisBNB" → 6), skip the question and proceed directly.

---

## Step 2 — Wallet address (for types 1, 4, 5)

Reports 1, 4, 5 require a wallet address. Reports 2, 3, 6 do not — skip this step for them.

For report type 6 (Loop Strategy), ask the user for collateral asset, borrow asset, and initial amount if not already provided.

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

## Step 3 — Dispatch

Read the specific reference files for the selected report type:

| Report type | Read these files |
|---|---|
| 1 — Position Report | `references/domain.md`, `references/position.md` |
| 2 — Market Lending Rates | `references/market.md` |
| 3 — Vault Yield | `references/yield.md` |
| 4 — Risk Check | `references/domain.md`, `references/risk.md` |
| 5 — Daily Digest | `references/domain.md`, `references/digest.md` |
| 6 — Loop Strategy | `references/domain.md`, `references/loop.md` |

Follow the instructions in the referenced files to fetch data, compute metrics, and generate the report using the selected language.
