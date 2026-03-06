# Lista Lending Agent Skills

LLM-agnostic agent skills for [Lista Lending](https://lista.org/lending) — daily DeFi workflows on BSC, powered by live on-chain data via MCP.

## Skills

| Command | Description |
|---|---|
| `/lista-report <wallet(s)>` | Report hub: position status, market overview, yield scan, risk check, daily digest |
| `/lista-loop <collateral> <borrow> <amount>` | Calculate optimal leverage loop strategy & net APY |

### lista-report sub-reports

| # | Report | Wallet required |
|---|---|---|
| 1 | Position Report — collateral, debt, health factor, LTV, liquidation price, recommendations | Yes |
| 2 | Market Overview — TVL, lending rates, top vaults, high-utilization markets | No |
| 3 | Yield Opportunities — best deposit APY across vaults | No |
| 4 | Risk Check — liquidation risk alerts with configurable thresholds | Yes |
| 5 | Daily Digest — positions + yield + market snapshot in one report | Yes |

## Repository Structure

```
.agents/
├── lista-report/
│   ├── SKILL.md              # Orchestrator: language, format rules, menu, dispatch
│   ├── REFERENCE.md          # Index of reference files
│   └── references/
│       ├── computation.md    # Shared: price fetching, metrics, correlated pairs, risk levels
│       ├── position.md       # Report 1: position report templates (EN + 中文)
│       ├── market.md         # Report 2: market overview templates
│       ├── yield.md          # Report 3: yield opportunities templates
│       ├── risk.md           # Report 4: risk check templates + push setup
│       └── digest.md         # Report 5: daily digest templates + subscription
├── lista-loop/
│   └── SKILL.md              # Loop strategy calculator
└── scripts/
    └── moolah.js             # Node.js RPC helper (fallback only, no external deps)
```

> `lista-market` and `lista-yield` are distributed as user-settings skills (not tracked as files in this repo).

## Setup

### 1. Connect the Lista MCP server

Skills fetch live data via MCP (Model Context Protocol). Add the Lista MCP server to your LLM tool:

**Claude Code** — run once:
```bash
claude mcp add lista --transport sse https://localhost:3001/mcp
```

**Cursor / other MCP clients** — add to your MCP config file:
```json
{
  "mcpServers": {
    "lista": {
      "url": "https://localhost:3001/mcp"
    }
  }
}
```

> Replace the URL above with the production endpoint if provided by Lista.

### 2. Install skills

#### Via npx skills (recommended)

```bash
# Install all skills
npx skills add lista-dao/skills

# Install specific skills only
npx skills add lista-dao/skills --skill lista-report

# List available skills first
npx skills add lista-dao/skills --list

# Install globally (available across all projects)
npx skills add lista-dao/skills -g
```

Supports: Claude Code, Codex, Cursor, OpenCode, Gemini CLI, and 30+ more agents.

#### Via add-skill (alternative)

```bash
npx add-skill lista-dao/skills
```

## Usage Examples

```
/lista-report 0xYourWalletAddress          # position report (default)
/lista-report 0xWallet1 0xWallet2          # multi-wallet
/lista-report                               # pick from 5 report types
/lista-loop slisBNB WBNB 10
/lista-loop BTCB USD1 0.5
```

Language and wallet address are saved locally (`~/.lista/`) on first run — no need to re-enter.

## How It Works

Each skill is a plain markdown prompt file. The LLM loads the relevant skill and fetches live data using **Lista MCP tools**:

1. **`lista_get_position`** — wallet positions, collateral, debt, prices
2. **`lista_get_borrow_markets`** — market rates, LLTV, liquidity, supply APY
3. **`lista_get_lending_vaults`** — vault APY, TVL, per-market allocation weights
4. **`lista_get_oracle_price`** — token and LP price (ERC20, LST, Smart Lending LP)
5. **`lista_get_staking_info`** — slisBNB / BNB-LST native staking yield

No backend infrastructure required. `moolah.js` is available as a last-resort fallback for direct RPC access when MCP is unavailable.

SKILL.md uses progressive disclosure: it's a compact orchestrator that dispatches to reference files in `references/` on demand, so the LLM only loads what it needs for the selected report type.

## Data Sources

- **Lista MCP:** `lista_get_position`, `lista_get_borrow_markets`, `lista_get_lending_vaults`, `lista_get_oracle_price`, `lista_get_staking_info`
- **BSC RPC (fallback):** `https://bsc-dataseed.bnbchain.org` via `moolah.js`
- **Smart Contracts:** See [docs/rpc-reference.md](docs/rpc-reference.md) for all contract addresses

## Docs

- [API Reference](docs/api-reference.md) — REST endpoints with curl examples
- [RPC Reference](docs/rpc-reference.md) — Moolah ABI, eth_call examples, contract addresses

## About Lista Lending

Lista Lending (powered by the Moolah protocol) is a permissionless lending protocol on BNB Smart Chain. It features:
- **Isolated markets** — each market has its own collateral, oracle, and risk params
- **Curated vaults** — ERC4626 vaults that aggregate capital across markets
- **Smart Lending** — collateral doubles as DEX liquidity, earning trading fees
- **Fixed Rate markets** — via PT token integrations
- **Alpha / Aster Zones** — curated markets for emerging and partner assets

Learn more: [docs.bsc.lista.org](https://docs.bsc.lista.org/lista-lending/smart-contract)
