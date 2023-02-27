## Unify bit access to variables

Packed variables are sometimes accessed using `Decoder.decode` and sometimes accessed by directly applying a mask and a shift.

## `binStep` size should probably be 16 in `LBPairInformation` struct

The `createLBPair` function present in the factory takes the `binStep` argument as a `uint16`, which aligns also with packed size in the fee parameters and other uses.

However, the size for the `binStep` in the `LBPairInformation` is `uint24`.

## `burn` function in `LBPair` should validate array lengths

The `burn` function receives two arrays that are iterated simultaneously and should have the same length. Consider adding a validation to ensure their lengths match.
