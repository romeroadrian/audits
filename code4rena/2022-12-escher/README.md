# Escher Audit - code4rena

Contest: https://code4rena.com/contests/2022-12-escher-contest

## High:

- [H-01 Sale contracts can be bricked if any other minter mints a token with an id that overlaps the sale](./H-01.md)
- [H-02 `LPDA` price can underflow the price due to bad settings and potentially brick the contract](./H-02.md)

## Medium:

- [M-01 There is no way to cancel a sale or withdraw ETH unless all NFTs are sold](./M-01.md)
- [M-02 Token ID 0 can't be sold using the Sales contracts](./M-02.md)
- [M-03 Sale end time is not checked in `LPDA` contract](./M-03.md)
- [M-04 Creator can still "cancel" a sale after it has started by revoking permissions in `OpenEdition` contract](./M-04.md)
- [M-05 `LPDA` sale should be able to be finalized when the end time is reached](./M-05.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
