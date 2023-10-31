# Report

## Summary

### Low Issues

Total of **10 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-confusing-semantics-for-bridged-messages-approvals) | Confusing semantics for bridged messages approvals |
| [L-2](#l-2-no-default-threshold-configuration) | No default threshold configuration |
| [L-3](#l-3-validate-number-of-approvers-in-setthresholds) | Validate number of approvers in `setThresholds()` |
| [L-4](#l-4-missing-safe-wrapper-for-erc20-transfer-in-rescuetokens) | Missing safe wrapper for ERC20 transfer in `rescueTokens()` |
| [L-5](#l-5-account-may-get-blacklisted-during-the-bridge-process) | Account may get blacklisted during the bridge process |
| [L-6](#l-6-missing-validation-for-index-parameter-in-overriderange) | Missing validation for index parameter in `overrideRange()` |
| [L-7](#l-7-oracle-assumes-asset-has-18-decimals) | Oracle assumes asset has 18 decimals |
| [L-8](#l-8-wrong-argument-in-transfer-event-of-wrap-function) | Wrong argument in `Transfer` event of `wrap()` function |
| [L-9](#l-9-wrong-argument-in-transfershares-event-of-wrap-function) | Wrong argument in `TransferShares` event of `wrap()` function |
| [L-10](#l-10-contracts-can-be-re-deployed-in-rusdy-factory) | Contracts can be re-deployed in rUSDY factory |

### Non Critical Issues

Total of **2 issues**:

|ID|Issue|
|:--:|:---|
| [NC-1](#nc-1-missing-calls-to-base-initializers-in-rusdy) | Missing calls to base initializers in rUSDY |
| [NC-2](#nc-2-missing-event-to-notify-oracle-has-changed-in-rusdy) | Missing event to notify oracle has changed in rUSDY |

## Low Issues

### <a name="L-1"></a>[L-1] Confusing semantics for bridged messages approvals

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L111

Bridged messages get an "automatic" approval while the message is being executed in the destination bridge contract, which means that all messages effectively get one approval as soon as they are bridged.

This implies that "real" approvals would require setting a threshold with a `numberOfApprovalsNeeded` of at least two. 

This brings a lot of unnecessary confusing semantics to the approvals scheme and could potentially lead to errors in configuration settings that would allow messages to go through automatically when actually they are expected to be approved. 

It is recommended to remove this automatic approval and treat each approval as an explicit approval action of the set of enabled approvers.

### <a name="L-2"></a>[L-2] No default threshold configuration

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L128

The implementation of `_attachThreshold()` loops through each of the configured set of thresholds searching for the element that adjusts to the bridged amount. 

If no configuration is found the function will revert, the message will be lost and it will require manual handling. 

It is recommended setting a default value (a high value number of approvals) to be used when no threshold configuration matches.

### <a name="L-3"></a>[L-3] Validate number of approvers in `setThresholds()`

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L255

The `setThresholds()` function should validate that elements in the `numOfApprovers` array are greater than zero.

### <a name="L-4"></a>[L-4] Missing safe wrapper for ERC20 transfer in `rescueTokens()`

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L322

Use a "safe" wrapper to execute ERC20 transfer for better compatibility.

### <a name="L-5"></a>[L-5] Account may get blacklisted during the bridge process

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L349

Accounts that bridge tokens may get blacklisted in the source chain after the bridging process had been initiated. This could lead to different issues:

- If the account is not blacklisted in the destination chain, the user may still operate their funds.
- If the account is blacklisted, then the function will revert and could be treated as a failure that would require manual intervention.

### <a name="L-6"></a>[L-6] Missing validation for index parameter in `overrideRange()`

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L186

The `overrideRange()` function should validate that `indexToModify` is within bounds, i.e. `require(indexToModify < rangeLength)`.

### <a name="L-7"></a>[L-7] Oracle assumes asset has 18 decimals

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L282

The `roundUpTo8()` implementation assumes the asset has 18 decimals as the rounding is done using `1e10` (since `18 - 8 = 10`). It is recommended to use a constant to properly state this dependency and for better understanding. 

### <a name="L-8"></a>[L-8] Wrong argument in `Transfer` event of `wrap()` function

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L438

The third argument to the `Transfer` event is expecting the amount of rUSDY tokens but it is wrongly called with `getRUSDYByShares(_USDYAmount)`, which lacks the proper scaling to convert the USDY amount to shares.

The correct version should be:

```solidity
emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR));
```

### <a name="L-9"></a>[L-9] Wrong argument in `TransferShares` event of `wrap()` function

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L439

The third argument to the `TransferShares` event is expecting the amount of shares but it is wrongly called with `_USDYAmount`, which is the amount of USDY tokens, not the shares.

The correct version should be:

```solidity
emit TransferShares(address(0), msg.sender, _USDYAmount * BPS_DENOMINATOR);
```

### <a name="L-10"></a>[L-10] Contracts can be re-deployed in rUSDY factory

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L75

The rUSDY contract can be redeployed by the guardian by calling `deployrUSDY()` in the rUSDYFactory contract. This will deploy a new set of contracts and overwrite the storage variables that link to the contract.

It is recommended to add a check to avoid redeployment once contracts have been deployed.

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] Missing calls to base initializers in rUSDY

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L120

The `__rUSDY_init()` function doesn't call the initializers for some of the base contracts:

- Initializable
- ContextUpgradeable
- PausableUpgradeable
- AccessControlEnumerableUpgradeable

### <a name="NC-2"></a>[NC-2] Missing event to notify oracle has changed in rUSDY

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L662

The `setOracle()` function should emit an event to signal off-chain actors that the oracle has been updated.

