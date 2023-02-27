# Debt DAO Audit - code4rena

Contest: https://code4rena.com/contests/2022-11-debt-dao-contest

## High:

- [H-01 Whitelisted functions aren't scoped to revenue contracts and may lead to unnoticed calls due to selector clashing](./H-01.md)
- [H-02 Revenue tokens coming from a particular contract can be claimed with settings from another contract that would benefit a particular party](./H-02.md)
- [H-03 Uninitialized Revenue settings can be used to claim revenue that goes 100% to the treasury](./H-03.md)
- [H-04 Potential DoS in `claimRevenue` function with ETH token](./H-04.md)
- [H-05 Potential DoS in `updateOutstandingDebt` function in LineOfCredit contract](./H-05.md)
- [H-06 Potential DoS in `accrueInterest` function in LineOfCredit contract](./H-06.md)
- [H-07 Mutual consent may lead to ETH deposit lost in `addCredit` function of LineOfCredit contract](./H-07.md)
- [H-08 Mutual consent may lead to ETH deposit lost in `increaseCredit` function of LineOfCredit contract](./H-08.md)
- [H-09 Unprotected access to `depositAndRepay` function in LineOfCredit contract](./H-09.md)
- [H-10 Potential DoS in `borrow` function in LineOfCredit contract](./H-10.md)
- [H-11 Closing a paid credit may lead to incorrectly stepping the queue](./H-11.md)
- [H-12 Potential DoS while repaying debt in LineOfCredit contract](./H-12.md)
- [H-13 Potential DoS when closing a credit nominated in ETH in the LineOfCredit contract](./H-13.md)
- [H-14 Reentrancy attack when closing a credit in the LineOfCredit contract](./H-14.md)
- [H-15 Closing an unexisting credit can overflow the credit count variable in the LineOfCredit contract](./H-15.md)
- [H-16 Line of credit status can be set to REPAID even if having credits with debt](./H-16.md)
- [H-17 `useAndRepay` function can be used to underflow the principal debt of a credit](./H-17.md)

## Medium:

- [M-01 Missing relation between Revenue contract and token in `claimRevenue` function of Spigot contract](./M-01.md)
- [M-02 `operate` function in Spigot can be used to make arbitrary calls on behalf of the Spigot](./M-02.md)
- [M-03 `LineLib.receiveTokenOrETH` can receive a greater ETH amount than expected](./M-03.md)
- [M-04 The `accrueInterest` function in LineOfCredit contract doesn't check for null/deleted credits](./M-04.md)
- [M-05 FIFO repayment invariant isn't technically correct when a credit is re-borrowed](./M-05.md)
- [M-06 `close` function in LineOfCredit can't be called by the lender if credit is in ETH](./M-06.md)
- [M-07 Functions that involve a trade in the SpigotedLine contract should link trade output to credit debt](./M-07.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
