# `EthRouter` contract

- ERC721 transfers to the `EthRouter` contract can be executed using `transferFrom` instead of `safeTransferFrom` in order to save gas, as it is presumed that the `EthRouter` contract can handle ERC721 tokens and the callback isn't needed.  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L162  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L240  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L266  

- The `EthRouter` contract always reapproves the ERC721 contract to the pool for each item in the `sell`, `deposit` and `change` functions. Consider using a lookup mapping to know which contracts have been already approved, if the contract has been already approved it will save a call to `setApprovalForAll` that does an SSTORE and LOG operation (taking OZ and solmate implementations as references).  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L166  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L244  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L270
  
- The whole `Change` struct is copied into memory in each iteration of the loop. Consider referencing it directly from `calldata`.  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L262  
  
# `Factory` contract

- ERC721 transfers to the `Factory` contract can be executed using `transferFrom` instead of `safeTransferFrom` in order to save gas, as it is presumed that the `Factory` contract can handle ERC721 tokens and the callback isn't needed.  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L120  

# `PrivatePool` contract

- Unchecked math can be safely used to save gas due to conditions already checked in the surrounding if statement.
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L268  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L436  

- ERC721 transfers to the `PrivatePool` contract can be executed using `transferFrom` instead of `safeTransferFrom` in order to save gas, as it is presumed that the `PrivatePool` contract can handle ERC721 tokens and the callback isn't needed.  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L331
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L442
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L497
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L648
  
- `setAllParameters` will call all the config setters, and each of these will add a check for the owner. Consider extracting the functionality to avoid repeated access control checks.  
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L602-L615  

- In the `sell` function, the `salePrice` variable is calculated in each iteration of the loop. This isn't needed as the value doesn't depend on anything specific to the loop and remains constant across all iterations. Consider moving this calculation outside the loop to save gas.
  https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335
