---
name: renzo
description: Query Renzo protocol data — ezETH metrics, vault yields, TVL, supported chains, and EigenLayer operators.
metadata:
  openclaw:
    emoji: "\U0001F7E2"
    requires:
      bins: ["curl", "jq"]
---

# Renzo Protocol

Query live data from the Renzo liquid restaking protocol: ezETH metrics, vault information, protocol stats, supported chains, and operator details.

## When to Use

Activate this skill when the user asks about:
- Renzo protocol, ezETH, pzETH, ezSOL, or any Renzo vault token
- Liquid restaking yields, APRs, or staking returns on Renzo
- Renzo TVL (total value locked) or protocol statistics
- Renzo vault details, performance, or comparisons
- Chains or networks Renzo supports
- EigenLayer operators delegated through Renzo
- Institutional vault management on Renzo

## Available Tools

The helper script `renzo-mcp.sh` (located in this skill's directory) calls the Renzo MCP server and returns clean JSON.

| Tool | Purpose | Arguments |
|------|---------|-----------|
| `get_ezeth_info` | ezETH metrics: APR, supply, TVL, price, exchange rate | None |
| `get_protocol_stats` | Aggregate protocol stats: total TVL, APRs, chain count | None |
| `get_supported_chains` | List of blockchain networks Renzo operates on | None |
| `get_vaults` | List vaults with TVL and APR | Optional: `{"ecosystem":"eigenlayer"}` (eigenlayer, symbiotic, jito, generic) |
| `get_vault_details` | Detailed info for one vault | Required: `{"vaultId":"<symbol_or_address>"}` |
| `get_operators` | List protocol operators | Optional: `{"product":"ezETH"}` (ezETH, pzETH, ezSOL, etc.) |

## How to Call

Run the helper script via the Bash tool. The script path is relative to this skill's directory.

```bash
# No arguments
./skills/renzo/renzo-mcp.sh get_ezeth_info
./skills/renzo/renzo-mcp.sh get_protocol_stats
./skills/renzo/renzo-mcp.sh get_supported_chains

# With optional filter
./skills/renzo/renzo-mcp.sh get_vaults '{"ecosystem":"jito"}'
./skills/renzo/renzo-mcp.sh get_operators '{"product":"pzETH"}'

# With required argument
./skills/renzo/renzo-mcp.sh get_vault_details '{"vaultId":"ezREZ"}'
```

## Presenting Results

Format data for readability. Follow these rules:

- **APR/APY**: Display as percentages with 2 decimal places (e.g., "2.84%")
- **TVL**: Format in USD with commas and 2 decimal places (e.g., "$430,813,580.01"). For values over $1M, also show shorthand (e.g., "$430.8M")
- **Exchange rates**: Show 4-6 decimal places (e.g., "1.0721 ETH per ezETH")
- **Token amounts**: Show 2-4 decimal places with the token symbol
- **Tables**: Use markdown tables when comparing vaults or listing multiple items
- **Context**: Briefly explain what the numbers mean for users unfamiliar with liquid restaking

### Example: ezETH Info Response

The `get_ezeth_info` tool returns:
```json
{
  "token": "ezETH",
  "aprPercent": 2.83,
  "aprAvgPeriodDays": 30,
  "totalSupplyEth": 215986.82,
  "lpTotalSupply": 201461.43,
  "tvlUsd": 430813580.00,
  "ethPriceUsd": 2138.44,
  "exchangeRate": 1.0721
}
```

Present this as:
> **ezETH** is currently earning **2.83% APR** (30-day average). The exchange rate is **1.0721 ETH per ezETH**, with a total TVL of **$430.8M**. Total supply is **215,987 ETH**.

### Example: Vault Comparison

When listing vaults, use a table:

| Vault | Underlying | APR | TVL | Ecosystem |
|-------|-----------|-----|-----|-----------|
| ezSOL | SOL | 6.41% | $6.4M | Jito |
| ezEIGEN | EIGEN | 18.23% | $329.6K | EigenLayer |
| ezREZ | REZ | 1.64% | $2.5M | EigenLayer |

### Example: Protocol Overview

For general "tell me about Renzo" questions, call `get_protocol_stats` and `get_ezeth_info` together, then summarize:

> Renzo is a liquid restaking protocol with **$469.5M total TVL** across **8 chains**. The flagship product ezETH earns **2.84% APR** with pzETH at **2.34% APR**. The protocol spans both EVM chains (Ethereum, Arbitrum, Base, Linea, BNB Chain, Mode, Blast) and Solana.

## Choosing the Right Tool

| User asks about... | Call |
|-------------------|------|
| ezETH price, rate, yield, APR, TVL | `get_ezeth_info` |
| Overall Renzo stats, total TVL | `get_protocol_stats` |
| Which chains/networks Renzo supports | `get_supported_chains` |
| Available vaults, vault yields, vault comparison | `get_vaults` |
| Specific vault details (by name or symbol) | `get_vault_details` with the vault symbol |
| EigenLayer operators, validators, delegation | `get_operators` |
| General Renzo overview | `get_protocol_stats` + `get_ezeth_info` |
| "What yield can I get?" | `get_vaults` (shows all vault APRs) |

When the user asks about a specific vault by name, first call `get_vaults` to find the symbol, then call `get_vault_details` with that symbol.

## Error Handling

If the script exits with an error:
- **Network failure**: Tell the user the Renzo MCP server is unreachable and suggest trying again later
- **Unknown tool**: This is a bug in the skill — report the tool name attempted
- **Invalid JSON arguments**: Check the argument format matches what the tool expects
- **Server error**: Report the error message from the server response

Never show raw JSON error output to the user. Summarize the issue in plain language.
