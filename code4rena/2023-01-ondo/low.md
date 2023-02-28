## Unneeded initialization in `CashManager` constructor

https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L168

The `currentEpoch` variable is initialized with the same variable.

## `setPendingMintBalance` doesn't check for mint limit

https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L336-L350

The function `setPendingMintBalance` of the `CashManager` contract allows the manager to set the mint balance for a user, but fails to check and update the mint limit (`mintLimit` variable).

## `overrideExchangeRate` fails to validate the new exchange rate

https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L366-L385

The `overrideExchangeRate` function of the `CashManager` contract should validate that the new rate is not 0 (`correctExchangeRate != 0`).

## Validate new config parameter is not `address(0)`

Occurrences:

- https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L452
- https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L465

## The `sweepToken` function is missing in the `CErc20DelegatorKYC` contract

The `sweepToken` function  which is implemented in the original Compound contract (https://github.com/compound-finance/compound-protocol/blob/3affca87636eecd901eb43f81a4813186393905d/contracts/CErc20Delegator.sol#L330-L336) is missing in the `CErc20DelegatorKYC` contract.
