# Asymmetry Audit - code4rena

Contest: https://code4rena.com/contests/2023-03-asymmetry-contest

## High

- [H-01 `ethPerDerivative` can yield different prices for the Reth derivative during the call to `stake`](./H-01.md)
- [H-02 `WstEth` derivative assumes a ~1=1 peg of stETH to ETH](./H-02.md)
- [H-03 Rebalancing strategy is highly inefficient and will lead to losses](./H-03.md)

## Medium

- [M-01 Reth withdrawal may fail due to insufficient collateral](./M-01.md)
- [M-02 Reth `poolCanDeposit` is missing the validation to check if Rocket Pool deposits are enabled](./M-02.md)
- [M-03 Reth `poolPrice` calculation may overflow](./M-03.md)
- [M-04 Inaccurate minimum exchange amount out calculation in SfrxEth withdrawal](./M-04.md)
- [M-05 WstEth and SfrxEth derivatives don't verify staking requirements](./M-05.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
