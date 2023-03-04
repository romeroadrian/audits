## Redundant if statement in SmartAccount contract

Line 174 checks if `_handler != address(0)` to call `internalSetFallbackHandler` but that is always the case since line 171 ensures the same condition.

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L174

## Redundant check in `execute` and `executeBatch` functions

Both function have a `onlyOwner` modifier that only allows the owner to execute the function. However, there's a redundant call to `_requireFromEntryPointOrOwner` in the body of both functions.

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L461
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L466

## Usage of safe unchecked math

- In `StakeManager.withdrawTo`, line 118 can be wrapped in an unchecked block due to the require guard in line 117.  
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/core/StakeManager.sol#L118

- In `VerifyingSingletonPaymaster.withdrawTo`, line 58 can be wrapped in an unchecked block due to the require guard in line 57.  
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/paymasters/verifying/singleton/VerifyingSingletonPaymaster.sol#L58