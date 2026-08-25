# GMX Synthetics V2 — Partial Close Pending Impact Accounting Analysis

**Research Type:** Independent Security Research / Accounting & Invariant Analysis
**Target:** GMX Synthetics V2
**Component:** Position Price Impact / Pending Impact Accounting
**Repository:** GMX Synthetics
**Status:** Investigation completed — no exploitable accounting inconsistency identified within the investigated scope

---

## 1. Executive Summary

This research investigates the accounting behavior of `pendingImpactAmount` during partial position decreases in GMX Synthetics V2.

The primary hypothesis was that asymmetric rounding during proportional pending-impact realization could introduce cumulative accounting drift when a position is closed through multiple partial decreases.

The investigation focused on the interaction between:

* `position.pendingImpactAmount`
* `totalPendingImpactAmount`
* proportional pending-impact realization
* positive and negative pending-impact rounding
* `positionImpactPoolAmount`
* `lentPositionImpactPoolAmount`

The relevant implementation uses sign-dependent rounding when calculating the proportional pending impact realized during a position decrease:

* Positive pending impact uses downward rounding.
* Negative pending impact uses upward rounding of the absolute magnitude.

At first glance, this asymmetric rounding creates small differences from the mathematically exact fractional value. However, the realized proportional amount is applied consistently to both the individual position and the market-level `totalPendingImpactAmount`.

Existing tests also exercise partial decreases, full decreases, multiple positions, and market-level pending-impact accounting.

Within the investigated scope, no exploitable accounting inconsistency or direct value-extraction path was identified.

The investigation therefore concludes as a **negative security finding** rather than a confirmed vulnerability.

---

## 2. Scope

The investigation was limited to pending price-impact accounting during position decreases.

The following areas were examined:

```text
DecreasePositionCollateralUtils
DecreasePositionUtils
MarketUtils
Precision.mulDiv
pendingImpactAmount
totalPendingImpactAmount
positionImpactPoolAmount
lentPositionImpactPoolAmount
```

The research specifically considered:

1. Partial position decreases.
2. Positive pending price impact.
3. Negative pending price impact.
4. Repeated partial decreases.
5. Multiple positions sharing the same market.
6. Interaction between position-level and market-level accounting.
7. Potential cumulative effects caused by integer rounding.

This analysis does not constitute a complete audit of GMX Synthetics V2.

---

## 3. Research Objective

The original hypothesis was:

> Can asymmetric integer rounding during repeated partial position decreases cause `pendingImpactAmount` accounting to drift from the expected proportional value and eventually create an exploitable discrepancy?

The expected attack pattern would require a sequence in which:

```text
partial decrease
        ↓
proportional pending impact
        ↓
rounding
        ↓
remaining pending impact
        ↓
repeated partial decreases
        ↓
cumulative accounting divergence
        ↓
economic extraction
```

The investigation therefore focused on whether rounding could accumulate into a discrepancy between position-level accounting, market-level accounting, or the impact pools.

---

## 4. Technical Background

A position has a `pendingImpactAmount` representing pending price impact associated with that position.

During a partial decrease, only the proportional portion of the pending impact associated with the closed size is realized.

The relevant calculation is conceptually:

```text
proportionalPendingImpactAmount =
    pendingImpactAmount * sizeDeltaUsd / sizeInUsd
```

Because the implementation operates on integers, the result must be rounded.

The implementation additionally chooses the rounding direction based on the sign of the pending impact.

---

## 5. Proportional Pending Impact Calculation

The relevant calculation is:

```text
Precision.mulDiv(
    positionPendingImpactAmount,
    sizeDeltaUsd,
    sizeInUsd,
    positionPendingImpactAmount < 0
)
```

The final argument controls the rounding behavior.

Therefore:

### Positive pending impact

For:

```text
P > 0
```

the proportional amount uses downward rounding:

```text
floor(P × ΔS / S)
```

### Negative pending impact

For:

```text
P < 0
```

the magnitude is rounded upward:

```text
-ceil(|P| × ΔS / S)
```

This creates asymmetric rounding around zero.

---

## 6. Rounding Example

Consider:

```text
P = +80
S = 100
ΔS = 33
```

The mathematically exact proportional amount is:

```text
80 × 33 / 100 = 26.4
```

