# [ADRIRO-NEW-M-03] Safe approval could lead to a denial of service in VotiumStrategy

## Summary

The introduction of the SafeERC20 wrapper may lead to an accidental denial of service due to how the `safeApprove()` function works internally.

## Impact

The updated codebase uses the [SafeERC20](https://docs.openzeppelin.com/contracts/5.x/api/token/erc20#SafeERC20) wrapper provided by the OpenZeppelin contracts library to handle ERC20 interaction in the VotiumStrategyCore contract. This was presumably added to provide safer support for the `applyRewards()` function, since this function needs to handle arbitrary tokens.

However, the SafeERC20 wrapper has been also applied as part of the CVX handling in the VotiumStrategyCore contract. This can be seen in the implementations of `depositRewards()` and `sellCvx()`:

```solidity
219:     function depositRewards(
220:         uint256 _amount,
221:         uint256 _cvxMinout
222:     ) public payable onlyManager {
223:         uint256 cvxAmount = buyCvx(_amount);
224:         if (cvxAmount < _cvxMinout) revert MinOut();
225:         IERC20(CVX_ADDRESS).safeApprove(VLCVX_ADDRESS, cvxAmount);
226:         ILockedCvx(VLCVX_ADDRESS).lock(address(this), cvxAmount, 0);
227:         trackedCvxBalance -= cvxAmount;
228:         emit DepositReward(cvxPerVotium(), _amount, cvxAmount);
229:     }
```

```solidity
276:     function sellCvx(
277:         uint256 _cvxAmountIn
278:     ) internal returns (uint256 ethAmountOut) {
279:         address CVX_ETH_CRV_POOL_ADDRESS = 0xB576491F1E6e5E62f1d8F26062Ee822B40B0E0d4;
280:         // cvx -> eth
281:         uint256 ethBalanceBefore = address(this).balance;
282:         IERC20(CVX_ADDRESS).safeApprove(CVX_ETH_CRV_POOL_ADDRESS, _cvxAmountIn);
283: 
284:         ICrvEthPool(CVX_ETH_CRV_POOL_ADDRESS).exchange_underlying(
285:             1,
286:             0,
287:             _cvxAmountIn,
288:             0 // this is handled at the afEth level
289:         );
290:         ethAmountOut = address(this).balance - ethBalanceBefore;
291:         trackedCvxBalance -= _cvxAmountIn;
292:     }
```

The implementation of `safeApprove()` has a very important detail which, if not correctly handled by the caller, may lead to an accidental denial of service.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.2/contracts/token/ERC20/utils/SafeERC20.sol#L45-L54

```solidity
function safeApprove(IERC20 token, address spender, uint256 value) internal {
    // safeApprove should only be called when setting an initial allowance,
    // or when resetting it to zero. To increase and decrease it, use
    // 'safeIncreaseAllowance' and 'safeDecreaseAllowance'
    require(
        (value == 0) || (token.allowance(address(this), spender) == 0),
        "SafeERC20: approve from non-zero to non-zero allowance"
    );
    _callOptionalReturn(token, abi.encodeWithSelector(token.approve.selector, spender, value));
}
```

As pointed out in the comment present in the code, the function should only be called if `value` is zero or if the current allowance is zero, i.e. to reset it to zero or to assign it a new value when the current allowance is zero. Any other use case will fail the check, reverting the call.

While this is correctly handled in `applyRewards()`, as the implementation first resets the allowance to zero before assigning it the approval value, this is not the case for both `depositRewards()` and `sellCvx()`. As we can see in the previous snippets of code, lines 255 and 282 execute the call to `safeApprove()` without first resetting the value to zero.

This means that, for both cases, if the current allowance is not zero, the call to `safeApprove()` will fail, reverting the transaction and leading to a denial of service in `depositRewards()` and `sellCvx()`.

## Recommendation

Since CVX is a known token with a correct ERC20 implementation that adheres to the standard, it is not needed to use the SafeERC20 wrapper to handle approvals. Simple remove `safeApprove()` in favor of the standard `approve()`.

```diff
    function depositRewards(
        uint256 _amount,
        uint256 _cvxMinout
    ) public payable onlyManager {
        uint256 cvxAmount = buyCvx(_amount);
        if (cvxAmount < _cvxMinout) revert MinOut();
-       IERC20(CVX_ADDRESS).safeApprove(VLCVX_ADDRESS, cvxAmount);
+       IERC20(CVX_ADDRESS).approve(VLCVX_ADDRESS, cvxAmount);
        ILockedCvx(VLCVX_ADDRESS).lock(address(this), cvxAmount, 0);
        trackedCvxBalance -= cvxAmount;
        emit DepositReward(cvxPerVotium(), _amount, cvxAmount);
    }
```

```diff
    function sellCvx(
        uint256 _cvxAmountIn
    ) internal returns (uint256 ethAmountOut) {
        address CVX_ETH_CRV_POOL_ADDRESS = 0xB576491F1E6e5E62f1d8F26062Ee822B40B0E0d4;
        // cvx -> eth
        uint256 ethBalanceBefore = address(this).balance;
-       IERC20(CVX_ADDRESS).safeApprove(CVX_ETH_CRV_POOL_ADDRESS, _cvxAmountIn);
+       IERC20(CVX_ADDRESS).approve(CVX_ETH_CRV_POOL_ADDRESS, _cvxAmountIn);

        ICrvEthPool(CVX_ETH_CRV_POOL_ADDRESS).exchange_underlying(
            1,
            0,
            _cvxAmountIn,
            0 // this is handled at the afEth level
        );
        ethAmountOut = address(this).balance - ethBalanceBefore;
        trackedCvxBalance -= _cvxAmountIn;
    }
```
