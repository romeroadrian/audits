# Biconomy - Smart Contract Wallet Audit - code4rena

Contest: https://code4rena.com/contests/2023-01-biconomy-smart-contract-wallet-contest/

## High

- [H-01 SmartAccount implementation can be destroyed by a bad actor](./H-01.md)
- [H-02 `tokenGasPriceFactor` in `FeeRefund` struct can be malleable in calls to `execTransaction`](./H-02.md)
- [H-03 SmartAccount authorization can be bypassed using a contract signature](./H-03.md)
- [H-04 SmartAccount wallet creation can be backdoored](./H-04.md)
- [H-05 Attacker can gain control of counterfactual wallet](./H-05.md)
- [H-06 Proxy creation isn't check in `deployWallet` function of `SmartAccountFactory` contract](./H-06.md)

## Medium

- [M-01 Loss of funds if mistakenly sent to the implementation contract](./M-01.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)