# Particle Leverage AMM Protocol Audit - code4rena

Contest: https://code4rena.com/audits/2023-12-particle-leverage-amm-protocol-invitational

## High

- [H-01 Incorrect fee calculation may lead to borrower overpaying](./H-01.md)
- [H-02 Liquidation condition should not factor the liquidation reward into the premiums](./H-02.md)

## Medium

- [M-01 Modifying the loan term setting can default existing loans](./M-01.md)
- [M-02 Owners of LPs can be dosed when removing their position](./M-02.md)
- [M-03 Add premium doesn't collect fees](./M-03.md)
- [M-04 Increase liquidity in close position may not cover original borrowed liquidity](./M-04.md)
- [M-05 Liquidator has no incentives to execute a favorable trade to the borrower](./M-05.md)
- [M-06 LP owner cannot control slippage while managing their position](./M-06.md)
- [M-07 ERC20 implementations may revert on zero approval](./M-07.md)
- [M-08 Dangerous use of deadline parameter](./M-08.md)
- [M-09 Non-upgradeable contract may cause storage clashing during upgrades](./M-09.md)
- [M-10 Zero amount token transfers may cause a denial of service during liquidations](./M-10.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
