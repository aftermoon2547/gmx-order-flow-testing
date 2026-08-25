# GMX V2 Order Flow Testing & Security Research

![Foundry Tests](https://github.com/aftermoon2547/gmx-order-flow-testing/actions/workflows/test.yml/badge.svg)

Security research and reproducible testing of **GMX V2 perpetual trading flows on Arbitrum One** using Foundry and mainnet-fork execution.

This repository combines:

- GMX V2 order-flow reproduction
- Arbitrum One mainnet-fork testing
- Solidity / EVM state analysis
- Accounting invariant analysis
- Pending price-impact research
- Regression-oriented security testing
- Reproducible security research documentation

---

## Security Research

### Pending Price Impact — Partial Close Accounting

The primary completed research case study in this repository investigates whether asymmetric integer rounding during partial position decreases can create cumulative accounting drift in GMX Synthetics V2.

**Research status:** CLOSED

**Result:** No exploitable accounting inconsistency identified within the investigated scope.

The investigation examined the interaction between:

```text
position.pendingImpactAmount
totalPendingImpactAmount
proportionalPendingImpactAmount
positionImpactPoolAmount
lentPositionImpactPoolAmount