The integer calculation realizes:

```text
26
```

The remaining pending impact becomes:

```text
80 - 26 = 54
```

instead of the exact mathematical value:

```text
53.6
```

The difference is therefore:

```text
+0.4
```

relative to the exact fractional representation.

---

### Negative Example

Consider:

```text
P = -80
S = 100
ΔS = 33
```

The exact magnitude is:

```text
80 × 33 / 100 = 26.4
```

The implementation realizes:

```text
-27
```

The remaining value becomes:

```text
-80 - (-27) = -53
```

instead of:

```text
-53.6
```

Again, the remaining integer value differs from the exact fractional representation.

This demonstrates that rounding exists and is directionally asymmetric.

However, the existence of rounding alone does not establish a vulnerability.

---

## 7. Position-Level Accounting

After calculating the proportional pending impact, the remaining position value is updated as:

```text
position.pendingImpactAmount =
    position.pendingImpactAmount
    - proportionalPendingImpactAmount
```

Therefore, if:

```text
P = old pending impact
R = realized proportional impact
```

then:

```text
P_new = P - R
```

The rounding error is therefore incorporated directly into the position's remaining integer state.

---

## 8. Market-Level Accounting

The market-level pending impact is updated using the same realized proportional amount.

Conceptually:

```text
totalPendingImpactAmount =
    totalPendingImpactAmount
    - proportionalPendingImpactAmount
```

Therefore:

```text
T_new = T - R
```

where `R` is exactly the same value used when updating the position.

This creates the following accounting relationship:

```text
Position:
P_new = P - R

Global:
T_new = T - R
```

This is important because a rounding difference does not independently appear in the global accounting path.

The same rounded integer is used for both updates.

---

## 9. Core Invariant

The primary accounting invariant examined was:

```text
totalPendingImpactAmount
==
Σ pendingImpactAmount
```

for all relevant live positions within the market.

For a partial decrease:

```text
old position = P
realized = R

new position = P - R
```

and:

```text
old global total = T
new global total = T - R
```

Therefore, assuming the global total was initially consistent with the positions, subtracting the same `R` from both sides preserves the relationship.

This means that proportional rounding alone does not automatically create a position/global accounting mismatch.

---

## 10. Existing Test Evidence

The repository already contains tests covering several relevant scenarios.

### Negative Price Impact / Negative PnL

The test:

```text
test/exchange/DecreasePosition/NegativePriceImpact_NegativePnl.ts
```

contains a 10% position decrease.

The position pending impact changes from approximately:

```text
-0.079999999999999999
```

to:

```text
-0.071999999999999999
```

The test therefore exercises proportional pending-impact reduction during a partial decrease.

---

### Positive Price Impact / Negative PnL

The test:

```text
test/exchange/DecreasePosition/PositivePriceImpact_NegativePnl.ts
```

also performs a 10% position decrease and verifies the resulting pending-impact value.

This provides additional evidence that proportional pending-impact accounting is exercised in both relevant impact configurations.

---

### PairMarket

The test:

```text
test/exchange/PositionPriceImpact/PairMarket.ts
```

contains more extensive pending-impact accounting scenarios.

These include:

* multiple positions
* positive pending impact
* negative pending impact
* partial decreases
* full decreases
* market-level `totalPendingImpactAmount`
* `positionImpactPoolAmount`
* `lentPositionImpactPoolAmount`

The test explicitly checks market-level values after several position operations.

For example, the test verifies changes such as:

```text
-0.08 + 0.04 - 0.08
```

and subsequently tracks the corresponding market-level total.

---

## 11. Multiple Position Analysis

A potentially more interesting scenario is:

```text
Position A = +80
Position B = -80

Global = 0
```

A partial decrease on either position applies the same proportional realized amount to:

1. The affected position.
2. The market-level total.

Therefore, the operation itself does not introduce an independent global discrepancy.

For example:

```text
A_new = A - R_A

Global_new = Global - R_A
```

and similarly:

```text
B_new = B - R_B

Global_new = Global - R_B
```

The accounting relationship therefore remains structurally aligned.

---

## 12. Impact Pool Interaction

Pending impact is not isolated from the impact-pool accounting.

The investigation therefore also considered:

