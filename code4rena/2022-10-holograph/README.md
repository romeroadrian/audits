# Holograph Audit - code4rena

Contest: https://code4rena.com/contests/2022-10-holograph-contest

## High

- [H-01 Bridged messages may fail in the destination chain and be irrecoverable](./H-01.md)
- [H-02 ETH payout in `PA1D` contract could always fail if at least one receiver reverts](./H-02.md)
- [H-03 The function `revertedBridgeOutRequest` may not revert in all cases and have side effects](./H-03.md)

## Medium

- [M-01 Suspicious gas cost estimate in `_payoutEth()` function of `PA1D` contract](./M-01.md)
- [M-02 The function `getDeploymentBlock()` in the `Holographer` contract returns an incorrect data type](./M-02.md)
- [M-03 Wrong parameter is sent in the `onERC721Received` hook of the `HolographERC721` contract](./M-03.md)
- [M-04 Pseudorandomness could be abused to favor an operator](./M-04.md)
- [M-05 Packed variables can be overlapped when packing a job to storage in the `HolographOperator` contract](./M-05.md)
- [M-06 Unprotected `bridgeIn` and `deployHolographableContract` in `HolographFactory` are susceptible to replay attacks](./M-06.md)
- [M-07 Unclear usage of payable functions involved in the "bridge in" flow](./M-07.md)
- [M-08 ERC20 and ERC721 base contracts have a non-reverting `receive` function](./M-08.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
