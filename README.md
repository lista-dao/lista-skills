# Lista Lending Agent Skills

LLM-agnostic agent skills for [Lista Lending](https://lista.org/lending) — daily DeFi workflows on BSC, powered by live on-chain data.

Skills live in `skills/` and follow the `npx skills` standard format.

## Skills

| Command | Description |
|---|---|
| `/lista-report <wallet(s)>` | Report hub: position status, market overview, yield scan, risk check, daily digest |
| `/lista-yield [asset]` | Scan best yield opportunities across all Lista vaults |
| `/lista-loop <asset> <amount> [loops]` | Calculate optimal leverage loop strategy & net APY |
| `/lista-market` | Daily protocol digest: TVL, utilization, top vaults |

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
skills/                       # Canonical skill files (npx skills format)
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
├── lista-market/
├── lista-yield/
└── scripts/moolah.js         # Shared Node.js RPC helper (no external deps)

.agents/                      # Backward compatibility (mirrors skills/)
```

## Installation

### Via npx skills (recommended)

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

### Via add-skill (alternative)

```bash
npx add-skill lista-dao/skills
```

## Usage Examples

```
/lista-report 0xYourWalletAddress          # position report (default)
/lista-report 0xWallet1 0xWallet2          # multi-wallet
/lista-report                               # pick from 5 report types
/lista-yield BNB
/lista-yield USD1
/lista-loop slisBNB BNB 10
/lista-loop BTCB BNB 0.5 3
/lista-market
```

Language and wallet address are saved locally (`~/.lista/`) on first run — no need to re-enter.

## How It Works

Each skill is a plain markdown prompt file. Any LLM tool that loads markdown slash commands from a directory can use these skills directly.

1. Call the **Lista REST API** (`https://api.lista.org/api/moolah`) for vault and market data
2. Call the **BSC RPC** (`https://bsc-dataseed.bnbchain.org`) for user-specific on-chain position data
3. Perform calculations and format results into a clean report

No backend infrastructure required — skills work out of the box using the LLM's Bash/shell tool.

SKILL.md uses progressive disclosure: it's a compact orchestrator that dispatches to reference files in `references/` on demand, so the LLM only loads what it needs for the selected report type.

## Data Sources

- **Lista REST API:** `https://api.lista.org/api/moolah`
- **BSC RPC:** `https://bsc-dataseed.bnbchain.org`
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
