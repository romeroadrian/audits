# Caviar Private Pools Audit - code4rena

Contest: https://code4rena.com/contests/2023-04-caviar-private-pools

## Medium

- [M-01 Private Pool creation in Factory contract should check implementation is initialized](./M-01.md)
- [M-02 Flash loans can be used by attackers to claim rewards over token ownership](./M-02.md)
- [M-03 `changeFeeQuote` will fail for low decimal ERC20 tokens](./M-03.md)
- [M-04 Checking for interface support while fetching royalties might revert operation](./M-04.md)
- [M-05 `sell` function in EthRouter doesn't check pool base token is ETH](./M-05.md)
- [M-06 Incorrect ETH amount sent to `change` function in EthRouter](./M-06.md)
- [M-07 Flash loan fee is incorrect in Private Pool contract](./M-07.md)
- [M-08 Deposit function should be only accessible by the pool owner](./M-08.md)
- [M-09 Royalty fees may be incorrectly accounted in the `buy` and `sell` functions of the PrivatePool contract](./M-09.md)
- [M-10 Users cannot control slippage in buy and sell operations](./M-10.md)
- [M-11 PrivatePool creation may be front-runned or be recreated by an attacker during a chain reorganization](./M-11.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
