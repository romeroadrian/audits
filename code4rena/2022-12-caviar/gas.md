## LpToken owner can be an immutable variable

The `LpToken` contract uses the `Owned` contract from solmate to handle access protection to the `mint` and `burn` functions. There's no benefit from using a storage variable to represent the owner of this contract, since the intended owner of this token is always the `Pair` contract itself, which doesn't transfer the ownership.

Consider using an immutable variable that is initialized at construction time. This change could save one storage read in every call `mint` and `burn`.

## Use `transferFrom` instead of `safeTransferFrom` in `wrap` function

https://github.com/code-423n4/2022-12-caviar/blob/main/src/Pair.sol#L239

The `Pair` contract uses `safeTransferFrom` when wrapping NFTs in the `wrap` function. There's no point in doing a "safe" transfer here since the receiver is the `Pair` contract itself which already implements the `onERC721Received` hook, as the safe check will always succeed. Consider using `transferFrom` to save gas.
