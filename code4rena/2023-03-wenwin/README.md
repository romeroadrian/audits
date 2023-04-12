# Wenwin Audit - code4rena

Contest: https://code4rena.com/contests/2023-03-wenwin-contest

## High

- [H-01 Winner is unable to claim winnings at the very end of the claimable period](./H-01.md)
- [H-02 Loss of funds when buying tickets with no frontend](./H-02.md)

## Medium

- [M-01 Ticket buyer can set arbitrary frontend address and potentially buy tickets at a discounted rate](./M-01.md)
- [M-02 Calculation in `calculateNewProfit` function is broken when jackpot is not won](./M-02.md)
- [M-03 Overflow risk in `calculateExcessPot` function](./M-03.md)
- [M-04 Protocol fails to support arbitrary token for rewards](./M-04.md)
- [M-05 Fixed rewards in DAI (or similar token) can potentially overflow when being packed](./M-05.md)
- [M-06 Malicious user can frontrun the selling or transferring of a ticket to claim the rewards](./M-06.md)
- [M-07 Owner can silently withdraw funds just before deadline in `StakedTokenLock`](./M-07.md)
- [M-08 Ticket minting should use `safeMint`](./M-08.md)
- [M-09 `swapSource` in `RNSourceController` contract can be frontrunned and eventually lead to a DoS](./M-09.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
