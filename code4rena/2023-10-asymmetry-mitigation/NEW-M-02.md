# [ADRIRO-NEW-M-02] AfEth withdrawals are delayed even if the vAfEth withdrawal amount is zero

## Summary

While zero amount withdrawals of SafEth have been prevented, the updated codebase still executes the withdrawal process for zero amount withdrawals of vAfEth, creating an unnecessary delay in AfEth withdrawals.

## Impact

In AfEth, the withdrawal process is initiated by requesting a withdrawal using `requestWithdraw()`. As we can see in the implementation, even if the resulting amount of vAfEth (`votiumWithdrawAmount`) is zero, the function still calls `VotiumStrategy::requestWithdraw()`:

https://github.com/asymmetryfinance/afeth/blob/74f340568480aa03d043e970fcf2578bea037cf6/contracts/AfEth.sol#L214-L219

```solidity
214:         uint256 votiumWithdrawAmount = (withdrawRatio *
215:             trackedvStrategyBalance) / 1e18;
216:         uint256 withdrawTimeBefore = withdrawTime(votiumWithdrawAmount);
217:         uint256 vEthWithdrawId = AbstractStrategy(vEthAddress).requestWithdraw(
218:             votiumWithdrawAmount
219:         );
```

Drilling down into `VotiumStrategy::requestWithdraw()`, we can that even for a zero amount the request must undergo a delay:

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

The calculated `cvxAmount` in line 78 will be zero, and the rest of the algorithm is executed. This means that even if the amount is zero, the request will be delayed between 1 and 16 epochs, depending on the current queued withdrawals.

This means that a withdrawal in AfEth which consists only of SafEth will still need to be subject to an unnecessary delay due to the withdrawal request in VotiumStrategy.

## Recommendation

Avoid executing the withdrawal request in VotiumStrategy if `votiumWithdrawAmount` is zero. 
