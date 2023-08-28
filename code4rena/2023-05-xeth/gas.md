# xETH.sol contract

- `grantRole` call in `setAMO` function can be changed to use the internal variant `_grantRole` to avoid re-checking for access control, saving gas.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L54
  
- The `curveAMO` storage variable is read 3 times from storage in `setAMO`. Consider using a local variable to save gas.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L44  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L45  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L54

- `mintShares` function visibility can be changed to external as it isn't used internally in the contract.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L63

- `burnShares` function visibility can be changed to external as it isn't used internally in the contract.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L75

- `whenNotPaused` modifier can be removed from the `mintShares` function as this check is already provided by the internal `_beforeTokenTransfer` callback.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L63
  
- `whenNotPaused` modifier can be removed from the `burnShares` function as this check is already provided by the internal `_beforeTokenTransfer` callback.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L75
  
- `whenNotPaused` modifier can be removed from the `pause` function as this check is already included in the internal `_pause` function.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L86
  
- `whenNotPaused` modifier can be removed from the `unpause` function as this check is already included in the internal `_unpause` function.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L93

# wxETH.sol contract

- The SafeERC20 library can be safely removed as the contract will only deal with the xETH token, which is a known implementation that adheres correctly to the ERC20 standard.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L10

- `lastReport` and `dripEnabled` storage variables can be combined to use a single slot and save gas by decreasing the precision of `lastReport` to `uint248`, which should still be plenty enough as this represents a block number.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L60-L62
  
- `lastReport` variable initialization can be safely skipped in the constructor, as it will be eventually initialized by the call to `startDrip()` (dripping is disabled at start).  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L73

- `lockedFunds` storage variable is re-read from storage while emitting the event in the `addLockedFunds()` function. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L156
  
- `drip` is unneeded in the function `startDrip()` as it is presumed that currently dripping is disabled, making `_accrueDrip` have no effect.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L170

- Updating `lastReport` in the `stopDrip()` is not needed as it is already updated during the call to the `drip` modifier.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L191
  
- `lockedFunds` storage variable is read from storage 5 times during the course of the `_accrueDrip()` function. Consider using local variable variants.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L236  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L241  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L248  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L254

- Calculation to update `lockedFunds` can be done using unchecked math since `dripAmount` is guaranteed to be not greater than `lockedFunds`.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L241

# CVXStaker.sol contract

- `rewardsRecipient` is re-read multiple times from storage (once per each iteration of the loop) in the `getRewards()` function. Consider reading it once and using a local variable after.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L190  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L195

- `rewardTokens[i]` is read twice in each iteration of the loop in the `getRewards()` function. Consider reading it once and using a local variable after.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L192
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L195

# AMO2.sol contract

- The SafeERC20 library can be safely removed as the contract will only deal with xETH and stETH tokens, which are known implementations that adheres correctly to the ERC20 standard.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L12

- Infinite token approval can be granted to the curve pool for the xETH and stETH tokens at contract construction time. This will save the approval calls present in each of the functions that deal with adding liquidity to the pool.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L309  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L545  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L546  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L571
  
- `rebalanceUp` function can take struct parameter from `calldata` instead of `memory`, saving a copy operation.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L240
  
- `rebalanceDown` function can take struct parameter from `calldata` instead of `memory`, saving a copy operation.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L287
  
- `bestRebalanceUpQuote` function can take struct parameter from `calldata` instead of `memory`, saving a copy operation.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L336

- `bestRebalanceDownQuote` function can take struct parameter from `calldata` instead of `memory`, saving a copy operation.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L361

- `rebalanceUp` function reads the `cvxStaker` twice from storage. Consider reading it once and using a local variable after. 
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L256  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L261
  
- `rebalanceDown` function reads the `cvxStaker` twice from storage. Consider reading it once and using a local variable after. 
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L314  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L317
  
- `bestRebalanceUpQuote` and `bestRebalanceDownQuote` can return a single result (`min_xETHReceived` and `minLpReceived` respectively) instead of returning a whole new struct in memory.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L337  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L362

- `defender` storage variable is read 3 times from storage in the implementation of `setRebalanceDefender`. Consider using a local variable variant.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L391  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L392  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L395

- `minAmounts` array is not needed in the `removeLiquidityOnlyStETH()` function and can be safely removed.  
  https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L640-L642
