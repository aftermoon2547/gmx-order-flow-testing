# GMX V2 Order Flow Testing & Research

A Foundry-based research and testing project for exploring **GMX V2 order execution flows on Arbitrum**.

This project simulates and validates the lifecycle of GMX V2 perpetual trading orders using an Arbitrum mainnet fork environment.

## Overview

The goal of this repository is to study how GMX V2 handles:

- Creating increase orders
- Executing orders through keepers
- Opening long positions
- Creating decrease orders
- Closing positions
- Oracle price simulation during execution

## Tech Stack

- **Solidity**
- **Foundry**
- **Forge Tests**
- **Arbitrum Mainnet Fork**
- **GMX V2 Contracts**

## Testing Environment

Tests run against a local Arbitrum fork using Foundry.

Environment:

- Chain: Arbitrum One
- Fork testing with Anvil
- Solidity version: 0.8.24

## Implemented Flows

### Open Long Position

The test flow:

1. Fund test user with ETH
2. Create GMX increase order
3. Send collateral to Order Vault
4. Execute order using keeper flow
5. Validate created position

### Close Long Position

The test flow:

1. Create an existing position
2. Create decrease order
3. Execute decrease order
4. Validate received assets

## Test Results

Latest successful run:

- `testOpenLongPosition()` — PASS
- `testCloseLongPosition()` — PASS

Result:

**2 tests passed, 0 failed, 0 skipped.**

## Research Notes

The tests use a local Arbitrum fork and simulate GMX V2 order execution.

A mock oracle provider is used during execution to provide deterministic WETH and USDC prices.

The current test configuration uses:

- WETH price: $3,892
- USDC price: $1
- ETH collateral: 0.001 ETH
- Position size: approximately $9
- Execution fee: 0.1 ETH

## Repository Structure

```text
contracts/
├── interfaces/
├── mock/
└── utils/

test/
└── GmxOrderFlow.t.sol

foundry.toml
foundry.lock
README.md
