## Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions	

Add some storage gap to base contracts to allow adding storage variables in future upgrades.

Occurrences:

- BaseSmartAccount (used by SmartAccount)

Also the following mixins are used in the SmartAccount contract

- Singleton
- ModuleManager
- SignatureDecoder
- SecuredTokenTransfer
- FallbackManager

## Move setters to the "setters" section in SmartAccount contract

Setters defined between lines 94-103 should be placed after line 107.

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L93-L103

## Use `block.chainid` instead of assembly

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L140-L147

In Solidity 0.8 `block.chainid` can be used to retrieve the chain id value.

## Duplicated getter `getNonce` in SmartAccount contract

The getter `getNonce` is already defined by the `nonce(uint256 _batchId)` function.

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L155-L159

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L97-L99

## Wrong require description in SmartAccount contract

Line 171 reads "Invalid Entrypoint", but the check belongs to the "handler" variable.

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L171


## Dedupe code between `handlePayment` and `handlePaymentRevert` functions

Most of the code is duplicated in both functions. Consider extracting common functionality to an internal function.

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L247-L269
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L271-L295

## No bounds check in `checkSignatures` function

The `signatures` variable length isn't checked before the call to `signatureSplit`. Since this variable should contain at least one element, consider adding a check `require(signatures.length >= 65)`.

## Function `_requireFromEntryPointOrOwner` can be safely removed from the SmartAccount contract

The `_requireFromEntryPointOrOwner` is only used in the `execute` and `executeBatch` function, but as its usage is redundant (see gas report) the function can be entirely removed.

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L494-L496

## Use `abi.encode` to set arguments for creation payload in SmartAccountFactory

Prefer using `abi.encode` to safely encode arguments that are using to build the creation payload. Example:

```solidity
bytes memory deploymentData = abi.encodePacked(type(Proxy).creationCode, abi.encode(_defaultImpl));
```

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccountFactory.sol#L35

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccountFactory.sol#L54

## Function `getUserOpGasPrice` in `EntryPoint` contract is duplicated

The `UserOperation` library defines exactly the same function (`gasPrice`).

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/core/EntryPoint.sol#L484

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/interfaces/UserOperation.sol#L45

## Use typed selector instead of bytes4 constant

In `DefaultCallbackHandler`, functions `onERC1155Received` / `onERC1155BatchReceived` / `onERC721Received`, use the typed selector instead of the constant. For example:

```
function onERC721Received(
    address,
    address,
    uint256,
    bytes calldata
) external pure override returns (bytes4) {
    return ERC721TokenReceiver.onERC721Received.selector;
}
```

## Missing event for important parameter change	

Important parameter or configuration changes should trigger an event to allow being tracked off-chain:

Occurrences:

- https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/paymasters/verifying/singleton/VerifyingSingletonPaymaster.sol#L65
- https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/paymasters/BasePaymaster.sol#L24