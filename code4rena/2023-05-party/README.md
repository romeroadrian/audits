# Party DAO Audit - code4rena

Contest: https://code4rena.com/contests/2023-05-party-dao-invitational

## High

- [H-01 Reentrancy guard in `rageQuit()` can be bypassed](./H-01.md)
- [H-02 Tokens with multiple entry points can lead to loss of funds in `rageQuit()`](./H-02.md)

## Medium

- [M-01 PartyGovernanceNFT implementation constructor is allowed to receive ETH](./M-01.md)
- [M-02 Rage quit modifications should be limited to provide stronger guarantees to party members](./M-02.md)
- [M-03 Rage quit can also be used by party members to exit their position when they don't agree with proposals](./M-03.md)
- [M-04 Burning an NFT can be used to block voting](./M-04.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
