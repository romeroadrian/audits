# [ADRIRO-NEW-H-03] Invalid operation in `withdrawStuckTokens()` will break CVX balance tracking in VotiumStrategy

## Summary

The updated code for `withdrawStuckTokens()` contains an update to the `trackedCvxBalance` variable that will break CVX accounting in the VotiumStrategy contract, leading to multiple severe consequences.

## Impact

To mitigate a potential withdrawal of CVX tokens using `withdrawStuckTokens()`, the sponsor has updated the implementation to handle the special case of CVX.

https://github.com/asymmetryfinance/afeth/blob/fe543431677df8273cbb7d2c92f1253956f633bc/contracts/strategies/votium/VotiumStrategyCore.sol#L231-L240

```solidity
236:     function withdrawStuckTokens(address _token) public onlyOwner {
237:         uint256 tokenBalance = IERC20(_token).balanceOf(address(this));
238:         if (_token == CVX_ADDRESS) {
239:             if (tokenBalance <= trackedCvxBalance) revert InvalidAmount();
240:             tokenBalance -= trackedCvxBalance;
241:         }
242: 
243:         IERC20(_token).safeTransfer(msg.sender, tokenBalance);
244:         if (_token == CVX_ADDRESS) trackedCvxBalance -= tokenBalance;
245:     }
```

As we can see in the previous snippet of code, lines 238-241 handle the special case of CVX by subtracting the `trackedCvxBalance` (CVX owned by the protocol depositors) amount to the `tokenBalance` amount, to just remove the excess of CVX and avoid withdrawing protocol owned funds.

However, line 244 updates the `trackedCvxBalance` by subtracting the `tokenBalance` amount. This is wrong, as it is updating the tracked CVX balance by subtracting the excess of tokens.

This will completely break the internal CVX accounting, which tracks user deposits in VotiumStrategy. This is a core variable of the contract, which has impact in different places:

- It is used to calculate `cvxInSystem()`, which also affects `cvxPerVotium()` **that is used to calculate deposits, withdrawals, and the price itself of vAfEth**.
- The tracked balance is also used in `requestWithdraw()`, to calculate the withdrawal epoch based on the requested withdrawal amount.
- In `relock()`, to calculate the amount of tokens which should be relocked in Convex.

In summary, any subtracted amount to `trackedCvxBalance` means decreasing the amount of CVX tokens owned by the depositors, which translates to loss of funds. As we can see in the next section, this can even lead to updating `trackedCvxBalance` to zero.

## Proof of Concept 

Let's say that `trackedCvxBalance = 100`.

1. An user (could be an attacker or anyone accidentally) donates 100 CVX tokens to the VotiumStrategy.
2. The owner calls `withdrawStuckTokens(CVX)`.
3. The balance is `tokenBalance = IERC20(_token).balanceOf(address(this)) = 200`.
4. Because `_token == CVX_ADDRESS`, the implementation does `tokenBalance -= trackedCvxBalance` which results in `tokenBalance = 100`.
5. 100 tokens are transferred to the owner.
6. Finally, the implementation executes `trackedCvxBalance -= tokenBalance`, leaving `trackedCvxBalance = 0`.

## Recommendation

The implementation of `withdrawStuckTokens()` should not update the `trackedCvxBalance`, only use it to calculate the excess of CVX tokens that can be transferred out of the contract.

```diff
    function withdrawStuckTokens(address _token) external onlyOwner {
        uint256 tokenBalance = IERC20(_token).balanceOf(address(this));
        if (_token == CVX_ADDRESS) {
            if (tokenBalance <= trackedCvxBalance) revert InvalidAmount();
            tokenBalance -= trackedCvxBalance;
        }

        IERC20(_token).safeTransfer(msg.sender, tokenBalance);
-       if (_token == CVX_ADDRESS) trackedCvxBalance -= tokenBalance;
    }
```
