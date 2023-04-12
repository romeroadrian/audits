## Change `claimable` function visibility

The `claimable` function in the Lottery contract is declared as `external`, but it is used within the contract as an external call. Consider changing the visibility to `public` in order to save gas by executing a local jump within the contract instead of an external call.

## Storage variable is read twice in `claimWinningTicket`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L266

```solidity
unclaimedCount[ticketsInfo[ticketId].drawId][ticketsInfo[ticketId].combination]--;
```

Consider storing `ticketsInfo[ticketId]` as a local variable to prevent a second sload.

## Unneeded storage variable reloading in `claimPerDraw` function

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L139
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L145

The `claimPerDraw` function reads the `unclaimedTickets` storage variable to calculate and claim pending rewards. The function first reads it in line 139 to calculate the referrer side and then needlessly reloads it in line 145 to calculate the player side.

## Unneeded call to `_transferOwnership` in StakedTokenLock constructor

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L17

The base contract `Ownable2Step` constructor will already give ownership to the caller. The call on line 17 can be safely removed to save gas.

## Deposited balance in StakedTokenLock contract can be simplified with token `balanceOf`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L14

The `depositedBalance` storage variable in the `StakedTokenLock` contract is superfluous since this value can be directly obtained from the staking token `balanceOf` function. Consider removing this variable and its usages to save gas.
