---
name: renzo
description: Query Renzo crypto liquid restaking protocol — DeFi vault yields, TVL, ezETH exchange rates, EigenLayer operators, supported blockchain networks, user token balances, and withdrawal status.
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
- Their Renzo token balances or portfolio (given an Ethereum address)
- ezETH withdrawal requests, unstaking status, or cooldown timers

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
| `get_token_balances` | User's Renzo token balances (ezETH, pzETH, vault LPs) with ETH/USD values | Required: `{"address":"0x..."}` (Ethereum address) |
| `get_withdrawal_requests` | User's pending ezETH withdrawal requests with claimability and time remaining | Required: `{"address":"0x..."}` (Ethereum address) |

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

# User-specific queries (require an Ethereum address)
./skills/renzo/renzo-mcp.sh get_token_balances '{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"}'
./skills/renzo/renzo-mcp.sh get_withdrawal_requests '{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"}'
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

### Example: Token Balances Response

The `get_token_balances` tool returns:
```json
{
  "address": "0xABC...123",
  "network": "Ethereum Mainnet",
  "tokens": [
    {
      "symbol": "ezETH",
      "balance": "12.5432",
      "balanceEth": "13.4476",
      "balanceUsd": 28751.23
    },
    {
      "symbol": "pzETH",
      "balance": "5.0000",
      "balanceEth": "5.2100",
      "balanceUsd": 11134.02
    }
  ],
  "totalValueUsd": 39885.25
}
```

Present this as:
> **Renzo Portfolio** for `0xABC...123` on Ethereum Mainnet:
>
> | Token | Balance | Value (ETH) | Value (USD) |
> |-------|---------|-------------|-------------|
> | ezETH | 12.5432 | 13.4476 | $28,751.23 |
> | pzETH | 5.0000 | 5.2100 | $11,134.02 |
>
> **Total value: $39,885.25**

If `tokens` is empty, tell the user the address holds no Renzo tokens on Ethereum mainnet.

### Example: Withdrawal Requests Response

The `get_withdrawal_requests` tool returns:
```json
{
  "address": "0xABC...123",
  "requests": [
    {
      "withdrawRequestId": 42,
      "ezEthAmount": "2.5000",
      "ethAmount": "2.6803",
      "claimable": false,
      "createdAt": "2025-02-10T14:30:00Z",
      "claimableAt": "2025-02-17T14:30:00Z",
      "timeRemainingSeconds": 172800
    }
  ],
  "totalRequests": 1,
  "cooldownPeriodSeconds": 604800
}
```

Present this as:
> **Pending Withdrawals** for `0xABC...123`:
>
> | # | ezETH | ETH to Receive | Status | Claimable At |
> |---|-------|---------------|--------|-------------|
> | 42 | 2.5000 | 2.6803 | Pending (~2 days left) | Feb 17, 2025 |
>
> The cooldown period is **7 days** from the time of request. Once claimable, the ETH can be claimed from the WithdrawQueue contract.

If `requests` is empty, tell the user they have no pending ezETH withdrawals.

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
| "What are my Renzo balances?" (given an address) | `get_token_balances` with the address |
| "Check my withdrawal status" (given an address) | `get_withdrawal_requests` with the address |
| Full portfolio overview (given an address) | `get_token_balances` + `get_withdrawal_requests` |

When the user asks about a specific vault by name, first call `get_vaults` to find the symbol, then call `get_vault_details` with that symbol.

When the user provides an Ethereum address (0x..., 42 hex characters), use it directly with `get_token_balances` or `get_withdrawal_requests`. If the user asks about "my position" or "my balance" without providing an address, ask them for their Ethereum address first.

## Error Handling

If the script exits with an error:
- **Network failure**: Tell the user the Renzo MCP server is unreachable and suggest trying again later
- **Unknown tool**: This is a bug in the skill — report the tool name attempted
- **Invalid JSON arguments**: Check the argument format matches what the tool expects
- **Server error**: Report the error message from the server response

Never show raw JSON error output to the user. Summarize the issue in plain language.
