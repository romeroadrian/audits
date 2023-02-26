# Unnecessary Code

In `MerkleVerifier.sol` the function `_verifyProof` can be removed (along with the `InvalidProof` error) since the library is used just to compute the root that will be used in the signature check, no verification is needed.

# Missing functions in interfaces

The `IBlurExchange` interface contains the definition for `close()` but it's missing the `open()` function. The getters for the following public variables are missing: `isOpen`, `weth`, `executionDelegate`, `policyManager`, `oracle`, `blockRange`, `cancelledOrFilled`.

The `IExecutionDelegate` interface is missing the getters for `contracts` (consider renaming to something like `isContractApproved`) and `revokedApproval`.

# Unused variables

In `BlurExchange.sol` line 388 the variable `merklePath` isn't used. Consider removing or commenting the variable like this: `(/*bytes32[] memory merklePath */, uint8 _v, bytes32 _r, bytes32 _s) = abi.decode(extraSignature, (bytes32[], uint8, bytes32, bytes32));`
