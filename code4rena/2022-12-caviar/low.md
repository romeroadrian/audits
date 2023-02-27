## Typo in `_validateTokenIds`

https://github.com/code-423n4/2022-12-caviar/blob/main/src/Pair.sol#L465

Line 465 of the Pair contract compares the `merkleRoot` variable against 0 using `bytes23` instead of the correct type `bytes32`. This represents just a typo, as the comparison is still correct.

## Validate `tokenIds` and `proofs` are same length in `_validateTokenIds`

Add a check to ensure both arrays have the same length.

## Unused function in `SafeERC20Namer` library

The `parseStringData` function isn't used and can be safely removed from the library.
