# AfEth contract

- The `totalSupply()` function is called twice during the execution of `price()`. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L134  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L140  
  
- In `deposit()` line 162, there is no need to increase the variable `totalValue` since its previous value is zero. Consider changing this to an assignment.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L162  
  
- The variable `ratio` is read twice from storage in the function `deposit()`. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L156  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L160  

- In the `deposit()` function, the implementation recalculates the amount deposited by multiplying the collateral's minted tokens by their value. Instead of recalculating everything, simply use the original deposit amount (`msg.value`).  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L162-L165  
  
- The variable `latestWithdrawId` is read 7 times from storage in the function `requestWithdraw()`. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L175-L215  

- In `requestWithdraw()`, the implementation can simply burn the tokens from the holders, instead of first transferring them and then burning later in `withdraw()`.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L187

- In `requestWithdraw()`, the calculation of `votiumWithdrawAmount` can be simplified as `amount * votiumBalance / AfEthBalance`.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L190  
  
- In `requestWithdraw()`, the calculation of `safEthWithdrawAmount` can be simplified as `amount * safEthBalance / AfEthBalance`.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L197  
  
- The variable `vEthAddress` is read twice from storage in the function `requestWithdraw()`. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L221  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L225  
  
- The `withdrawTime` field from the `WithdrawInfo` struct is just used for informative purposes, but can be already derived from the Votium contract by referencing the `vEthWithdrawId` withdrawal. Consider removing this field as it is already stored elsewhere. This also allows the removal of the call to `withdrawTime()` which can be gas intensive.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L207  
  
- Avoid copying the `WithdrawInfo` struct to memory while reading `withdrawIdInfo[_withdrawId]` in the `withdraw()` function. Alias the variable to storage instead.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L249  
  
- The condition `canWithdraw()` in the `withdraw()` function is already checked by the VotiumStrategy contract when executing the withdrawal. Consider removing this duplication to save gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L288  
  
- The variable `vEthAddress` is read twice from storage in the function `depositRewards()`. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L273  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L285  
  
- The visibility of the `setRatio()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L90  
  
- The visibility of the `setFeeAddress()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L98  
  
- The visibility of the `setProtocolFee()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L106  
  
- The visibility of the `depositRewards()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L272  
  
# AbstractStrategy contract

- The `ReentrancyGuardUpgradeable` mixin is not used here. Consider removing it from the inheritance chain. A concrete strategy implementation can add it if needed.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/AbstractStrategy.sol#L10

# VotiumStrategy contract

- The variable `latestWithdrawId` is read 3 times from storage in the function `requestWithdraw()`. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L54-L103  

- In `requestWithdraw()`, the `cvxUnlockObligations` is read from storage in every iteration of the loop. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L80  
  
- In `requestWithdraw()`, the call to `ILockedCvx(VLCVX_ADDRESS).epochs(currentEpoch)` is constant in the loop and can be moved outside of it to execute it only once.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L81-L82  

- In `requestWithdraw()`, the call to `ILockedCvx(VLCVX_ADDRESS).rewardsDuration()` is constant in the loop and can be moved outside of it to execute it only once.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L86   
  
- The function `canWithdraw()` can be marked as `public` instead of `external` to allow `withdraw()` to call it internally instead of executing an external call.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L112  

- The variable `cvxUnlockObligations` is read twice from storage in the function `relock()`. Consider using a local variable as a cache to avoid multiple SLOAD to storage and reduce gas costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L142  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L143  

- The calculation in line 143 can be done using unchecked math since the overflow cannot happen due to the condition in line 142.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L143  

- The visibility of the `deposit()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L39  
  
- The visibility of the `requestWithdraw()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L54  

# VotiumStrategyCore contract

- The visibility of the `setChainlinkCvxEthFeed()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L76  

- The visibility of the `claimRewards()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L192  

- The visibility of the `withdrawStuckTokens()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L215  

- The visibility of the `applyRewards()` can be changed to `external`, as the function isn't used internally in the contract. This will save deployment costs.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L272  

- In `cvxInSystem()`, the implementation can use `ILockedCvx::lockedBalanceOf()` which just returns the total, instead of calling `lockedBalances()` which retrieves more unnecessary data, wasting more gas.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L134  
  
- In both `buyCvx()` and `sellCvx()`, the received output amount from the swap can be directly fetched from the result of the call to `exchange_underlying()`, which returns the `dy` amount. This will avoid querying the balances in the contract to calculate the difference of the received amount, saving gas.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L232-L241  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L255-L264  

- In `claimVlCvxRewards()`, using the ClaimZap will incur in a lot of overhead and wasted gas, since most the claims executed by the ClaimZap contract underneath will be skipped (we can the see the function call has lots of empty arrays/zero values). Instead of using the ClaimZap contract, simply call the `getReward()` of the vlCVX contract which would have the same effect, saving a lot of gas.  
  https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L323  
