# Ambire Wallet Audit - code4rena

Contest: https://code4rena.com/contests/2023-05-ambire-wallet-invitational

## High

- [H-01 Factory can potentially brick accounts that deploy a initcode that returns empty](./H-01.md)

## Medium

- [M-01 AmbireAccount implementation can be destroyed by privileges](./M-01.md)
- [M-02 `deployAndExecute()` function in Factory should be payable](./M-02.md)
- [M-03 Project may fail to be deployed to chains not compatible with Shanghai hardfork](./M-03.md)
- [M-04 Low level calls to accounts with no code succeed in AmbireAccount](./M-04.md)
- [M-05 AmbireAccount should provide a function to allow to cancel current nonce](./M-05.md)
- [M-06 Callee can intentionally make caller spend more gas than specified in `tryCatchLimit()`](./M-06.md)
- [M-07 Recovery transaction can be replayed after a cancellation](./M-07.md)
- [M-08 Griefing attack on `executeMultiple()` function](./M-08.md)
- [M-09 Attacker can force the failure of transactions that use `tryCatch`](./M-09.md)
- [M-10 Current design won't allow to update reference implementation without breaking counterfactuality](./M-10.md)
- [M-11 Wallet fallback does not fail when there is no handler](./M-11.md)
- [M-12 Fallback handlers can trick users into calling functions of the AmbireAccount contract](./M-12.md)
- [M-13 EIP-712 signatures are not properly implemented](./M-13.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
