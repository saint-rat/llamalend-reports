# ZCHF Methodology

This page defines the terms and formulas shared by the ZCHF simulation reports. Rates are fractions inside formulas: $0.023=2.3\%$.

## Simulation parameters

**$A$ — LLAMMA amplification**

Approximate band width is $1/A$, so a larger $A$ produces narrower bands.

**$N$ — number of bands**

Number of bands across which the simulated collateral is distributed.

**EMA half-life**

Time required for the weight of an earlier oracle observation to halve. Oracle price is ZCHF/USD.

**Base fee**

Minimum fee charged on a simulated LLAMMA swap.

**Dynamic-fee multiplier**

Multiplier for the additional distance-based fee when the oracle moves outside a band's price range.

**External arbitrage fee**

Assumed market-side execution cost applied before an arbitrage trade.

**Sample or window**

1 contiguous historical price path of the stated duration.

## Oracle EMA

$$
d_t=2^{-\Delta t/T_{1/2}},
\qquad
P^{\mathrm{EMA}}_t
=d_tP^{\mathrm{EMA}}_{t-1}+(1-d_t)P^{\mathrm{market}}_t
$$

$P^{\mathrm{market}}_t$ is the market input, $P^{\mathrm{EMA}}_t$ is the oracle value, $T_{1/2}$ is the EMA half-life, and $\Delta t$ is elapsed time. $d_t$ is the weight retained from the previous oracle value. A longer half-life makes the oracle smoother but slower to react.

## Simulated loss

$$
L_i=1-\frac{V_{i,\mathrm{end}}}{V_{i,\mathrm{start}}}
$$

$V_{i,\mathrm{start}}$ and $V_{i,\mathrm{end}}$ are the starting and ending values of simulated window $i$, expressed in the borrowed asset. $L_i$ is the value lost during soft liquidation.

- **Average loss:** mean across all simulated windows.
- **Median loss:** middle simulated loss.
- **Worst $m$ loss:** mean of the $m$ largest losses, not the $m$-th largest observation.
- **Maximum loss:** single largest simulated loss.

## Band value and effective haircut

### Analytical coefficient

$$
C(A,N)=\frac{1}{N}\sum_{k=0}^{N-1}
\left(\frac{A-1}{A}\right)^{k+\frac{1}{2}}
$$

$k$ indexes the bands from $0$ to $N-1$. $C(A,N)$ estimates the initial value of collateral distributed evenly across $N$ bands, relative to its oracle value. The EMA and A/fee sweep reports use this analytical coefficient.

### Initialized-model coefficient

$$
C_{\mathrm{model}}=\frac{V_{\mathrm{start}}}{q_0P_0}
$$

$q_0$ is the deposited collateral and $P_0$ is its opening oracle price. The discounts report measures $C_{\mathrm{model}}$ from the initialized simulator, so it can differ slightly from $C(A,N)$.

### Effective haircut

$$
H(L,C)=1-(1-L)C
$$

$H$ combines the initial multi-band valuation haircut with the loss incurred during the simulation. Use $C(A,N)$ for sweep estimates and $C_{\mathrm{model}}$ for the final calibration.

## Liquidation and loan discounts

### Liquidation discount

$$
D_{\mathrm{liq}}\geq H_{\mathrm{calibration}}
$$

$H_{\mathrm{calibration}}$ is the chosen tail or maximum effective haircut. $D_{\mathrm{liq}}$ is the protocol liquidation discount selected at or above it, then rounded to a practical parameter value.

### Loan discount

$$
D_{\mathrm{loan}}=D_{\mathrm{liq}}+B
$$

$B$ is the added loan buffer for risks not fully represented by historical simulation. $D_{\mathrm{loan}}$ reduces maximum borrowing and must be larger than $D_{\mathrm{liq}}$.

## Maximum LTV

$$
\mathrm{LTV}_{\max}
\approx 1-D_{\mathrm{loan}}-\frac{N}{2A}
$$