```text
positionImpactPoolAmount
lentPositionImpactPoolAmount
```

The existing `PairMarket` scenarios demonstrate that closing positions can cause impact to move between pending impact and the impact pool, including cases where positive impact is effectively lent because the available impact pool is insufficient.

The tests explicitly track these state transitions.

This is important because a theoretical rounding issue would become security-relevant only if it could ultimately influence an economically valuable state transition.

No such exploitable transition was identified during this investigation.

---

## 13. Attack Hypothesis

The primary attack hypothesis was:

```text
Open position
    ↓
Partial close
    ↓
Rounding
    ↓
Remaining pending impact changes
    ↓
Repeat
    ↓
Rounding accumulates
    ↓
Impact accounting diverges
    ↓
Attacker extracts value
```

The hypothesis was considered for both:

```text
positive pending impact
```

and:

```text
negative pending impact
```

It was also considered in the presence of multiple positions.

---

## 14. Why Rounding Alone Was Not Sufficient

The important distinction is between:

```text
mathematical fractional rounding error
```

and:

```text
accounting inconsistency
```

The implementation can produce a value that differs slightly from the ideal fractional calculation.

However, the same rounded proportional value is subsequently used to update both the position and the market-level pending-impact accounting.

Therefore:

```text
rounding error
≠
accounting leak
```

without an additional state transition that converts the discrepancy into extractable value.

No such conversion path was identified within the investigated scope.

---

## 15. Security Assessment

### Finding

**No confirmed vulnerability.**

### Severity

**Informational / Negative Finding**

### Confidence

**Medium**

The investigation provides evidence that the examined partial-close accounting path remains internally consistent under the tested scenarios.

However, this is not a formal proof of the correctness of every possible pending-impact state transition.

---

## 16. Limitations

This investigation was intentionally scoped to the pending-impact accounting path.

It does not establish the absence of vulnerabilities in:

* order execution
* liquidation
* funding
* borrowing fees
* position fees
* price validation
* oracle handling
* market-token accounting
* collateral accounting
* unrelated impact-pool transitions
* cross-market interactions

The analysis also does not constitute a complete audit of GMX Synthetics V2.

---

## 17. Conclusion

The investigation examined whether asymmetric rounding during partial position decreases could create cumulative pending-impact accounting drift.

The analysis found that:

1. Positive and negative pending impacts use different rounding directions.
2. The rounding can produce a small difference from the exact fractional mathematical result.
3. The rounded proportional amount is applied consistently to the affected position.
4. The same rounded amount is applied to the market-level `totalPendingImpactAmount`.
5. Existing tests cover partial and full position decreases and multiple-position pending-impact accounting.
6. The investigated scenarios did not demonstrate an exploitable divergence.
7. No direct path from the observed rounding behavior to economic value extraction was identified.

### Final Assessment

> **No exploitable accounting inconsistency was identified within the investigated scope.**

The observed asymmetric rounding should therefore be treated as an implementation detail requiring invariant awareness rather than as a confirmed vulnerability.

This investigation is considered complete for the current research scope.

---

## 18. Relevant Repository Locations

```text
test/exchange/DecreasePosition/NegativePriceImpact_NegativePnl.ts

test/exchange/DecreasePosition/PositivePriceImpact_NegativePnl.ts

test/exchange/PositionPriceImpact/PairMarket.ts
```

Relevant implementation areas include:

```text
DecreasePositionCollateralUtils
DecreasePositionUtils
MarketUtils
Precision.mulDiv
```

---

## 19. Research Methodology

The investigation followed the following workflow:

```text
Source-code tracing
        ↓
Identify state variables
        ↓
Trace partial-close calculation
        ↓
Analyze positive/negative rounding
        ↓
Identify accounting invariants
        ↓
Review existing regression tests
        ↓
Compare position/global state transitions
        ↓
Assess exploitability
        ↓
Security conclusion
```

The key principle was to distinguish between an unusual numerical behavior and a security-relevant accounting vulnerability.

---

## 20. Final Research Statement

This research did not identify a confirmed vulnerability.

Its primary value is the documented analysis of a potentially suspicious rounding mechanism and the demonstration that, within the investigated scope, the mechanism did not produce an independently observable accounting leak.

**Research status: CLOSED — no exploitable finding identified.**
