# LiquidityMiningPath contract

- In `setConcRewards()`, split the required conjunction into multiple `require` statements to improve gas costs.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67  

- In `setAmbRewards()`, split the required conjunction into multiple `require` statements to improve gas costs.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L76  

# LiquidityMining contract

- In `accrueConcentratedGlobalTimeWeightedLiquidity()`, the `nextWeek` variable calculation can be simplified by just adding a `WEEK` period to `currWeek`, i.e. `uint32 nextWeek = currWeek + WEEK`.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L52  
  
- In `accrueConcentratedGlobalTimeWeightedLiquidity()`, the calculation of `dt` can be safely done using unchecked math due to the check in the ternary expression conditional.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L53-L57  
  
- In `accrueConcentratedPositionTimeWeightedLiquidity()`, the `nextWeek` variable calculation can be simplified by just adding a `WEEK` period to `currWeek`, i.e. `uint32 nextWeek = currWeek + WEEK`.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L97  

- In `accrueConcentratedPositionTimeWeightedLiquidity()`, the calculation of `dt` can be safely done using unchecked math due to the check in the ternary expression conditional.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L98-L102  
  
- In `accrueConcentratedPositionTimeWeightedLiquidity()`, the increment of `tickTrackingIndex` can be done using unchecked math since it is practically impossible to overflow the counter.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L122  

- In `accrueAmbientGlobalTimeWeightedLiquidity()`, the `nextWeek` variable calculation can be simplified by just adding a `WEEK` period to `currWeek`, i.e. `uint32 nextWeek = currWeek + WEEK`.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L209  

- In `accrueAmbientGlobalTimeWeightedLiquidity()`, the calculation of `dt` can be safely done using unchecked math due to the check in the ternary expression conditional.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L210-L214  

- In `accrueAmbientPositionTimeWeightedLiquidity()`, the `nextWeek` variable calculation can be simplified by just adding a `WEEK` period to `currWeek`, i.e. `uint32 nextWeek = currWeek + WEEK`.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L239  

- In `accrueAmbientPositionTimeWeightedLiquidity()`, the calculation of `dt` can be safely done using unchecked math due to the check in the ternary expression conditional.  
  https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L240-L244  