Maximum LTV is the approximate maximum debt divided by oracle-valued collateral for a position spread across $N$ bands.

$$
1-0.043-\frac{4}{2\times180}
=0.94588889
\approx 94.6\%
$$

The executable on-chain maximum may be slightly lower because of rounding, current oracle and band state, extra health, and available market liquidity.

## Price declines

$$
d=1-\frac{P_{\mathrm{end}}}{P_{\mathrm{start}}}
$$

$d$ is the decline from an earlier price $P_{\mathrm{start}}$ to a later price $P_{\mathrm{end}}$.

**Exact 24-hour decline**

Return between prices exactly 24 hours apart.

**24-hour drawdown**

Largest peak-to-later-trough decline contained within a period of no more than 24 hours.

**Daily fixing decline**

Return between official reference-rate observations on adjacent calendar days.

## Monetary policy

### Utilization

$$
u=
\frac{\mathrm{total\ debt}}
{\mathrm{available\ balance}+\mathrm{total\ debt}-\mathrm{admin\ fees}}
$$

$u$ is the fraction of net market assets lent to borrowers, capped at $1$.

### Hyperbolic borrower rate

$$
r_{\mathrm{borrower}}(u)
=
r_0\left(r_{-\infty}+\frac{A_h}{u_\infty-u}\right)+s
$$

$r_0$ is the target borrower rate at target utilization $u_0$. $A_h$, $u_\infty$, and $r_{-\infty}$ are coefficients derived by the contract from $u_0$, the low ratio $\alpha$, and the high ratio $\beta$. $s$ is the rate shift. With $s=0$, the curve satisfies $r(0)=\alpha r_0$, $r(u_0)=r_0$, and $r(1)=\beta r_0$, before the Controller's rate cap.

### Supplier APR

$$
r_{\mathrm{supplier}}(u)
=
r_{\mathrm{borrower}}(u)\,u\,q
$$

$q$ is the share of borrower interest distributed to suppliers. The market reports these rates as simple APR, not compounded APY.

## Fresh-market borrow cap

For a market without live borrower positions, the fresh-market method sizes the cap from stressed executable liquidity and a conservative borrower assumption.

### Executable liquidity

$$
\mathrm{PI}_{\max}=D_{\mathrm{liq}},
\qquad
X_{\mathrm{stress}}
=\frac{1}{\lambda_{\mathrm{liq}}}
\max_{S\in\mathcal D}\sum_{i\in S}x_i^*
$$

$x_i^*$ is executable collateral on route $i$, measured at the full liquidation discount, and $\lambda_{\mathrm{liq}}$ is the liquidity-contraction factor. $\mathcal D$ contains compatible route sets without shared pools or liquidity. Capacities within a compatible set can be added; overlapping aggregator routes cannot be double-counted.

Only liquidity directly executable against the debt asset on the market's chain is included. Cross-chain and CEX liquidity require a separate methodology for transfer latency, inventory exposure, fill probability, basis, and venue risks.

### Hard-liquidation stress

$$
L_{\mathrm{hard}}=1-D_{\mathrm{liq}},
\qquad
s_{\mathrm{trigger}}=1-\frac{L}{L_{\mathrm{hard}}},
\qquad
\phi=\max\left(0,\frac{s-s_{\mathrm{trigger}}}{s}\right)
$$

$L$ is the assumed borrower LTV, $s$ is the selected downside shock, and $\phi$ is the fraction of that stress beyond the borrower's buffer. $\phi$ is a severity factor, not a probability.

### Cap

$$
P_s=P_0(1-s),
\qquad
C_{\mathrm{fresh}}
=\frac{X_{\mathrm{stress}}P_sL}{(1-\eta)\phi}
$$

$P_0$ is the current collateral price, $P_s$ is its shocked price, and $\eta$ is assumed soft-liquidation efficiency. Once borrowers exist, this approximation should be replaced by analysis of live positions and liquidation exposure.
