# ZCHF Monetary Policy

## Recommendation

Use Curve's fixed [`HyperbolicMP`](https://github.com/curvefi/curve-stablecoin/blob/70716d6868642956fa7dfd56c100274148fb0150/curve_stablecoin/mpolicies/v2/HyperbolicMP.vy).

- Target utilization = 80%
- Target supplier APR = 3.8%
- Target borrower APR = 5.2778%
- Low ratio = 0.1×
- High ratio = 20×
- Rate shift = 0%

A custom monetary-policy contract is not required.

## Contract configuration

Constructor order:

`HyperbolicMP(controller, target_utilization, target_rate, low_ratio, high_ratio, rate_shift)`

| Setting | Human value | Constructor value |
|---|---:|---:|
| Controller | Market controller | `<CONTROLLER_ADDRESS>` |
| Target utilization | 80% | `800000000000000000` |
| Target borrower APR | 5.2778% | `1673572355` per second |
| Low ratio | 0.1× target rate at 0% utilization | `100000000000000000` |
| High ratio | 20× target rate at 100% utilization | `20000000000000000000` |
| Rate shift | 0% | `0` |

The 0.1× low ratio keeps borrowing inexpensive when the market is underused. The 20× high ratio increases rates sharply as available liquidity is exhausted.

## Rate curve

![Borrower and supplier APR plotted against market utilization](monetary_policy_assets/rate_curve.svg)

The chart is capped at 20% APR; higher rates are shown in the table below.

| Utilization | Borrower APR | Supplier APR |
|---:|---:|---:|
| 0% | 0.5278% | 0.0000% |
| 25% | 0.9407% | 0.2117% |
| 50% | 1.7570% | 0.7906% |
| 70% | 3.3518% | 2.1117% |
| 80% | 5.2778% | 3.8000% |
| 85% | 7.1325% | 5.4564% |
| 90% | 10.6434% | 8.6212% |
| 95% | 19.8186% | 16.9449% |
| 99% | 57.2084% | 50.9727% |
| 100% | 105.5556% | 95.0000% |

At 80% utilization, integer rounding produces 3.80000000% supplier APR.

The curve reaches 105.56% borrower APR at 100% utilization, below the Controller's 138.63% cap, so the cap does not bind.

## Deployment

The Controller is immutable in `HyperbolicMP`. For a new market, predict the Controller address, deploy the policy against it, create the market, and verify that the returned Controller matches the prediction.

The constructor values are listed in the table above and reproduced in the accompanying `monetary_policy_config.json` file.

The Controller address remains the only unresolved constructor input.
