# DestinationBridge contract

- Store value of `chainToApprovedSender[srcChain]` as a local variable instead of reading it multiple times from storage. The same value is read 4 times:  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L96  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L99  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L102  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L106  
  
- Validation in line 96 (`chainToApprovedSender[srcChain] == bytes32(0)`) is not needed as it is already covered by the validation in line 99 (`chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))`).  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L96  
  
- Avoid copying the full array of `Treshold` struct to memory while reading `chainToThresholds[srcChain]` in `_attachThreshold()` function. Alias the variable to `storage` instead.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L135
  
- Improve storage of approvers by using a mapping instead of an array. Currently the array of approvers needs to traverse each time an approver grants approval to check for duplicate entries. By using a mapping, the same operation can be done in O(1).  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L160-L164  
  
- Avoid copying the `TxnThreshold` struct to memory while reading `txnToThresholdSet[txnHash]` in `_checkThresholdMet()` function. Alias the variable to `storage` instead.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L178  
  
- The check in line 270 can use the local `amounts` array instead of fetching the entry from storage.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L270  

- Both branches of the if in line 265 have the same duplicated code that push the entry to the `chainToThresholds[srcChain]`. The code can be refactored to execute the operation outside the if statement and avoid code duplication.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L265  
  
- In `rescueTokens()`, transfer the tokens to `msg.sender` instead of reading the owner from storage.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L324  
  
- Avoid copying the `Transaction` struct to memory while reading `txnHashToTransaction[txnHash]` in `_mintIfThresholdMet()` function. Alias the variable to `storage` instead.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L339  
  

# RWADynamicOracle contract

- In `getPrice()` loop using the indices in revese order (`i = length - 1; i >=0l; --i`) instead of calculating the `(length - 1) - i` expression in every iteration.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L75

- In `getPrice()`, the `range.end` variable is read from storage two times. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L80  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L81  
  
- In `setRange()`, the length of the `ranges` array is fetched from storage two times. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L155  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L165  

- Avoid copying the `Range` struct to memory while reading `currentRange` in `derivePrice()` function. Alias the variable to `storage` instead.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L263  
  
- Reduce size of variables in `Range` struct to enable efficient storage packing.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L295-L300

- The `_mul` doesn't need to check for overflow since checked arithmetic is already provided in Solidity 0.8  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L404-L406  


# rUSDY contract

- Consider not updating allowance if it the current allowance is set to an "infinite" (`type(uint256).max`) value.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L310  
  
- Calculation in line 310 can be unchecked since the condition has been already checked in line 307.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L310  

- Calculation in line 362 can be unchecked since the condition has been already checked in line 359.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L362

- The `wrap()` function doesn't require the `whenNotPaused` since the implementation of `_mintShares` already checks this.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L434  
  
- The `unwrap()` function doesn't require the `whenNotPaused` since the implementation of `_burnShares` already checks this.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L449  
  
- Calculation in line 530 can be unchecked since the condition has been already checked in line 526.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L530  

- Calculation in line 531 can be unchecked since overflow is already covered by the `totalShares` variable.  
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L531  

- The `totalShares` variable is re-fetched from storage in `_mintShares`. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L555  
  
- Calculation in line 590 can be unchecked since the condition has been already checked in line 584.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L590  

- The `totalShares` variable is re-fetched from storage in `_burnShares`. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L601  
  

# rUSDYFactory contract

- The `rUSDYImplementation` is read multiple times from storage in `deployrUSDY()`. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L82  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L85  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L104  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L111  

- The `rUSDYProxyAdmin` is read multiple times from storage in `deployrUSDY()`. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L83  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L86  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L99  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L100  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L103  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L110  

- The `rUSDYProxy` is read multiple times from storage in `deployrUSDY()`. Consider using a local variable cache.  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L84  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L89  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L102  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L109  

- Remove unnecessary `assert()` in `deployrUSDY()`  
  https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L100  
