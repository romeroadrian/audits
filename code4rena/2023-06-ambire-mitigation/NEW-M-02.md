# [adriro-NEW-M-02]: Wallet design prevents EIP-165 extensibility

The current wallet fallback design prevents the extensibility of the EIP-165 functionality.

## Impact

Ambire wallet extensibility is provided by a fallback mechanism. If a fallback handler is defined in slot `FALLBACK_HANDLER_SLOT`, any non-matching function will be delegated to this address.

This is a simple and elegant pattern that can provide extensibility by forwarding calls to a configured external contract. If the function selector isn't present in the AmbireAccount contract, the call is forwarded to the fallback contract using `delegatecall`.

There is a drawback to this pattern though. In this design, functions that are both defined in the wallet contract and the fallback handler will be caught by the main contract (AmbireAccount) instead of being delegated to the fallback, since function dispatch happens first in the main contract. The fallback handler won't be able to extend or override functions defined in the wallet contract.

This means that EIP-165 functionality cannot be extended. The implementation of EIP-165 is done by defining a `supportsInterface()` function that can be queried for interface support. This function is currently defined in the AmbireAccount contract to signal support for `ERC721TokenReceiver` and `ERC1155TokenReceiver`. The fallback handler won't be able to extend this functionality as the call can't reach the fallback, it will be caught by the wallet contract and will only answer for the already defined cases.

Any unimplemented or future standard that requires publishing interface support via EIP-165 won't be able to be integrated in the Ambire wallet.

## Recommendation

Similar to how [`ERC165Storage`](https://docs.openzeppelin.com/contracts/4.x/api/utils#ERC165Storage) works in the OpenZeppelin library, it is possible to define a mapping that will hold supported interfaces. The fallback (or any transaction to the wallet) can register interfaces by using a function only callable from the same contract.

```solidity
mapping(bytes4 => bool) private _supportedInterfaces;

function setInterface(bytes4 interfaceId, bool value) external {
  require(msg.sender == address(this), 'ONLY_IDENTITY_CAN_CALL');
  _supportedInterfaces[interfaceId] = value;
}

function supportsInterface(bytes4 interfaceID) external pure returns (bool) {
  return
  interfaceID == 0x01ffc9a7 || // ERC-165 support (i.e. `bytes4(keccak256('supportsInterface(bytes4)'))`).
  interfaceID == 0x150b7a02 || // ERC721TokenReceiver
  interfaceID == 0x4e2312e0 || // ERC-1155 `ERC1155TokenReceiver` support (i.e. `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)")) ^ bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))`).
  _supportedInterfaces[interfaceId];
}

```
