## Use typed selector during encoding of `executeCalls` function

Instead of using `abi.encodeWithSignature` with the signature as a string, use the typed selector present in the `ICrossChainExecutor` interface to improve readability, resilience and be less error prone. 

This is present in the relayers for Arbitrum and Optimism.

Example:

```solidity
bytes memory _data = abi.encodeWithSelector(
  ICrossChainExecutor.executeCalls.selector,
  _nonce,
  _sender,
  _calls
);
```

## Unused import in `EthereumToPolygonRelayer.sol`

The imported interface `ICrossChainExecutor` isn't used in the contract.

## Check if `msg.value` is zero in calls to `relayCalls`

Although the interface for the `relayCalls` function requires a `payable` mutability modifier, the relayer implementation in all 3 cases (Optimism, Arbitrum, Polygon) doesn't use `msg.value` and doesn't forward it, making it unnecessary to transfer payments in calls to this function. Consider adding a check to revert if a transaction transfers ETH by mistake.
