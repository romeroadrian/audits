## Unneeded check in Oracle `getPrice` function

The `twoDayLow > 0` check in line 136 isn't needed since `twoDayLow` will always take at least the current `normalizedPrice` value, which is greater than 0 due to the check in line 117.

## Move check condition in `expansion` function of `FED` contract to save gas

The check in line 93 can be moved up in the stack to early exit the function in case the condition fails in order to save gas. 

```
uint256 newGlobalSupply = globalSupply + amount;
require(newGlobalSupply <= supplyCeiling);
...
globalSupply = newGlobalSupply;
emit Expansion(market, amount);
```

## Use unchecked math in function `contraction` of `FED` contract

Lines 110 and 111 can be placed under an unchecked math group since `amount <= supply` due to check in line 107 and can't possibly underflow these two updates.

## Force replenish checks are done twice in `Market` and `DBR` contracts

The checks around the deficit value (`deficit > 0` and `deficit >= amount`) are done twice and repeated over the `Market` and `DBR` contracts in the functions `forceReplenish` and `onForceReplenish` respectively. 

## Store result of `debts[borrower] += amount` in `borrowInternal` function of `Market` contract

The calculation executed in line 395 is stored in storage and re-read from storage in the next line to check against the credit limit. Store the calculation locally to prevent an unnecessary re-read to storage.

## `name` and `symbol` variables in `DolaBorrowingRights` contract can be changed to immutable to save gas

These two variables are defined at construction time and can't be updated. Consider defining them as immutable to avoid reading from storage to save gas.

## Use unchecked math in `transfer` and `transferFrom` functions of `DolaBorrowingRights` contract

The updates in lines 172 and 196 can be done using unchecked math since `balances[msg.sender] >= amount` and `balances[from] >= amount` due to the checks in lines 171 and 195 respectively.

## Store `lastUpdated[user]` locally to prevent a second read in function `accrueDueTokens` of `DolaBorrowingRights` contract

Value is loaded in line 286 and read again in the next line. Store the first read locally to avoid a second sload.

## Check if debt is zero in `accrueDueTokens` to skip unnecessary calculations and save gas 

If debt is zero in the `accrueDueTokens` function (line 285), then operations in lines 287, 288 and 290 can be skipped to save gas.

## Use unchecked math in `_burn` function of `DolaBorrowingRights` contract

The update in line 374 can be done using unchecked math since `balances[from] >= amount` due to the check in line 373.
