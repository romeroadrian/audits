# QA Report

## Summary

### Low Issues

Total of **18 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-extra-notifyexecutionresult-gas-cost-is-charged-to-users-on-l1-l2-transactions-processing) | Extra `notifyExecutionResult` gas cost is charged to users on L1->L2 transactions processing |
| [L-2](#l-2-if-dummy-verifier-true-block-in-executor-introduces-risk-of-halting-batch-proving-in-production) | `if DUMMY_VERIFIER == true` block in `Executor` introduces risk of halting batch proving in production |
| [L-3](#l-3-refund-recipient-addresses-not-aliased-if-transactions-are-requested-on-constructor) | Refund recipient addresses not aliased if transactions are requested on constructor |
| [L-4](#l-4-double-alias-conversion-in-l1-bridge-deposits) | Double alias conversion in L1 bridge deposits |
| [L-5](#l-5-wrong-validation-on-max-number-of-numberofl2tol1logs) | Wrong validation on max number of numberOfL2ToL1Logs |
| [L-6](#l-6-isfacetfreezable-returns-false-for-non-existing-facets-instead-of-reverting) | `isFacetFreezable()` returns false for non-existing facets instead of reverting |
| [L-7](#l-7-sending-many-logs-via-a-single-l1-l2-transactions-can-halt-the-publishing-of-pubdata) | Sending many logs via a single L1->L2 transactions can halt the publishing of pubdata |
| [L-8](#l-8-generating-a-large-pubdata-via-a-single-l1-l2-transaction-can-halt-the-publishing-to-l1-operation) | Generating a large pubdata via a single L1->L2 transaction can halt the publishing to L1 operation |
| [L-9](#l-9-upgrade-openzeppelin-dependencies) | Upgrade OpenZeppelin dependencies |
| [L-10](#l-10-missing-check-for-address0-for-the-allowlist-in-diamondinitinitialize) | Missing check for address(0) for the `allowList` in `DiamondInit::initialize()` |
| [L-11](#l-11-missing-getters-in-getters-sol) | Missing getters in `Getters.sol` |
| [L-12](#l-12-prevent-weth-deposits-in-l1erc20bridge) | Prevent WETH deposits in L1ERC20Bridge |
| [L-13](#l-13-facet-state-is-not-entirely-cleared-when-last-selector-is-removed) | Facet state is not entirely cleared when last selector is removed |
| [L-14](#l-14-shadow-operations-will-ultimately-need-to-be-revealed-on-chain) | Shadow operations will ultimately need to be revealed on-chain |
| [L-15](#l-15-potential-gas-bomb-in-governance-execution) | Potential gas bomb in Governance execution |
| [L-16](#l-16-verifier-params-can-be-accidentally-skipped-during-upgrades) | Verifier params can be accidentally skipped during upgrades |
| [L-17](#l-17-governance-contract-does-not-implement-erc721-or-erc1155-receivers) | Governance contract does not implement ERC721 or ERC1155 receivers |
| [L-18](#l-18-potential-missing-validation-in-systemcontextpublishtimestampdatatol1) | Potential missing validation in `SystemContext::publishTimestampDataToL1()` |

### Non Critical Issues

Total of **5 issues**:

|ID|Issue|
|:--:|:---|
| [NC-1](#nc-1-compressor-uses-hardcoded-values-instead-of-designated-constants) | `Compressor` uses hardcoded values instead of designated constants |
| [NC-2](#nc-2-wrong-documentation-in-parsel2withdrawalmessage) | Wrong documentation in `_parseL2WithdrawalMessage` |
| [NC-3](#nc-3-inconsistencies-in-state-diffs-header-docs) | Inconsistencies in "State Diffs Header" Docs |
| [NC-4](#nc-4-rename-block-batch-as-part-of-the-codebase-refactor) | Rename block -> batch as part of the codebase refactor |
| [NC-5](#nc-5-update-forcedeployonaddresses-natspec-to-reflect-its-current-permissions) | Update `forceDeployOnAddresses()` NatSpec to reflect its current permissions |

## Low Issues

### <a name="L-1"></a>[L-1] Extra `notifyExecutionResult` gas cost is charged to users on L1->L2 transactions processing

`getExecuteL1TxAndGetRefund()` in the `Bootloader` calculates the gas costs of an L1->L2 transaction execution to set a `potentialRefund`:

Note how `notifyExecutionResult()` is executed in-between the `gasSpentOnExecution` and `gasBeforeExecution` calculations:

```yul
    let gasBeforeExecution := gas()
    success := ZKSYNC_NEAR_CALL_executeL1Tx(
        callAbi,
        txDataOffset
    )
@>  notifyExecutionResult(success)
    let gasSpentOnExecution := sub(gasBeforeExecution, gas())
  
    potentialRefund := sub(gasForExecution, gasSpentOnExecution)
```

Now note how this value is not used to calculate the L2 transaction execution cost (`gasSpentOnExecute`) in `l2TxExecution()`:

[bootloader.yul#L985-L993](https://github.com/code-423n4/2023-10-zksync/blob/main/code/system-contracts/bootloader/bootloader.yul#L985-L993)

```yul
        let gasBeforeExecute := gas()
        // for this one, we don't care whether or not it fails.
        success := ZKSYNC_NEAR_CALL_executeL2Tx(
            executeABI,
            txDataOffset
        )
  
        gasSpentOnExecute := add(gasSpentOnFactoryDeps, sub(gasBeforeExecute, gas()))
    }
  
@>  notifyExecutionResult(success)
```

[bootloader.yul#L1269-L1279](https://github.com/code-423n4/2023-10-zksync/blob/main/code/system-contracts/bootloader/bootloader.yul#L1269-L1279)

#### Impact

Gas costs of `notifyExecutionResult()` might not be significant for a refund on transactions of a single user, but they would make a difference in less refunds for an operator that processes a huge number of transactions.

#### Recommendation

Be consistent in the way gas costs are charged for executions of L2, and L1->L2 transactions.

Here's a recommended diff for omitting `notifyExecutionResult()` gas costs in L1->L2 transactions:

```diff
    let gasBeforeExecution := gas()
    success := ZKSYNC_NEAR_CALL_executeL1Tx(
        callAbi,
        txDataOffset
    )
-   notifyExecutionResult(success)
    let gasSpentOnExecution := sub(gasBeforeExecution, gas())
+   notifyExecutionResult(success)
  
    potentialRefund := sub(gasForExecution, gasSpentOnExecution)
```

### <a name="L-2"></a>[L-2] `if DUMMY_VERIFIER == true` block in `Executor` introduces risk of halting batch proving in production

The `proveBatches()` in the `Executor` has a conditional block that will be added during build time [depending on the Hardhat configuration](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/hardhat.config.ts#L30).

The conditional execution is given by comments, meaning that if the `DUMMY_VERIFIER` condition is not processed, all the following code will be included in the build.

In the case of the `Executor`, it will try to assert a wrong `chainId` and revert on any attempt to prove batches:

```solidity
  // #if DUMMY_VERIFIER

  // Additional level of protection for the mainnet
  assert(block.chainid != 1);
  /// ...
```

[Executor.sol#L348-L369](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L348-L369)

Note how it was differently handled on the codebase of the previous C4 audit.

In this case the condition is `#if DUMMY_VERIFIER == false`, meaning that in case of any error, the production code will be injected, but no test/playground code, making it safer, and preventing any production issue:

```solidity
  // #if DUMMY_VERIFIER == false
  bool successVerifyProof = s.verifier.verify_serialized_proof(proofPublicInput, _proof.serializedProof);
  require(successVerifyProof, "p"); // Proof verification fail

  // Verify the recursive part that was given to us through the public input
  bool successProofAggregation = _verifyRecursivePartOfProof(_proof.recurisiveAggregationInput);
  require(successProofAggregation, "hh"); // Proof aggregation must be valid
  // #endif
```

[Executor.sol#L259-L266](https://github.com/0xJuancito/zksync-million-dollar-review/blob/9a6b43ad88087e6c88f8f79ae0d1b430e498f9dc/2022-10-zksync/ethereum/contracts/zksync/facets/Executor.sol#L259-L266)

#### Recommendation

Given the high severity of halting batch proving on production, it is recommended to only set `DUMMY_VERIFIER == false` conditionals that would be safe to run on a production environment.

### <a name="L-3"></a>[L-3] Refund recipient addresses not aliased if transactions are requested on constructor

The `Mailbox` does the following check to alias a `refundRecipient` address during `requestL2Transaction()`.

`code.length` is `0` during the `constructor` execution of a smart contract. 

So, if a contract requests an L2 transaction **during their constructor** execution, and **set theirselves as the `refundRecipient`**, their address will not be aliased.

```solidity
  // If the `_refundRecipient` is not provided, we use the `_sender` as the recipient.
  address refundRecipient = _refundRecipient == address(0) ? _sender : _refundRecipient;
  // If the `_refundRecipient` is a smart contract, we apply the L1 to L2 alias to prevent foot guns.
  if (refundRecipient.code.length > 0) {
      refundRecipient = AddressAliasHelper.applyL1ToL2Alias(refundRecipient);
  }
```

[Mailbox.sol#L312-L314](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L312-L314)

Same thing happens with the bridges:

```solidity
    address refundRecipient = _refundRecipient;
    if (_refundRecipient == address(0)) {
        refundRecipient = msg.sender != tx.origin ? AddressAliasHelper.applyL1ToL2Alias(msg.sender) : msg.sender;
    }
```

[L1ERC20Bridge.sol#L195-L197](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/bridge/L1ERC20Bridge.sol#L195-L197)

#### Impact

Requesting L2 executions without the corresponding aliasing can lead to loss of funds, due to the impossibility of receiving the expected refunds, or perform an L1->L2 transaction to withdraw them.

#### Recommendation

Provide consistent aliasing, regardless of `code.length`. Consider providing an option to inform the transaction processing function if the address should be aliased.

### <a name="L-4"></a>[L-4] Double alias conversion in L1 bridge deposits

The `refundRecipient` is aliased on the bridges when it is not provided, and a smart contract is the one calling `deposit()`:

```solidity
  if (_refundRecipient == address(0)) {
      refundRecipient = msg.sender != tx.origin ? AddressAliasHelper.applyL1ToL2Alias(msg.sender) : msg.sender;
  }
```

[L1ERC20Bridge.sol#L195-L197](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/bridge/L1ERC20Bridge.sol#L195-L197)

The `Mailbox` can alias the address again in case the newly aliased `refundRecipient` is also a smart contract on L1.

Although the assumption that for a specific contract the probability of this event is very low, it is still possible for some addresses if we consider the whole spectrum of deployed and to be deployed contracts on Ethereum. In such an event, refunds that are a product of bridge transactions would be lost.

```solidity
    if (refundRecipient.code.length > 0) {
        refundRecipient = AddressAliasHelper.applyL1ToL2Alias(refundRecipient);
    }
```

[Mailbox.sol#L312-L314](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L312-L314)

#### Recommendation

Consider informing the `Mailbox` if the refund recipient was already aliased, and prevent the possibility of double aliasing.

### <a name="L-5"></a>[L-5] Wrong validation on max number of numberOfL2ToL1Logs

The `publishPubdataAndClearState()` function in `L1Messenger` has an incorrect validation where both terms of an inequality comparison are the same.

This leads to the requirement always being true, meaning that the parameter is actually not being validated.

`numberOfL2ToL1Logs` defines the number of logs that will be included in the logs merkle tree.

```solidity
    require(numberOfL2ToL1Logs <= numberOfL2ToL1Logs, "Too many L2->L1 logs");
```

[L1Messenger.sol#L206](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L206)

#### Impact

If the number of logs is greater than `L2_TO_L1_LOGS_MERKLE_TREE_LEAVES`, the operation will revert with an [index out of bounds error](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L216).

If the number is big enough but lower than `L2_TO_L1_LOGS_MERKLE_TREE_LEAVES`, it will successfully publish the pubdata, but may generate an invalid batch with more logs than expected by the zk-EVM, the Executor on L1, or a too expensive tx to process by the Verifier. This could lead to a higher impact as a DOS of the affected components and L1->L2 batch execution.

#### Recommendation

Set the `numberOfL2ToL1Logs` limit according to a proper constant, with a value lower than or equal to `L2_TO_L1_LOGS_MERKLE_TREE_LEAVES`, considering the rest of the system components limits.

### <a name="L-6"></a>[L-6] `isFacetFreezable()` returns false for non-existing facets instead of reverting

The function is responsible for informing "whether the facet can be frozen by the governor or always accessible".

If it returns `false`, it means that the facet is not freezable.

But in the case that the facet doesn't exist, it will still return `false`. This breaks trust assumptions, and can break integrations as it will bypass validations on non-existing facets.

```solidity
  /// @return isFreezable Whether the facet can be frozen by the governor or always accessible
  function isFacetFreezable(address _facet) external view returns (bool isFreezable) {
      Diamond.DiamondStorage storage ds = Diamond.getDiamondStorage();

      // There is no direct way to get whether the facet address is freezable,
      // so we get it from one of the selectors that are associated with the facet.
      uint256 selectorsArrayLen = ds.facetToSelectors[_facet].selectors.length;
      if (selectorsArrayLen != 0) {
          bytes4 selector0 = ds.facetToSelectors[_facet].selectors[0];
          isFreezable = ds.selectorToFacet[selector0].isFreezable;
      }
  }
```

[Getters.sol#L135-L145](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Getters.sol#L135-L145)

#### Recommendation

Revert the transaction in case the facet doesn't exist.

### <a name="L-7"></a>[L-7] Sending many logs via a single L1->L2 transactions can halt the publishing of pubdata

Users can send messages from L2 to L1 via the [sendToL1()](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L119) function of the `L1Messenger`. Each of this function calls generate a log that is hashed and rolled into the [chainedLogsHash](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L110) storage variable.

This rolling hash is recalculated among all the logs the operator decided to publish with the pubdata. And if the logs don't match [the whole publishing operation will be reverted](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L219-L222).

The number of logs is limited by the `L2_TO_L1_LOGS_MERKLE_TREE_LEAVES` constant which is equal to [2048](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/Constants.sol#L102). This means that the [rolling hash can calculated up to a maximum of 2048 logs](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L209-L218). If more logs were sent, the resulting hash will be different and the publishing operation will revert.

In the case of L2 transactions, the operator may decide to not include them.

In the case of L1->L2 transactions, the operator can't omit any transaction, and has to include all of them in order [as they are checked in order on L1 against the priority queue](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L281-L282).

The operator can still and decide to include less transactions, so that this number is not reached.

But there can still be the case that a single L1->L2 transaction generates enough logs that will make the publishing operation always revert (because of the mismatching rolling hash).

#### Impact

In this event, the operator will be forced to not include any L1->L2 transactions, as they have to be in order, and the malicious one would make the publishing operation always revert.

L1->L2 transaction confirmations on L1 will be halted as no batch would be generated including them.

Given the fact that the number of logs has to be > 2048, and to be generated by a single L1->L2 transaction, the attack vector is constrained to gas limitations on L2, but feasible in the event of config or gas changes.

#### Recommendation

Limit the number of logs that can be generated in a single transaction to a number lower than the merkle tree leaves. There already exists a [numberOfLogsToProcess](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L113) variable that may also be useful to achieve this solution.

### <a name="L-8"></a>[L-8] Generating a large pubdata via a single L1->L2 transaction can halt the publishing to L1 operation

Users can send messages from L2 to L1 via the [sendToL1()](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L119) function of the `L1Messenger`. 

Messages are included in the pubdata that is processed on the `publishPubdataAndClearState()` function. Each time a message is processed [an internal pointer is incremented](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L247).

When the pointer grows too large, it means that the pubdata is too long and [the whole publishing operation will revert](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L303).

If an L1->L2 transaction is capable of generating such message, the operator won't be able to continue processing any other L1->L2 transactions [as they are checked in order on L1 against the priority queue](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L281-L282).

It is not needed that the L1 transaction sends the big chunk of data. It may execute a contract on L2 that calls `L1Messenger::sendToL1()` with it. The current value to make this attack possible is [520_000 bytes](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/Constants.sol#L87).

It is also worth noting that the pubdata includes other data apart from messages that contribute to reaching this limit, as the state diffs.

Bytecode publication is another way a malicious actor can contribute to generating a large pubdata via `requestBytecodeL1Publication()`, per contract deployments. In this case, large contract bytecodes ([up to 2**16 bytes](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/common/libraries/L2ContractHelper.sol#L26)) can be submitted via `factoryDeps` ([limited to 32](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L294)) to perform the attack.

#### Impact

In this event, the operator will be forced to not include any L1->L2 transactions, as they have to be in order, and the malicious one would make the publishing operation always revert.

L1->L2 transaction confirmations on L1 will be halted as no batch would be generated including them.

The attack vector is constrained to gas costs on L2, but can lead to a critical scenario where L1->L2 txs commiting is halted.

#### Recommendation

Limit the maximum length of combined messages that can be sent on a single transaction via the `sendToL1()` or any possible function that affects these logs.

Limit the maximum size of combined bytecodes that can be requested for L1 publication on a single transaction.

### <a name="L-9"></a>[L-9] Upgrade OpenZeppelin dependencies

The current codebase is using OpenZeppelin v4.9.2 which contains published vulnerabilities (no direct impact on the current codebase). 

It is still recommended to upgrade dependencies to the latest version v4.9.3 to prevent any possible bugs that are already fixed there.

- [OpenZeppelin Security Updates](https://contracts.openzeppelin.com/security)
- [package.json#L15-L16](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/zksync/package.json#L15-L16)
- [package.json#L10](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/package.json#L10)

### <a name="L-10"></a>[L-10] Missing check for address(0) for the `allowList` in `DiamondInit::initialize()`

This is a missing instance from the bot report.

The `DiamondInit` performs `address(0)` checks for all of its parameters except for the `allowList`. Please consider adding it:

```solidity
    function initialize(InitializeData calldata _initalizeData) external reentrancyGuardInitializer returns (bytes32) {
        require(address(_initalizeData.verifier) != address(0), "vt");
        require(_initalizeData.governor != address(0), "vy");
        require(_initalizeData.admin != address(0), "hc");
        require(_initalizeData.priorityTxMaxGasLimit <= L2_TX_MAX_GAS_LIMIT, "vu");
```

[DiamondInit.sol#L55-L59](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/DiamondInit.sol#L55-L59)

### <a name="L-11"></a>[L-11] Missing getters in `Getters.sol`

`Getters` is missing some getter functions to obtain the values of private variables:

Please consider adding getter functions for the variables `zkPorterIsAvailable`, `totalDepositedAmountPerUser`, `admin`, and `pendingAdmin`

- [Storage.sol#L116](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/Storage.sol#L116)
- [Storage.sol#L133](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/Storage.sol#L133)
- [Storage.sol#L143-L145](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/Storage.sol#L143-L145)

### <a name="L-12"></a>[L-12] Prevent WETH deposits in L1ERC20Bridge

The L1ERC20Bridge contract can be used to bridge any ERC20 token from L1 to L2. This also includes the reference WETH implementation in L1, as this is just another ERC20 token.

The protocol already includes a special bridge for WETH using the L1WethBridge contract. This contract provides a dedicated implementation that, in conjunction with L2W

#### Impact

WETH bridged from L1 using L1ERC20Bridge will deploy a new L2 token representation for WETH (L2StandardERC20) different than the intended L2Weth contract used in L2WethBridge, will bridge L1 WETH assets into the L2Weth reference implementation in L2.

#### Recommendation

In `deposit()`, check that `_l1Token` is different from the WETH address in L1.

```diff
    function deposit(
        address _l2Receiver,
        address _l1Token,
        uint256 _amount,
        uint256 _l2TxGasLimit,
        uint256 _l2TxGasPerPubdataByte,
        address _refundRecipient
    ) public payable nonReentrant senderCanCallFunction(allowList) returns (bytes32 l2TxHash) {
        require(_amount != 0, "2T"); // empty deposit amount
        uint256 amount = _depositFunds(msg.sender, IERC20(_l1Token), _amount);
        require(amount == _amount, "1T"); // The token has non-standard transfer logic
+       require(_l1Token != l1WethAddress);
```

### <a name="L-13"></a>[L-13] Facet state is not entirely cleared when last selector is removed

Facet state in the Diamond library is maintained by keeping track of the selectors associated with the facet (an address).

When the first selector is added a to new facet, it's state is initialized in `_saveFacetIfNew()`, and the the last selector is removed, it's state is cleared in `_removeFacet()`:

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/zksync/libraries/Diamond.sol#L262-L279

```solidity
262:     function _removeFacet(address _facet) private {
263:         DiamondStorage storage ds = getDiamondStorage();
264: 
265:         // Get index of `DiamondStorage.facets` of the facet and last element of array
266:         uint256 facetPosition = ds.facetToSelectors[_facet].facetPosition;
267:         uint256 lastFacetPosition = ds.facets.length - 1;
268: 
269:         // If the facet is not at the end of the array then move the last element to the facet position
270:         if (facetPosition != lastFacetPosition) {
271:             address lastFacet = ds.facets[lastFacetPosition];
272: 
273:             ds.facets[facetPosition] = lastFacet;
274:             ds.facetToSelectors[lastFacet].facetPosition = facetPosition.toUint16();
275:         }
276: 
277:         // Remove last element from the facets array
278:         ds.facets.pop();
279:     }
```

In the previous snippet of code, we can see that the facet is removed from the `ds.facets` array, but the corresponding entry in `ds.facetToSelectors[_faucet]` is not cleared.

#### Recommendation

In `_removeFacet()`, clear the associated state in the `facetToSelectors` mapping.

```diff
    function _removeFacet(address _facet) private {
        DiamondStorage storage ds = getDiamondStorage();

        // Get index of `DiamondStorage.facets` of the facet and last element of array
        uint256 facetPosition = ds.facetToSelectors[_facet].facetPosition;
        uint256 lastFacetPosition = ds.facets.length - 1;

        // If the facet is not at the end of the array then move the last element to the facet position
        if (facetPosition != lastFacetPosition) {
            address lastFacet = ds.facets[lastFacetPosition];

            ds.facets[facetPosition] = lastFacet;
            ds.facetToSelectors[lastFacet].facetPosition = facetPosition.toUint16();
        }

        // Remove last element from the facets array
        ds.facets.pop();
+       // Remove entry in facetToSelectors
+       delete ds.facetToSelectors[_facet];
    }
```

### <a name="L-14"></a>[L-14] Shadow operations will ultimately need to be revealed on-chain

The Governance contract supports the scheduling of _shadow_ operations. These operations are groups of on-chain actions that, due to their sensitive nature, are scheduled using its hash rather than the actions itself.

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/governance/Governance.sol#L142-L145

```solidity
142:     function scheduleShadow(bytes32 _id, uint256 _delay) external onlyOwner {
143:         _schedule(_id, _delay);
144:         emit ShadowOperationScheduled(_id, _delay);
145:     }
```

However, the operation still needs to be revealed when being executed using `execute()` or `executeInstant()`. This not only allows front-runners to anticipate the change and realize a potential attack before the scheduled operation is finally executed, but also opens the possibility that the operation itself fails, reverting the transaction and revealing the operation data without any concrete effect.

#### Recommendation

Utmost precaution should be taken when executing shadow operations. It is recommended to use private pools or flashbots to execute these types of operations, in order to both prevent front-running and also ensure successful execution of the transaction when being included in the blockchain.

### <a name="L-15"></a>[L-15] Potential gas bomb in Governance execution

Unsuccessful governance actions are reverted by bubbling the error and copying the response returned by the call:

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/governance/Governance.sol#L224-L234

```solidity
224:     function _execute(Call[] calldata _calls) internal {
225:         for (uint256 i = 0; i < _calls.length; ++i) {
226:             (bool success, bytes memory returnData) = _calls[i].target.call{value: _calls[i].value}(_calls[i].data);
227:             if (!success) {
228:                 // Propage an error if the call fails.
229:                 assembly {
230:                     revert(add(returnData, 0x20), mload(returnData))
231:                 }
232:             }
233:         }
234:     }
```

This can lead to a _gas bomb_ in which the callee intentionally returns a large response in order to incur in artificial gas costs.

#### Recommendation

Bound the size of the returned data (`returndatasize`) to a maximum constant, and copy the response to memory up to the capped size.

### <a name="L-16"></a>[L-16] Verifier params can be accidentally skipped during upgrades

Verifier params stored in the ZkSync diamond can be updated by executing an upgrade. The implementation can be found in the `_setVerifierParams()` function present in the the BaseZkSyncUpgrade contract:

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/upgrades/BaseZkSyncUpgrade.sol#L127-L139

```solidity
127:     function _setVerifierParams(VerifierParams calldata _newVerifierParams) private {
128:         if (
129:             _newVerifierParams.recursionNodeLevelVkHash == bytes32(0) ||
130:             _newVerifierParams.recursionLeafLevelVkHash == bytes32(0) ||
131:             _newVerifierParams.recursionCircuitsSetVksHash == bytes32(0)
132:         ) {
133:             return;
134:         }
135: 
136:         VerifierParams memory oldVerifierParams = s.verifierParams;
137:         s.verifierParams = _newVerifierParams;
138:         emit NewVerifierParams(oldVerifierParams, _newVerifierParams);
139:     }
```

As we can see in lines 128-134, if any of the 3 different values is zero, the implementation will simply skip the update of all parameters.

This could lead to an accidental scenario in which one or two of the parameters are scheduled to be updated, while at least another is accidentally set to zero. This will skip the update silently, without alerting of a potential inconsistent intention.

#### Recommendation

Check that either all parameters are zero, or that all parameters are not zero.

```solidity
    if (_newVerifierParams.recursionNodeLevelVkHash == bytes32(0)) {
        require(_newVerifierParams.recursionLeafLevelVkHash == bytes32(0) && _newVerifierParams.recursionCircuitsSetVksHash == bytes32(0));
        return;
    ) else {
        require(_newVerifierParams.recursionLeafLevelVkHash != bytes32(0) && _newVerifierParams.recursionCircuitsSetVksHash != bytes32(0));
    }
    ...
```

### <a name="L-17"></a>[L-17] Governance contract does not implement ERC721 or ERC1155 receivers

The [Governance](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/governance/Governance.sol#L20) contract is an adaptation of the [TimelockController](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.2/contracts/governance/TimelockController.sol) contract present in the OpenZeppelin library.

One of the differences with the OpenZeppelin implementation is that the Governance check doesn't implement the IERC721Receiver and IERC1155Receiver interfaces. These provide the required callbacks needed by the ERC721 and ERC1155 standards to receive these kinds of tokens when the receiver is a contract.

#### Recommendation

As the Governance contract is essentially a timelocked wrapper around a multisig account, it is recommended to implement the IERC721Receiver and IERC1155Receiver interfaces, in order to support these types of tokens and have a future proof implementation in case the contract needs to handle them.

### <a name="L-18"></a>[L-18] Potential missing validation in `SystemContext::publishTimestampDataToL1()`

The `publishTimestampDataToL1()` function is used by the bootloader in L2 to publish timestamp data to L1 by sending a L2->L1 system log.

https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/SystemContext.sol#L379-L395

```solidity
379:     function publishTimestampDataToL1() external onlyCallFromBootloader {
380:         (uint128 currentBatchNumber, uint128 currentBatchTimestamp) = getBatchNumberAndTimestamp();
381:         (, uint128 currentL2BlockTimestamp) = getL2BlockNumberAndTimestamp();
382: 
383:         // The structure of the "setNewBatch" implies that currentBatchNumber > 0, but we still double check it
384:         require(currentBatchNumber > 0, "The current batch number must be greater than 0");
385:         bytes32 prevBatchHash = batchHash[currentBatchNumber - 1];
386: 
387:         // In order to spend less pubdata, the packed version is published
388:         uint256 packedTimestamps = (uint256(currentBatchTimestamp) << 128) | currentL2BlockTimestamp;
389: 
390:         SystemContractHelper.toL1(
391:             false,
392:             bytes32(uint256(SystemLogKey.PACKED_BATCH_AND_L2_BLOCK_TIMESTAMP_KEY)),
393:             bytes32(packedTimestamps)
394:         );
395:     }
```

Line 385 fetches the previous batch hash from the `batchHash` storage mapping, but as we can see in the implementation the variable `prevBatchHash` is not used at all. 

#### Impact

It can be suspected that the `prevBatchHash` is part of a missing validation, such as checking if this value is present, i.e. `require(prevBatchHash != bytes32(0))`, or checking the stored previous batch hash matches an expected value, i.e. `require(prevBatchHash == expectedPrevBatchHash`.

#### Recommendation

Ensure there is no missing validation in relation to the unused variable `prevBatchHash`, if not then delete line 385 to avoid any potential confusion.

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] `Compressor` uses hardcoded values instead of designated constants

Several constants are declared on the `Constants` file, but are not used. The `Compressor` is currently hardcoding these values [like in these cases](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/Compressor.sol#L144-L146), and importing [constant that it doesn't use](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/Compressor.sol#L13-L21).

```solidity
/// @dev The length of the derived key in bytes inside compressed state diffs.
uint256 constant DERIVED_KEY_LENGTH = 32;
/// @dev The length of the enum index in bytes inside compressed state diffs.
uint256 constant ENUM_INDEX_LENGTH = 8;
/// @dev The length of value in bytes inside compressed state diffs.
uint256 constant VALUE_LENGTH = 32;

/// @dev The length of the compressed initial storage write in bytes.
uint256 constant COMPRESSED_INITIAL_WRITE_SIZE = DERIVED_KEY_LENGTH + VALUE_LENGTH;
/// @dev The length of the compressed repeated storage write in bytes.
uint256 constant COMPRESSED_REPEATED_WRITE_SIZE = ENUM_INDEX_LENGTH + VALUE_LENGTH;

/// @dev The position from which the initial writes start in the compressed state diffs.
uint256 constant INITIAL_WRITE_STARTING_POSITION = 4;

/// @dev Each storage diffs consists of the following elements:
/// [20bytes address][32bytes key][32bytes derived key][8bytes enum index][32bytes initial value][32bytes final value]
/// @dev The offset of the deriived key in a storage diff.
uint256 constant STATE_DIFF_DERIVED_KEY_OFFSET = 52;
/// @dev The offset of the enum index in a storage diff.
uint256 constant STATE_DIFF_ENUM_INDEX_OFFSET = 84;
/// @dev The offset of the final value in a storage diff.
uint256 constant STATE_DIFF_FINAL_VALUE_OFFSET = 124;
```

[Constants.sol#L104-L126](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/Constants.sol#L104-L126)

#### Recommendation

It is recommended that the `Compressor` uses the corresponding values in `Constants` to prevent any integration issue with other components that use them.

### <a name="NC-2"></a>[NC-2] Wrong documentation in `_parseL2WithdrawalMessage`

The `_parseL2WithdrawalMessage()` function in the `Mailbox` wrongly documents the possible `length` of an extended withdraw message, which comes from [L2EthToken](https://github.com/code-423n4/2023-10-zksync/blob/main/code/system-contracts/contracts/L2EthToken.sol#L123).

Note that `address l2Sender` is wrongly noted as `32`. Also note that the result `68` is wrongly calculated.

```solidity
  // 2. The message that is sent by `withdrawWithMessage(address _l1Receiver, bytes calldata _additionalData)`
  // It should be equal to the length of the following:
  // bytes4 function signature + address l1Receiver + uint256 amount + address l2Sender + bytes _additionalData =
  // = 4 + 20 + 32 + 32 + _additionalData.length >= 68 (bytes).
```

[Mailbox.sol#L415-L418](https://github.com/code-423n4/2023-10-zksync/blob/7ed3944429f437a611c32e782a12b320f6a67c17/code/contracts/ethereum/contracts/zksync/facets/Mailbox.sol#L415-L418)

This extended message is the same as the one expected by the `_parseL2EthWithdrawalMessage()` function in the `L1WethBridge`.

Note how it is correctly calculated here:

```solidity
  // Check that the message length is correct.
  // additionalData (WETH withdrawal data): l2 sender address + weth receiver address = 20 + 20 = 40 (bytes)
  // It should be equal to the length of the function signature + eth receiver address + uint256 amount +
  // additionalData = 4 + 20 + 32 + 40 = 96 (bytes).
  require(_message.length == 96, "Incorrect ETH message with additional data length");
```

[L1WethBridge.sol#L275-L279](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/contracts/ethereum/contracts/bridge/L1WethBridge.sol#L275-L279)

#### Recommendation

Fix the `_parseL2WithdrawalMessage()` documentation:

```diff
  // 2. The message that is sent by `withdrawWithMessage(address _l1Receiver, bytes calldata _additionalData)`
  // It should be equal to the length of the following:
  // bytes4 function signature + address l1Receiver + uint256 amount + address l2Sender + bytes _additionalData =
- // = 4 + 20 + 32 + 32 + _additionalData.length >= 68 (bytes).
+ // = 4 + 20 + 32 + 20 + _additionalData.length >= 76 (bytes).
```

### <a name="NC-3"></a>[NC-3] Inconsistencies in "State Diffs Header" Docs

The "State Diffs Header" comment on the `publishPubdataAndClearState()` function in the `L1Messenger` does not match with what the implementation does.

Upon further checking of the [Statediff Compression Spec](https://github.com/code-423n4/2023-10-zksync/blob/main/docs/Smart%20contract%20Section/Handling%20pubdata%20in%20Boojum/State%20diff%20compression%20v1%20spec.md#values), we can see that the code implementation is correct, but the comment on the code is wrong:

```solidity
  /// header (1 byte version, 2 bytes total len of compressed, 1 byte enumeration index size, 2 bytes number of initial writes)
```

[L1Messenger.sol#L281](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L281)

```solidity
  require(
      uint256(uint8(bytes1(_totalL2ToL1PubdataAndStateDiffs[calldataPtr]))) ==
          STATE_DIFF_COMPRESSION_VERSION_NUMBER,
      "state diff compression version mismatch"
  );
  calldataPtr++;    // @audit "version" -> 1 byte OK

  uint24 compressedStateDiffSize = uint24(bytes3(_totalL2ToL1PubdataAndStateDiffs[calldataPtr:calldataPtr + 3]));
  calldataPtr += 3; // @audit "total len of compressed" -> 3 bytes vs 2 bytes in comment

  uint8 enumerationIndexSize = uint8(bytes1(_totalL2ToL1PubdataAndStateDiffs[calldataPtr]));
  calldataPtr++;     // @audit "enumeration index size" -> 1 byte OK

  // @audit-info missing 2 bytes of "number of initial writes" from comment
```

[L1Messenger.sol#L279-L299](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/contracts/L1Messenger.sol#L279-L299)

#### Recommendation

Fix the comments according to the actual implementation.

### <a name="NC-4"></a>[NC-4] Rename block -> batch as part of the codebase refactor

The codebase has been refactored from using the term `block` to `batch` in many parts.

Please consider renaming these instances as well:

```solidity
    callSystemContext({{RIGHT_PADDED_INCREMENT_TX_NUMBER_IN_BLOCK_SELECTOR}})
    callSystemContext({{RIGHT_PADDED_RESET_TX_NUMBER_IN_BLOCK_SELECTOR}})
```

- [RIGHT_PADDED_INCREMENT_TX_NUMBER_IN_BLOCK_SELECTOR](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/bootloader/bootloader.yul#L2547)
- [RIGHT_PADDED_RESET_TX_NUMBER_IN_BLOCK_SELECTOR](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/bootloader/bootloader.yul#L3814)

The intention can be inferred from the `process.ts` file:

```typescript
    RIGHT_PADDED_INCREMENT_TX_NUMBER_IN_BLOCK_SELECTOR: getPaddedSelector('SystemContext', 'incrementTxNumberInBatch'),
    RIGHT_PADDED_RESET_TX_NUMBER_IN_BLOCK_SELECTOR: getPaddedSelector('SystemContext', 'resetTxNumberInBatch'),
```

- [RIGHT_PADDED_INCREMENT_TX_NUMBER_IN_BLOCK_SELECTOR](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/scripts/process.ts#L99)
- [RIGHT_PADDED_RESET_TX_NUMBER_IN_BLOCK_SELECTOR](https://github.com/code-423n4/2023-10-zksync/blob/1fb4649b612fac7b4ee613df6f6b7d921ddd6b0d/code/system-contracts/scripts/process.ts#L101)

### <a name="NC-5"></a>[NC-5] Update `forceDeployOnAddresses()` NatSpec to reflect its current permissions

`forceDeployOnAddresses()` has been updated to also allow the `COMPLEX_UPGRADER_CONTRACT` to execute it.

Please consider adding this to the `@dev` section of the NatSpec:

```solidity
  /// @notice This method is to be used only during an upgrade to set bytecodes on specific addresses.
  /// @dev We do not require `onlySystemCall` here, since the method is accessible only
  /// by `FORCE_DEPLOYER`.
  function forceDeployOnAddresses(ForceDeployment[] calldata _deployments) external payable {
      require(
          msg.sender == FORCE_DEPLOYER || msg.sender == address(COMPLEX_UPGRADER_CONTRACT),
          "Can only be called by FORCE_DEPLOYER or COMPLEX_UPGRADER_CONTRACT"
      );
```

