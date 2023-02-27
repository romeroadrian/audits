# Non Fungible Trading Audit - code4rena

Contest: https://code4rena.com/contests/2022-11-non-fungible-trading-contest

## High:

- [H-01  ETH payments may fail to be refunded and will be lost in the Exchange contract](./H-01.md)
- [H-02  Exchange refund operation will return all ETH stored in the contract instead of the remaining amount from the exchange operation](./H-02.md)
- [H-03  Pool contract uses proxies but doesn't implement the initializer function](./H-03.md)

## Medium:

- [M-01  Order listing time can be arbitrarily set by an attacker to modify matching role](./M-01.md)
- [M-02  Cached EIP-712 domain separator may lead to replay attacks](./M-02.md)
- [M-03  Reentrancy attack can be used to externally call the `_execute` function in the Exchange contract](./M-03.md)

## Gas

- [Gas Report](./gas.md)

## Low/QA

- [Low/QA Report](./low.md)
