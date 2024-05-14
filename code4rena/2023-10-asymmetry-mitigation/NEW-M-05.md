# [ADRIRO-NEW-M-05] Rewarder should not be allowed to apply rewards on CVX tokens

## Summary

The rewarder role should not be allowed to modify the balance of CVX tokens when applying rewards, otherwise the internal CVX balance tracking could get out of sync with major consequences for the protocol.

## Impact

The introduction of internal CVX balance tracking in the VotiumStrategy contract requires utmost care when handling token movements. Accounting should be done properly, as this essentially tracks the balance of CVX tokens that belong to depositors.

One of these sensitive areas is the `applyRewards()` function. This function is used by the rewarder role to swap arbitrary tokens from claimed rewards into ETH, in order to be compounded back into the protocol.

The current implementation of `applyRewards()` is extremely opaque. While the documentation says that the rewarder role will use the 0x protocol to process the swaps, the function executes arbitrary approvals and calls in its implementation, as can be seen in lines 320-323 and 325-327:

```solidity
301:     function applyRewards(
302:         SwapData[] calldata _swapsData,
303:         uint256 _safEthMinout,
304:         uint256 _cvxMinout
305:     ) public onlyRewarder {
306:         uint256 ethBalanceBefore = address(this).balance;
307:         for (uint256 i = 0; i < _swapsData.length; i++) {
308:             // Some tokens do not allow approval if allowance already exists
309:             uint256 allowance = IERC20(_swapsData[i].sellToken).allowance(
310:                 address(this),
311:                 address(_swapsData[i].spender)
312:             );
313:             if (allowance != type(uint256).max) {
314:                 if (allowance > 0) {
315:                     IERC20(_swapsData[i].sellToken).safeApprove(
316:                         address(_swapsData[i].spender),
317:                         0
318:                     );
319:                 }
320:                 IERC20(_swapsData[i].sellToken).safeApprove(
321:                     address(_swapsData[i].spender),
322:                     type(uint256).max
323:                 );
324:             }
325:             (bool success, ) = _swapsData[i].swapTarget.call(
326:                 _swapsData[i].swapCallData
327:             );
328:             if (!success) {
329:                 emit FailedToSell(_swapsData[i].sellToken);
330:             }
331:         }
332:         uint256 ethBalanceAfter = address(this).balance;
333:         uint256 ethReceived = ethBalanceAfter - ethBalanceBefore;
334: 
335:         if (address(manager) != address(0))
336:             IAfEth(manager).depositRewards{value: ethReceived}(
337:                 _safEthMinout,
338:                 _cvxMinout
339:             );
340:         else depositRewards(ethReceived, _cvxMinout);
341:     }
```

While this has its own problems, previously documented in issue M-02 of the original report, the introduction of the internal tracking of CVX brings a new issue vector. If CVX tokens are exchanged here, intentionally or accidentally, this would mean that the real CVX balance could potentially get out of sync with respect to the `trackedCvxBalance` variable.

The `trackedCvxBalance` variable plays a fundamental role in the new implementation of the VotiumStrategy contract, serving as an effective balance of held CVX by the contract. Any deviation of these values will cause issues to deposits, withdrawal and pricing, as this variable is involved in all these processes.

## Proof of Concept

1. The rewarder calls `applyRewards()` by providing some swap data for 0x that swaps CVX tokens.
2. CVX tokens are transferred out of the contract.
3. The real balance of CVX gets out of sync with the value of `trackedCvxBalance`.

## Recommendation

In `applyRewards()`, after rewards have been swapped, verify that the current CVX balance is at least `trackedCvxBalance`. This will ensure CVX tokens are not removed from the contract.

```diff
    function applyRewards(
        SwapData[] calldata _swapsData,
        uint256 _safEthMinout,
        uint256 _cvxMinout
    ) public onlyRewarder {
        uint256 ethBalanceBefore = address(this).balance;
        for (uint256 i = 0; i < _swapsData.length; i++) {
            // Some tokens do not allow approval if allowance already exists
            uint256 allowance = IERC20(_swapsData[i].sellToken).allowance(
                address(this),
                address(_swapsData[i].spender)
            );
            if (allowance != type(uint256).max) {
                if (allowance > 0) {
                    IERC20(_swapsData[i].sellToken).safeApprove(
                        address(_swapsData[i].spender),
                        0
                    );
                }
                IERC20(_swapsData[i].sellToken).safeApprove(
                    address(_swapsData[i].spender),
                    type(uint256).max
                );
            }
            (bool success, ) = _swapsData[i].swapTarget.call(
                _swapsData[i].swapCallData
            );
            if (!success) {
                emit FailedToSell(_swapsData[i].sellToken);
            }
        }
        uint256 ethBalanceAfter = address(this).balance;
        uint256 ethReceived = ethBalanceAfter - ethBalanceBefore;

+       // Ensure CVX tokens are not removed
+       require(IERC20(CVX_ADDRESS).balanceOf(address(this)) >= trackedCvxBalance);

        if (address(manager) != address(0))
            IAfEth(manager).depositRewards{value: ethReceived}(
                _safEthMinout,
                _cvxMinout
            );
        else depositRewards(ethReceived, _cvxMinout);
    }
```



