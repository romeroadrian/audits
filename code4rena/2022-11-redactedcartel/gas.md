## Unneeded zero address check in `PirexGmx.depositGlp` function

The `token` parameter is checked to guard against the zero address, but then the same parameter is validated using a list of whitelisted addresses, which should also cover the zero address check.

## Use immutable variable for `stakedGlp` token in AutoPxGlp contract

The implementation of `depositFsGlp` fetches the `stakedGlp` token in every call using the PirexGmx contract (`platform`). Since this is a fixed token from the GMX protocol, consider using an immutable variable defined at construction time.

## Use an infinite approve for the `stakedGlp` token in the AutoPxGlp contract

The function `depositFsGlp` will approve the spending for the given amount in every call. Consider adding an infinite approval during contract construction, similar to how it's done with the `gmxBaseReward` token.

## Duplicated call to `totalAssets()` in `compound` function of AutoPxGmx contract

The function fetches the total amount of assets in line 288 (in the if condition) and does the same calculation in line 290 (`asset.balanceOf(address(this)) - assetsBeforeClaim`).

## `pxGmx` token can be an immutable variable in the `PxGmxReward` contract

The `pxGmx` variable is defined at construction and can't be modified by the contract. Consider using an immutable variable to save gas.
