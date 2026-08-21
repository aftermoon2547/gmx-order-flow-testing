# Pending Impact Amount — Rounding Investigation

Date: 2026-08-21

## Scope

Investigation of `pendingImpactAmount` proportional reduction during
partial position decreases.

The main question is whether integer rounding during repeated partial
decreases can create an accounting discrepancy between:

- `position.pendingImpactAmount`
- `totalPendingImpactAmount`
- the proportional pending impact removed during a decrease

## Relevant Code

### DecreasePositionCollateralUtils.sol

Function:

`_getProportionalPendingImpactValues(...)`

Current calculation:

```solidity
int256 proportionalPendingImpactAmount =
    Precision.mulDiv(
        positionPendingImpactAmount,
        sizeDeltaUsd,
        sizeInUsd,
        positionPendingImpactAmount < 0
    );
