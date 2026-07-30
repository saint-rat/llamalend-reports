# ZCHF/crvUSD Market Overview

This page summarizes the recommended parameters for a crvUSD debt market using ZCHF as collateral. The linked reports contain the supporting simulations and methodology.

## Market settings

| Parameter | Selection |
|---|---:|
| Borrowed asset | crvUSD |
| Collateral | ZCHF |
| Amplification ($A$) | 180 |
| Base fee | 0.05% |
| Admin percentage | 10% |

## Risk settings

| Parameter | Selection |
|---|---:|
| Liquidation discount | 2.3% |
| Loan discount | 4.3% |
| Initial borrow cap | 500,000 crvUSD |
| Approximate maximum LTV at 4 bands | 94.6% |

The maximum LTV is an estimate for a position distributed across 4 bands, not a separate market parameter.

## Oracle

| Parameter | Selection |
|---|---|
| Price output | ZCHF/USD |
| Pool EMA half-life | 3,603 seconds, approximately 60 minutes |
| Curve ZCHF/crvUSD pool | `0x027B40F5917FCd0eac57d7015e120096A5F92ca9` |
| crvUSD/USD aggregator | `0x18672b1b0c623a30089A280Ed9256379fb0E4E62` |
| Wrapper | `CryptoFromPoolWAgg` |
| Proposed flow | `Market → ProxyOracle → CryptoFromPoolWAgg → pool and crvUSD aggregator` |
| Default proxy replacement deviation | 100 BPS (1%) |

The oracle returns the USD value of 1 ZCHF while LLAMMA continues to trade and settle in crvUSD.

## Monetary policy

| Parameter | Selection |
|---|---:|
| Contract | `HyperbolicMP` |
| Target utilization | 80% |
| Target borrower APR | 5.2778% |
| Supplier APR at target utilization | 3.8% |
| Low ratio | 0.3× |
| High ratio | 10× |
| Rate shift | 0% |
