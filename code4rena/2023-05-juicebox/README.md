# Juicebox Buyback Delegate Audit - code4rena

Contest: https://code4rena.com/contests/2023-05-juicebox-buyback-delegate

## High

- [H-01 Delegate uses incorrect weight ratio to calculate token count in `payParams()`](./H-01.md)

## Medium

- [M-01 Delegate doesn't verify payed ETH value matches amount in parameter](./M-01.md)
- [M-02 Delegate architecture forces users to set zero slippage](./M-02.md)
- [M-03 Delegate uses incorrect parameter for the token amount](./M-03.md)
- [M-04 Type casting can potentially change semantics of swap operation](./M-04.md)
- [M-05 Delegate should check that terminal is operating with ETH as the terminal token](./M-05.md)
- [M-06 Swaps in Uniswap V3 may be partial](./M-06.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
