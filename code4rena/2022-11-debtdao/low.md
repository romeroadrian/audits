# Low 

## Unused imports in Spigot contract

The following imports are unused and can be removed:

- LineLib

## Unused imports in SpigotLib library

The following imports are unused and can be removed:

- ReentrancyGuard

## `_claimRevenue` visibility in `SpigotLib` should be private

This function is only used in the `SpigotLib` library and its visibility is public. Consider changing it to private.

## Duplicated events definitions in `ISpigot` and `SpigotLib`

Both sources define the same events (`AddSpigot`, `RemoveSpigot`, etc.).

## Unused imports in Escrow contract

- Denominations
- IOracle
- ILineOfCredit
- CreditLib
- LineLib

## Change `getCollateralValue()` to view function in Escrow

The `getCollateralValue()` function mutability could probably be changed to view since it shouldn't modify state.


## Define `EscrowLib.MAX_INT` using `type(uint256).max`

The constant `EscrowLib.MAX_INT` is defined by using the literal maximum number. Consider using Solidity's `type(uint256).max` instead for a more declarative and readable version.


## `_getLatestCollateralRatio` visibility in `EscrowLib` should be private

This function is only used in the `EscrowLib` library and its visibility is public. Consider chaging it to private.

## Use staticcall for `previewRedeem(uint256)` call in EscrowLib

The call in line 65 to execute the `previewRedeem(uint256)` function can be changed to use `staticcall` since this function is defined as `view` in the ERC4626. See https://eips.ethereum.org/EIPS/eip-4626#previewredeem

## Use staticcall for `asset()` call in EscrowLib

The call in line 116 to execute the `asset()` function can be changed to use `staticcall` since this function is defined as `view` in the ERC4626. See https://eips.ethereum.org/EIPS/eip-4626#asset

## Use staticcall for `decimals()` call in EscrowLib

The call in line 131 to execute the `decimals()` function can be changed to use `staticcall` since this function is defined as `view` in the ERC20. See https://eips.ethereum.org/EIPS/eip-20#decimals

## Follow "Checks Effects Interactions" pattern in `releaseCollateral` function of `EscrowLib` library

External call in line 166 should be safe to reentrancy, but consider moving this to the bottom of the body to follow the recommended pattern as a precaution measure. See https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html

## Validate address for new line is not empty/zero in `updateLine` function of `EscrowLib` library

As a safe precaution, consider adding a guard `require(_line != address(0))` in the `updateLine` function. A bad value here may lead to the escrow being inaccessible.

## Unused imports in LineOfCredit contract

- Denominations

## `setRates` function in LineOfCredit contract doesn't validate credit isn't closed

This function doesn't check if the given credit has been already closed.

## Move `_sortIntoQ` to `CreditListLib`

This function can probably be moved to the `CreditListLib` library, which handles all the logic around the line of credit queue.

## Modify `CreditLib.create` mutability to view

This function shouldn't mutate state. The call in line 142 can be changed to `staticcall` since ERC20.decimals() is a view function. 

## Unused imports in SpigotedLine contract

- Denominations
- CreditLib
- MutualConsent
