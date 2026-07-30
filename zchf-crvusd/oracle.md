# ZCHF Oracle

The recommended oracle returns the USD value of 1 ZCHF. It combines the 3,603-second EMA from the Curve crvUSD/ZCHF pool with Curve's aggregate crvUSD/USD price.

## Oracle construction

$$
P_{\mathrm{ZCHF/USD}}
=
P_{\mathrm{ZCHF/crvUSD}}^{\mathrm{pool\ EMA}}
\times
P_{\mathrm{crvUSD/USD}}^{\mathrm{aggregate}}
$$

In dimensional terms:

$$
\frac{\mathrm{crvUSD}}{\mathrm{ZCHF}}
\times
\frac{\mathrm{USD}}{\mathrm{crvUSD}}
=
\frac{\mathrm{USD}}{\mathrm{ZCHF}}
$$

The crvUSD units cancel, leaving the USD value of 1 ZCHF. All prices use 18 decimals. The wrapper adds no further EMA; the ZCHF leg is smoothed by the pool's built-in `price_oracle()`.

## Recommended contracts

1. [`ProxyOracle`](https://github.com/curvefi/curve-stablecoin/blob/70716d6868642956fa7dfd56c100274148fb0150/curve_stablecoin/price_oracles/proxy/ProxyOracle.vy) - to set the oracle if the ZCHF/crvUSD pool is upgraded to a newer version
2. [`CryptoFromPoolWAgg`](https://github.com/curvefi/curve-stablecoin/blob/70716d6868642956fa7dfd56c100274148fb0150/curve_stablecoin/price_oracles/CryptoFromPoolWAgg.vy) - It reads the pool's existing EMA and performs the crvUSD-to-USD conversion in 1 wrapper.

The flow is:

`Market → ProxyOracle → CryptoFromPoolWAgg → pool and crvUSD aggregator`

## Wrapper configuration

Constructor order: `CryptoFromPoolWAgg(pool, N, borrowed_ix, collateral_ix, agg)`.

| Setting | Value |
|---|---|
| Contract | `CryptoFromPoolWAgg` |
| Pool (`pool`) | [`0x027B40F5917FCd0eac57d7015e120096A5F92ca9`](https://etherscan.io/address/0x027B40F5917FCd0eac57d7015e120096A5F92ca9) |
| Coins (`N`) | 2 |
| Borrowed coin (`borrowed_ix`) | 0 — crvUSD — `0xf939E0A03FB07F59A73314E73794Be0E57ac1b4E` |
| Collateral coin (`collateral_ix`) | 1 — ZCHF — `0xB58E61C3098d85632Df34EecfB899A1Ed80921cB` |
| Pool oracle interface | `price_oracle()` with no argument; detected automatically as `NO_ARGUMENT = true` |
| Pool EMA half-life | 3,603 seconds, approximately 60 minutes |
| crvUSD aggregate (`agg`) | [`0x18672b1b0c623a30089A280Ed9256379fb0E4E62`](https://etherscan.io/address/0x18672b1b0c623a30089A280Ed9256379fb0E4E62) |
| Final output | ZCHF/USD — USD value of 1 ZCHF, scaled by $10^{18}$ |

## Proxy configuration

| Setting | Value |
|---|---|
| Proxy implementation | `ProxyOracle` |
| Proposed initial underlying oracle | New `CryptoFromPoolWAgg` deployment; address to be recorded after deployment |
| Proposed market oracle | New proxy returned by `deploy_proxy_oracle()`; address to be recorded after deployment |
| Proposed factory owner | DAO ownership agent |
| Default replacement deviation | 100 BPS (1%) |

These are deployment recommendations; no wrapper, proxy, or proxy-factory deployment address is specified yet.

To change pools, we can deploy a new `CryptoFromPoolWAgg` configured for the new pool, verify its price and configuration, then do a DAO vote to call `replace_oracle(proxy, new_wrapper, False)`.

## crvUSD depeg behavior

If crvUSD falls against USD while ZCHF's USD value is unchanged, the pool should quote more crvUSD per ZCHF while the aggregate reports fewer USD per crvUSD. Once both inputs reflect the move, these changes largely offset.

$$
1.25\ \frac{\mathrm{crvUSD}}{\mathrm{ZCHF}}
\times
1.00\ \frac{\mathrm{USD}}{\mathrm{crvUSD}}
=
1.25\ \frac{\mathrm{USD}}{\mathrm{ZCHF}}
$$

$$
1.3889\ \frac{\mathrm{crvUSD}}{\mathrm{ZCHF}}
\times
0.90\ \frac{\mathrm{USD}}{\mathrm{crvUSD}}
\approx
1.25\ \frac{\mathrm{USD}}{\mathrm{ZCHF}}
$$

This configuration lets LLAMMA trade and settle directly in crvUSD while valuing ZCHF independently in USD. The crvUSD aggregator removes crvUSD-only movements from the ZCHF price signal and avoids assuming crvUSD is always worth $1. During a downward crvUSD depeg, borrowers can repurchase their nominal debt more cheaply, while temporary dislocations may generate arbitrage volume and LLAMMA fees.
