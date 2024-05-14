# Gas Report

## Summary

### Gas Optimizations

Total of **16 findings**:

|&nbsp;&nbsp;&nbsp;ID&nbsp;&nbsp;&nbsp;|Finding|
|:--:|:---|
| [G-1](#g-1-remove-isfree-logic-as-request-l2-transactions-cant-be-free) | Remove `_isFree` logic as request L2 transactions can't be free |
| [G-2](#g-2-safe-unchecked-operations-in-priorityqueue) | Safe `unchecked` operations in `PriorityQueue` |
| [G-3](#g-3-unnecessary-delete-of-map) | Unnecessary `delete` of map |
| [G-4](#g-4-replace-memory-with-storage-for-one-time-use-returned-struct-in-priorityqueuefront) | Replace memory with storage for one-time use returned struct in `PriorityQueue::front()` |
| [G-5](#g-5-unnecessary-copy-of-storage-struct-to-memory-in-diamond) | Unnecessary copy of storage struct to memory in `Diamond` |
| [G-6](#g-6-deposit-struct-in-iallowlist-can-be-tightly-packed-to-fit-in-a-single-storage-slot) | `Deposit` struct in IAllowList can be tightly packed to fit in a single storage slot |
| [G-7](#g-7-diamondproxy-delegation-doesnt-need-to-preserve-solidity-memory-layout) | DiamondProxy delegation doesn't need to preserve Solidity memory layout |
| [G-8](#g-8-totalbatchescommitted-is-read-twice-from-storage-in-executorcommitbatches) | `totalBatchesCommitted` is read twice from storage in `Executor::commitBatches()` |
| [G-9](#g-9-totalbatchesverified-is-read-twice-from-storage-in-executorprovebatches) | `totalBatchesVerified` is read twice from storage in `Executor::proveBatches()` |
| [G-10](#g-10-totalbatchescommitted-totalbatchesverified-and-totalbatchesexecuted-are-re-read-from-storage-in-executorrevertbatches) | `totalBatchesCommitted`, `totalBatchesVerified` and `totalBatchesExecuted` are re-read from storage in `Executor::revertBatches()` |
| [G-11](#g-11-avoid-copying-the-whole-facettoselectors-struct-to-memory-in-gettersfacets) | Avoid copying the whole `FacetToSelectors` struct to memory in `Getters::facets()` |
| [G-12](#g-12-the-l2gasperpubdatabytelimit-argument-in-mailboxrequestl2transaction-is-fixed-to-a-constant) | The `_l2GasPerPubdataByteLimit` argument in `Mailbox::requestL2Transaction()` is fixed to a constant |
| [G-13](#g-13-user-deposited-amount-in-mailbox-verifydepositlimit-is-read-twice-from-storage) | User deposited amount in `Mailbox::_verifyDepositLimit()` is read twice from storage |
| [G-14](#g-14-avoid-variable-initialization-for-default-values-in-transactionvalidatorgetminimalprioritytransactiongaslimit) | Avoid variable initialization for default values in `TransactionValidator::getMinimalPriorityTransactionGasLimit()` |
| [G-15](#g-15-mathmax-is-not-needed-while-initializing-batchoverheadfortransaction-in-transactionvalidatorgetoverheadfortransaction) | `Math::max()` is not needed while initializing `batchOverheadForTransaction` in `TransactionValidator::getOverheadForTransaction()` |
| [G-16](#g-16-transactionvalidatorgetoverheadfortransaction-can-be-greatly-simplified) | `TransactionValidator::getOverheadForTransaction()` can be greatly simplified |

## Gas Optimizations

### <a name="G-1"></a>[G-1] Remove `_isFree` logic as request L2 transactions can't be free

The `Mailbox` has an [_isFree](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L291) attribute on its `_requestL2Transaction()` function.

The function is only used by `requestL2Transaction()` and [always sets the `_isFree` attribute as false](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L270).

This attribute can be removed to save gas, as well as saving extra gas [on other unnecessary operations](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L304).

### <a name="G-2"></a>[G-2] Safe `unchecked` operations in `PriorityQueue`

Some computations are safe to be included under an `unchecked` block to save gas:

`_queue.tail - _queue.head` can safely use it, as `tail >= head`:

```diff
  function getSize(Queue storage _queue) internal view returns (uint256) {
-      return uint256(_queue.tail - _queue.head);
+      unchecked {
+          return uint256(_queue.tail - _queue.head);
+      }
  }
```

[PriorityQueue.sol#L46](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/PriorityQueue.sol#L46)

`_queue.tail = tail + 1` can use it as `tail` is `uint256`:

```diff
  function pushBack(Queue storage _queue, PriorityOperation memory _operation) internal {
      // Save value into the stack to avoid double reading from the storage
      uint256 tail = _queue.tail;

      _queue.data[tail] = _operation;
-     _queue.tail = tail + 1;
+      unchecked {
+         _queue.tail = tail + 1;
+      }
  }
```

[PriorityQueue.sol#L60](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/PriorityQueue.sol#L60)

`_queue.head = head + 1` can use it as `head` is `uint256`:

```diff
    function popFront(Queue storage _queue) internal returns (PriorityOperation memory priorityOperation) {
        require(!_queue.isEmpty(), "s"); // priority queue is empty

        // Save value into the stack to avoid double reading from the storage
        uint256 head = _queue.head;

        priorityOperation = _queue.data[head];
        delete _queue.data[head];
-       _queue.head = head + 1;
+        unchecked {
+           _queue.head = head + 1;
+        }
    }
```

[PriorityQueue.sol#L80](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/PriorityQueue.sol#L80)

### <a name="G-3"></a>[G-3] Unnecessary `delete` of map

The `PriorityQueue` deletes the `queue.data[head]` from the map, whenever `popFront()` is called.

This `delete` operation is unnecessary, and can be removed to save gas as the information for past queue heads is not publicly accessed nor used by other internal methods.

```diff
    function popFront(Queue storage _queue) internal returns (PriorityOperation memory priorityOperation) {
        require(!_queue.isEmpty(), "s"); // priority queue is empty

        // Save value into the stack to avoid double reading from the storage
        uint256 head = _queue.head;

        priorityOperation = _queue.data[head];
-       delete _queue.data[head];
        _queue.head = head + 1;
    }
```

[PriorityQueue.sol#L79](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/PriorityQueue.sol#L79)

### <a name="G-4"></a>[G-4] Replace memory with storage for one-time use returned struct in `PriorityQueue::front()`

`PriorityQueue::front()` is only used internally by [Getters::priorityQueueFrontOperation()](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Getters.sol#L73-L75). It can be safely changed from `memory` -> `storage` to save gas.

```diff
-    function front(Queue storage _queue) internal view returns (PriorityOperation memory) {
+    function front(Queue storage _queue) internal view returns (PriorityOperation storage) {
```

[PriorityQueue.sol#L64](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/PriorityQueue.sol#L64)

### <a name="G-5"></a>[G-5] Unnecessary copy of storage struct to memory in `Diamond`

The `Diamond` contract has several instances where a complete struct is being copied to memory to end up using only one of its attributes.

This is done inside `for` loops, so it ends up spending even more gas.

Consider accessing the specific attribute from the storage directly to save gas:

```diff
    for (uint256 i = 0; i < selectorsLength; i = i.uncheckedInc()) {
        bytes4 selector = _selectors[i];
-       SelectorToFacet memory oldFacet = ds.selectorToFacet[selector];
-       require(oldFacet.facetAddress == address(0), "J"); // facet for this selector already exists
+       require(ds.selectorToFacet[selector].facetAddress == address(0), "J"); // facet for this selector already exists

        _addOneFunction(_facet, selector, _isFacetFreezable);
    }
```

[Diamond.sol#L138-L144](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/Diamond.sol#L138-L144)

```diff
    for (uint256 i = 0; i < selectorsLength; i = i.uncheckedInc()) {
        bytes4 selector = _selectors[i];
-       SelectorToFacet memory oldFacet = ds.selectorToFacet[selector];
-       require(oldFacet.facetAddress != address(0), "L"); // it is impossible to replace the facet with zero address
+       require(ds.selectorToFacet[selector].facetAddress != address(0), "L"); // it is impossible to replace the facet with zero address

        _removeOneFunction(oldFacet.facetAddress, selector);
        // Add facet to the list of facets if the facet address is a new one
        _saveFacetIfNew(_facet);
        _addOneFunction(_facet, selector, _isFacetFreezable);
    }
```

[Diamond.sol#L159-L168](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/Diamond.sol#L159-L168)

```diff
    for (uint256 i = 0; i < selectorsLength; i = i.uncheckedInc()) {
        bytes4 selector = _selectors[i];
-       SelectorToFacet memory oldFacet = ds.selectorToFacet[selector];
-       require(oldFacet.facetAddress != address(0), "a2"); // Can't delete a non-existent facet
+       require(ds.selectorToFacet[selector].facetAddress != address(0), "a2"); // Can't delete a non-existent facet

        _removeOneFunction(oldFacet.facetAddress, selector);
    }
```

[Diamond.sol#L179-L185](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/Diamond.sol#L179-L185)

### <a name="G-6"></a>[G-6] `Deposit` struct in IAllowList can be tightly packed to fit in a single storage slot

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/common/interfaces/IAllowList.sol#L29-L32

The struct type used to keep track of deposit limits can be tightly packed into a single storage slot by reducing the size of the `depositCap` to `uint248`. This would still allow sufficient room to represent any reasonable limit, while saving on gas costs when accessing storage.

```diff
    struct Deposit {
        bool depositLimitation;
-       uint256 depositCap;
+       uint248 depositCap;
    }
```

### <a name="G-7"></a>[G-7] DiamondProxy delegation doesn't need to preserve Solidity memory layout

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/DiamondProxy.sol#L35

The `fallback()` implementation in the DiamondProxy contract loads the free memory pointer from position `0x40` in order to maintain the memory layout defined by Solidity. 

This is not needed, and the implementation can simply copy the response from `delegatecall` and overwrite memory at position `0x00`, since execution will derive in either a low level `return` or `revert`, without any further Solidity code.

### <a name="G-8"></a>[G-8] `totalBatchesCommitted` is read twice from storage in `Executor::commitBatches()`

The storage variable `totalBatchesCommitted` is loaded from storage twice in lines [184](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L184) and [199](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L199).

Consider storing the first storage access in a local variable to avoid multiple SLOADs and save on gas costs.

### <a name="G-9"></a>[G-9] `totalBatchesVerified` is read twice from storage in `Executor::proveBatches()`

The storage variable `totalBatchesCommitted` is loaded from storage twice in lines [317](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L317) and [371](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L371).

Consider storing the first storage access in a local variable to avoid multiple SLOADs and save on gas costs.

### <a name="G-10"></a>[G-10] `totalBatchesCommitted`, `totalBatchesVerified` and `totalBatchesExecuted` are re-read from storage in `Executor::revertBatches()`

The storage variables `totalBatchesCommitted`, `totalBatchesVerified` and `totalBatchesExecuted` are read again from storage to emit the `BlocksRevert` event in line [413](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L413).

Consider using the updated local variables to avoid multiple SLOADs and save on gas costs.

### <a name="G-11"></a>[G-11] Avoid copying the whole `FacetToSelectors` struct to memory in `Getters::facets()`

The whole structure `FacetToSelectors` is copied into memory while accessing the `facetToSelectors[facetAddr]` mapping in line [184](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Getters.sol#L184).

Consider aliasing the result to storage and/or just copy the needed fields into memory to avoid unneeded operations and save on gas costs.

### <a name="G-12"></a>[G-12] The `_l2GasPerPubdataByteLimit` argument in `Mailbox::requestL2Transaction()` is fixed to a constant

The function `requestL2Transaction()` contains a `_l2GasPerPubdataByteLimit` argument that is always enforced to be a constant value `REQUIRED_L2_GAS_PRICE_PER_PUBDATA`. 

Consider just using the constant value, in order to remove the check in line [257](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L257) and save on gas costs.

### <a name="G-13"></a>[G-13] User deposited amount in `Mailbox::_verifyDepositLimit()` is read twice from storage

The value `s.totalDepositedAmountPerUser[_depositor]` is accessed twice in the same function, first in line [279](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L279) to do the check and then in line [290](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L279) to implement the update using the `+=` operator.

Consider storing `s.totalDepositedAmountPerUser[_depositor]` in a local variable to use it in both places.

```solidity
uint256 newUserDepositedAmount = s.totalDepositedAmountPerUser[_depositor] + amount;
require(newUserDepositedAmount <= limitData.depositCap, "d2");
s.totalDepositedAmountPerUser[_depositor] = newUserDepositedAmount;
```

### <a name="G-14"></a>[G-14] Avoid variable initialization for default values in `TransactionValidator::getMinimalPriorityTransactionGasLimit()`

The local variable `costForPubdata` is initialized with zero in line [92](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/TransactionValidator.sol#L92). 

The initialization is not needed, since the assigned value is the default value.

### <a name="G-15"></a>[G-15] `Math::max()` is not needed while initializing `batchOverheadForTransaction` in `TransactionValidator::getOverheadForTransaction()`

The `batchOverheadForTransaction` variable is first initialized with the resulting value of `Math.max(batchOverheadForTransaction, txSlotOverhead)` in line [142](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/TransactionValidator.sol#L142), but since `batchOverheadForTransaction` value is zero at this point, this is effectively the same as doing `Math.max(0, txSlotOverhead)` which is the same as `txSlotOverhead`.

Change the assignment to just `txSlotOverhead` to avoid the `Math::max()` computation and save on gas costs.

```diff
-   batchOverheadForTransaction = Math.max(batchOverheadForTransaction, txSlotOverhead);
+   batchOverheadForTransaction = txSlotOverhead;
```

### <a name="G-16"></a>[G-16] `TransactionValidator::getOverheadForTransaction()` can be greatly simplified

The implementation of `getOverheadForTransaction()` first calculates the transaction's slot overhead and the length overhead, and then takes the maximum between these two:

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/TransactionValidator.sol#L138-L146

```solidity
138:         uint256 batchOverheadGas = BATCH_OVERHEAD_L2_GAS + BATCH_OVERHEAD_PUBDATA * _gasPricePerPubdata;
139: 
140:         // The overhead from taking up the transaction's slot
141:         uint256 txSlotOverhead = Math.ceilDiv(batchOverheadGas, MAX_TRANSACTIONS_IN_BATCH);
142:         batchOverheadForTransaction = Math.max(batchOverheadForTransaction, txSlotOverhead);
143: 
144:         // The overhead for occupying the bootloader memory can be derived from encoded_len
145:         uint256 overheadForLength = Math.ceilDiv(_encodingLength * batchOverheadGas, BOOTLOADER_TX_ENCODING_SPACE);
146:         batchOverheadForTransaction = Math.max(batchOverheadForTransaction, overheadForLength);
```
The resulting value for `batchOverheadGas` is `batchOverheadGas = BATCH_OVERHEAD_L2_GAS + BATCH_OVERHEAD_PUBDATA * _gasPricePerPubdata = 1200000 + 58823 * 800 = 48258400`.

Then `txSlotOverhead` is `txSlotOverhead = Math.ceilDiv(batchOverheadGas, MAX_TRANSACTIONS_IN_BATCH) = roundUp(48258400 / 1024) = 47128`.

At this point `batchOverheadForTransaction` is  `batchOverheadForTransaction = Math.max(batchOverheadForTransaction, txSlotOverhead) = Math.max(0, 47128) = 47128`.

Then comes `overheadForLength`, which is `overheadForLength = Math.ceilDiv(_encodingLength * batchOverheadGas, BOOTLOADER_TX_ENCODING_SPACE) = roundUp(_encodingLength * 48258400 / 273132)`. The variable `_encodingLength` is the length of the ABI encoded `L2CanonicalTransaction` structure. The minimum length for this structure, taking into account all its fields with empty dynamic arrays for all cases, is 800 bytes. Which means that `overheadForLength >= roundUp(800 * 48258400 / 273132) = 141349`.

The implementation now updates `batchOverheadForTransaction` by doing `batchOverheadForTransaction = Math.max(batchOverheadForTransaction, overheadForLength) = Math.max(47128, overheadForLength)`. But since we know `overheadForLength >= 141349`, this last calculation will always default with whatever value `overheadForLength` has, since `47128 < 141349`.

Consider skipping all these calculations and just using `overheadForLength` as the initial value for `batchOverheadForTransaction`.

```diff
    uint256 batchOverheadGas = BATCH_OVERHEAD_L2_GAS + BATCH_OVERHEAD_PUBDATA * _gasPricePerPubdata;

-   // The overhead from taking up the transaction's slot
-   uint256 txSlotOverhead = Math.ceilDiv(batchOverheadGas, MAX_TRANSACTIONS_IN_BATCH);
-   batchOverheadForTransaction = Math.max(batchOverheadForTransaction, txSlotOverhead);

    // The overhead for occupying the bootloader memory can be derived from encoded_len
-   uint256 overheadForLength = Math.ceilDiv(_encodingLength * batchOverheadGas, BOOTLOADER_TX_ENCODING_SPACE);
-   batchOverheadForTransaction = Math.max(batchOverheadForTransaction, overheadForLength);
    batchOverheadForTransaction = Math.ceilDiv(_encodingLength * batchOverheadGas, BOOTLOADER_TX_ENCODING_SPACE);
```

