# Low issues

## Follow "Checks Effects Interactions"

See https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html

- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L79  
  In `SubprotocolRegistry.register` external call to note transfer comes first.

## `delegatecall` in `CidNFT.mint` can be replaced by internal calls 

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L147

Prefer an internal call over the use of delegatecall as the `add` function is present in the same contract.

# Non-critical issues

## Non-library/interface files should use fixed compiler versions, not floating ones	

Pin fixed Solidity versions in pragma specifications.

- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2
- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2
- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L2

## `CidNFT` reimplements `onERC721Received`

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L339

The contract is reimplementing the `onERC721Received` callback as this is already provided by the inherited `ERC721TokenReceiver` mixin from solmate.


