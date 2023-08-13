# LiquidityPool contract

- Unused storage variable `addressResolver` can be removed  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L78
  
- `nextQueuedDepositId` is read twice from storage in `queueDeposit`  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L206-L207

- No need to store the field `id` in the `QueuedDeposit` struct as this is the key of the mapping.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L207
  
- `queuedDepositHead` is read and written to storage in every iteration of the loop in the `processDeposits` function. Consider storing a local variable and updating it at the end of the process.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L219
  
- `minDepositDelay` is read from storage in every iteration of the loop in the `processDeposits` function. Consider storing it in a local variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L226
  
- `current.requestedTime` is read from storage 3 times in every iteration of the loop in the `processDeposits` function. Consider loading it just once in a local variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L226  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L237  

- `current.depositedAmount` is read from storage 4 times in every iteration of the loop in the `processDeposits` function. Consider loading it just once in a local variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L230  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L233  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L234  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L237  

- `current.user` is read from storage 2 times in every iteration of the loop in the `processDeposits` function. Consider loading it just once in a local variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L235  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L237  

- `nextQueuedWithdrawalId` is read twice from storage in `queueWithdraw`  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L271-L272  

- No need to store the field `id` in the `QueuedWithdraw` struct as this is the key of the mapping.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L272  

- `queuedWithdrawalHead` is read and written to storage in every iteration of the loop in the `processWithdraws` function. Consider storing a local variable and updating it at the end of the process.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L289  
  
- `current.requestedTime` is read from storage 3 times in every iteration of the loop in the `processWithdraws` function. Consider loading it just once in a local variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L291  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L326  

- `current.withdrawnTokens` is read from storage 3 times in every iteration of the loop in the `processWithdraws` function. Consider loading it just once in a local variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L301  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L321  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L326  

- `current.user` is read from storage 2 times in every iteration of the loop in the `processWithdraws` function. Consider loading it just once in a local variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L311  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L314  

# ShortCollateral contract

- `Collateral` struct is copied entirely to memory in `liquidate` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L131  

- `userCollateral.collateral` is read 3 times from storage in `liquidate` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L130  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L141  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L143  
  
- `userCollateral.amount` is read 2 times from storage in `liquidate` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L138  

- `Collateral` struct is copied entirely to memory in `getMinCollateral` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L161  

- `Collateral` struct is copied entirely to memory in `getLiquidationBonus` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L184  
  
- `ShortPosition` struct is copied entirely to memory in `canLiquidate` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L196  
  
- `Collateral` struct is copied entirely to memory in `canLiquidate` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L200  

- `ShortPosition` struct is copied entirely to memory in `maxLiquidatableDebt` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L219  

- `Collateral` struct is copied entirely to memory in `maxLiquidatableDebt` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortCollateral.sol#L223  

# ShortToken contract

- No need to store the field `positionId` in the `ShortPosition` struct as this is the key of the mapping.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortToken.sol#L18

- Simplify expression for `totalShorts` calculation in `adjustPosition` function  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/ShortToken.sol#L73-L77  
  ```solidity
  totalShorts = totalShorts - position.shortAmount + shortAmount
  ```

# SynthetixAdapter contract

- `synthetix` and `exchangeRates` storage variables can be declared as `immutable` as these are not mutable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/SynthetixAdapter.sol#L10-L11

- The function present in the contract indicates this is a facade for some external calls to the Synthetix contracts. Consider using a library with `internal` functions to inline code instead of executing an external call.


# SystemManager contract

- `perpMarketAddress` and `perpMarket` always contain the same value. Consider removing one of these.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/SystemManager.sol#L85-L86

- `addressResolver` address isn't used in the contract or by external contracts (LiquidityPool has a reference but it isn't used either). Consider removing this storage variable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/SystemManager.sol#L30
  
- `addressResolver` and `futuresMarketManager` storage variables can be declared as `immutable` as these are not mutable.  
  https://github.com/code-423n4/2023-03-polynomial/blob/main/src/SystemManager.sol#L30-L31
