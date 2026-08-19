# GMX V2 Order Flow Testing & Research

![Foundry Tests](https://github.com/aftermoon2547/gmx-order-flow-testing/actions/workflows/test.yml/badge.svg)

A Foundry-based research and testing project for exploring **GMX V2 order execution flows on Arbitrum**.

This project reproduces and validates GMX V2 perpetual trading order lifecycles using an **Arbitrum One mainnet fork** and automated **GitHub Actions CI**.

## Overview

The goal of this repository is to study how GMX V2 handles:

- Creating increase orders
- Executing orders through keeper flows
- Opening long positions
- Creating decrease orders
- Closing positions
- Oracle price simulation during execution

## Tech Stack

- **Solidity 0.8.24**
- **Foundry / Forge**
- **GMX V2**
- **Arbitrum One**
- **Mainnet fork testing**
- **GitHub Actions**

## Testing Environment

Tests run against an Arbitrum One mainnet fork using Foundry.

The same test suite is automatically executed through GitHub Actions CI.

Environment:

- Chain: Arbitrum One
- Fork block: `392496384`
- Solidity: `0.8.24`
- Framework: Foundry

## Implemented Flows

### Open Long Position

The test flow:

1. Fund test user with ETH
2. Create GMX increase order
3. Send collateral to Order Vault
4. Execute order through keeper flow
5. Validate the created position

### Close Long Position

The test flow:

1. Create an existing position
2. Create a decrease order
3. Execute the decrease order
4. Validate received assets

## Test Results

Latest successful CI run:

- `testOpenLongPosition()` — PASS
- `testCloseLongPosition()` — PASS

**2 tests passed, 0 failed, 0 skipped.**

## Continuous Integration

Every push to the `main` branch triggers GitHub Actions to:

1. Install the Foundry toolchain
2. Compile Solidity contracts
3. Connect to an Arbitrum One mainnet fork
4. Execute GMX V2 integration tests

Current CI status:

**2 tests passed, 0 failed, 0 skipped.**

## Research Notes

The tests use a local Arbitrum fork and simulate GMX V2 order execution.

A mock oracle provider is used during execution to provide deterministic WETH and USDC prices.

Current test configuration:

- WETH price: `$3,892`
- USDC price: `$1`
- ETH collateral: `0.001 ETH`
- Position size: approximately `$9`
- Execution fee: simulated test value

## Repository Structure

```text
contracts/
├── interfaces/
├── mock/
└── utils/

test/
└── GmxOrderFlow.t.sol

.github/
└── workflows/
    └── test.yml

foundry.toml
foundry.lock
README.md
