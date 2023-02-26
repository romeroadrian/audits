## Validate operator is not `address(0)` in `setOperator` function of `BorrowController`

Validate the new operator is not `address(0)` so that the operator role remains accessible in case of mistakenly setting a zero/empty value.

## Duplicated code in Oracle `viewPrices` and `getPrices` functions

Consider refactoring duplicate code between these two functions since most of the code is exactly the same except for the difference in the update to the daily low.

## Consider using a `onlyGov` / `onlyChair` modifier in `FED` contract

Most of the protected functions in the `FED` contract have the explicit require to check the sender's role. Consider adding a `onlyGov` and `onlyChair` modifier to protect these functions.

## Ensure `gov` and `chair` addresses in the `FED` contract are initialized and non-empty

Check that `gov` and `chair` are not zero/empty (`address(0)`) in the contract's constructor and in the functions `changeGov` and `changeChair`.

## Unused interface functions in Market contract

Both `balanceOf` functions of `IERC20` and `IDolaBorrowingRights` interfaces aren't used in the scope of this contract and can be safely removed.

## Validate `_replenishmentIncentiveBps` is greater than 0 in `Market` constructor

Validate `_replenishmentIncentiveBps` is greater than 0 during initialization to match the semantics defined in the setter `setReplenismentIncentiveBps`.

## Validate `gov` address is initialized and non-empty in the `Market` contract

Validate `_gov != address(0)` in the contract's constructor and the `setGov` setter. A wrong value here will lead to the contract being ungovernable. 

## Typo in `setReplenismentIncentiveBps` function of `Market` contract

Missing **h** in the function's name.

## Change `_totalSupply` to private in `DolaBorrowingRights` contract

The storage variable `_totalSupply` is defined as public, which will define a `_totalSupply()` getter, which could cause confusion with the `totalSupply()` 
function that properly tracks debt.

## Validate `_operator` address is initialized and non-empty in the `DolaBorrowingRights` contract

Validate `_operator != address(0)` in the contract's constructor. A wrong value here will leave role protected functions inaccessible.
