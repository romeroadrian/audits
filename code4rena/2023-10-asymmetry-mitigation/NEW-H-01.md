# [ADRIRO-NEW-H-01] VotiumStrategy withdrawal can still be executed with minimal delay

## Summary

Within the mitigation changes, the sponsor has introduced a minimum delay of one epoch for VotiumStrategy withdrawals, in order to mitigate different issues related to the exposure to CVX . The fix contains an edge case which could still be used to make deposits in AfEth with minimal exposure to CVX.

## Impact

Epochs in Convex are synchronized with Curve gauge epochs, which are weekly periods that start each Thursday at 00:00 UTC.

For example, at the time of writing the current epoch is 85. This epoch started at timestamp `1698278400`, which is Thursday Oct 18th at 00:00 UTC. The next epoch, 86, starts on Thursday Oct 25th at 00:00 UTC, which also marks the end of epoch 85.

One of the new changes in the updated code is the introduction of a minimum delay of one epoch in VotiumStrategy withdrawals. Even if the available CVX balance (CVX held by the contract plus any unlockable balance in Convex) is enough to cover the withdrawal, the request is delayed until the next epoch. Locked balances are still implemented as they were before, because any locked balance naturally implies waiting for at least the next epoch.

The updated code can be seen in the `requestWithdraw()` function:

https://github.com/asymmetryfinance/afeth/blob/74f340568480aa03d043e970fcf2578bea037cf6/contracts/strategies/votium/VotiumStrategy.sol#L78-L96

```solidity
78:         uint256 cvxAmount = (_amount * _priceInCvx) / 1e18;
79:         cvxUnlockObligations += cvxAmount;
80: 
81:         uint256 totalLockedBalancePlusUnlockable = unlockable +
82:             trackedCvxBalance;
83: 
84:         if (totalLockedBalancePlusUnlockable >= cvxUnlockObligations) {
85:             withdrawIdToWithdrawRequestInfo[
86:                 latestWithdrawId
87:             ] = WithdrawRequestInfo({
88:                 cvxOwed: cvxAmount,
89:                 withdrawn: false,
90:                 epoch: currentEpoch + 1,
91:                 owner: msg.sender
92:             });
93:             emit WithdrawRequest(msg.sender, cvxAmount, latestWithdrawId);
94: 
95:             return latestWithdrawId;
96:         }
```

If the current available CVX balance (`totalLockedBalancePlusUnlockable`) is enough to cover the withdrawal, the request is scheduled for the next epoch (`currentEpoch + 1`).

Additionally, we note that the same behavior can also happen with locked balances:

https://github.com/asymmetryfinance/afeth/blob/74f340568480aa03d043e970fcf2578bea037cf6/contracts/strategies/votium/VotiumStrategy.sol#L98-L120

```solidity
098:         for (uint256 i = 0; i < lockedBalances.length; i++) {
099:             totalLockedBalancePlusUnlockable += lockedBalances[i].amount;
100:             // we found the epoch at which there is enough to unlock this position
101:             if (totalLockedBalancePlusUnlockable >= cvxUnlockObligations) {
102:                 (, uint32 currentEpochStartingTime) = ILockedCvx(VLCVX_ADDRESS)
103:                     .epochs(currentEpoch);
104:                 uint256 timeDifference = lockedBalances[i].unlockTime -
105:                     currentEpochStartingTime;
106:                 uint256 epochOffset = timeDifference /
107:                     ILockedCvx(VLCVX_ADDRESS).rewardsDuration();
108:                 uint256 withdrawEpoch = currentEpoch + epochOffset;
109:                 withdrawIdToWithdrawRequestInfo[
110:                     latestWithdrawId
111:                 ] = WithdrawRequestInfo({
112:                     cvxOwed: cvxAmount,
113:                     withdrawn: false,
114:                     epoch: withdrawEpoch,
115:                     owner: msg.sender
116:                 });
117: 
118:                 emit WithdrawRequest(msg.sender, cvxAmount, latestWithdrawId);
119:                 return latestWithdrawId;
120:             }
121:         }
```

If the first locked balance (`lockedBalances[0]`) corresponds to the next epoch, and the unlocked amount covers the requested amount (line 101), then the withdrawal will be scheduled for this next epoch (lines 109-116).

Now, as previously mentioned, epochs switch at the start of every Thursday. If we request a withdrawal at the very end of the current epoch, i.e. at most at Wednesday 11:59:59 PM UTC, then the withdrawal can potentially be scheduled for the next epoch which is only 1 second apart. This allows deposits in AfEth with minimal exposure to CVX.

A bad actor can use this to effectively deposit into AfEth, request a withdrawal, and withdraw with a minimum delay that can go as little as one second. As shown before, they would either need the funds to be unlockable (which sets the withdrawal for the next epoch) or to be unlocked in the next epoch.

As this issue still allows deposits with minimal exposure, the reward sandwiching attack and the intrinsic arbitrage due to price deviations are still feasible given the original scenarios. Given the error affects both original H-01 and M-05 issues, I'm assigning this issue a high severity.

## Proof of Concept

Let's say that the attacker makes a deposit such that `N` amount of CVX tokens are bought. To simplify the example, let's also say that the unlockable amount of tokens in Convex is greater than `N`. The attacker executes this at timestamp `1698278399`.

1. The attacker deposits into AfEth. Their deposited share consists of `N` CVX tokens.
2. The attack immediately requests a withdrawal. As the unlockable amount is enough to cover for the `N` tokens, the request is scheduled for the next epoch. Current epoch associated with timestamp `1698278399` is 85, which means the withdrawal is scheduled for epoch 86.
3. The attacker waits one second, it is now `1698278400`.
4. The attacker calls `withdraw()`, the epoch for the current timestamp `1698278400` is 86. Withdrawal is allowed and the attacker removes their share in the protocol.
5. The attacker effectively executed the deposit and withdraw cycle with an exposure of just one second.

## Recommendation

The issue can be fixed by reconsidering potential requests to withdraw near the end of the period. For example, one potential solution will be to check if the current timestamp is after the second half of the current period. If so, then schedule the withdrawal not for the next epoch, but for the epoch after the next epoch (i.e. with a delay of two periods).

Note that this should be **considered in both scenarios**, when the current unlockable amount covers the withdrawal and when the locked balances are iterated to find the epoch that releases the needed funds.

A pseudo-code of the algorithm can summed as:

```
1. Check if `totalLockedBalancePlusUnlockable >= cvxUnlockObligations`.
2. If so, set `withdrawEpoch = currentEpoch + 1`.
3. If not, loop through `lockedBalances`:
  3a. If `totalLockedBalancePlusUnlockable >= cvxUnlockObligations`, calculate epoch for `lockedBalances[i].unlockTime` and set that result as `withdrawEpoch`.
4. If `withdrawEpoch` is not found, revert.
5. If `withdrawEpoch.startTime - block.timestamp < epochDuration / 2`, then set `withdrawEpoch += 1`.
6. Schedule withdrawal for `withdrawEpoch`.
```
