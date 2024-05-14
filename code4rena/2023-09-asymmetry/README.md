# Asymmetry Finance afETH Audit - code4rena

Contest: https://code4rena.com/contests/2023-09-asymmetry-finance-afeth-invitational

## High

- [H-01 Direct depositors in the Votium strategy will lose rewards when these are routed to SafEth](./H-01.md)
- [H-02 AfEth deposits could use price data from an invalid Chainlink response](./H-02.md)
- [H-03 Inflation attack in VotiumStrategy](./H-03.md)
- [H-04 Zero amount withdrawals of SafEth or Votium will brick the withdraw process](./H-04.md)
- [H-05 AfEth price calculation doesn't factor locked tokens held in contract balance](./H-05.md)
- [H-06 Lack of access control and value validation in the reward flow exposes functions to public access](./H-06.md)
- [H-07 Missing slippage control when directly interacting with the VotiumStrategy contract](./H-07.md)

## Medium

- [M-01 Percentage calculation could leave unused ETH leftovers in AfEth deposit](./M-01.md)
- [M-02 Missing slippage control while depositing rewards in SafEth and VotiumStrategy](./M-02.md)
- [M-03 Missing storage gap in AbstractStrategy may affect contract upgradeability](./M-03.md)
- [M-04 Missing deadline check for AfEth actions](./M-04.md)
- [M-05 Inefficient reward compounding in Votium Strategy](./M-05.md)
- [M-06 Reward sandwiching in VotiumStrategy](./M-06.md)
- [M-07 Uninitialized protocol fee address could cause loss of funds](./M-07.md)
- [M-08 Inefficient reward split could cause unbalanced ratio and favor SafEth staking](./M-08.md)
- [M-09 VotiumStrategy withdrawal queue fails to consider available unlocked tokens causing different issues in the withdraw process](./M-09.md)
- [M-10 Forced relock in VotiumStrategy withdrawal causes denial of service if Convex locking contract is shutdown](./M-10.md)
- [M-11 Snapshot delegation cannot be cleared or modified](./M-11.md)
- [M-12 `cvxPerVotium()` calculation will return zero if all CVX tokens are pending withdrawal as obligations](./M-12.md)
- [M-13 Feature to recover stuck tokens is too permissive and could be used to remove CVX tokens](./M-13.md)
- [M-14 Swap functionality to sell rewards is too permissive and could cause accidental or intentional loss of value](./M-14.md)
- [M-15 AfEth collaterals cannot be balanced after ratio is changed](./M-15.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)

## Analisys

[Analisys](./analysis.md)
