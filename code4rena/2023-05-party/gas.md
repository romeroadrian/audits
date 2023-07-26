# PartyGovernance contract

- The new storage variable `lastBurnTimestamp` of type `uint40` was added below other short storage variables (`emergencyExecuteDisabled`, `feeBps` and `feeRecipient`) presumably with the intention of being tightly packed into a single slot. As this variable `lastBurnTimestamp` isn't used in conjunction with the other variables in the slot, there isn't any advantage to this behavior and will only introduce overhead gas costs due to packing.  
  https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernance.sol#L203
  
- The new check for `lastBurnTimestamp == block.timestamp` can be moved up in the function in order to have this checked earlier. This would save gas in case of a revert because the checks above are more costly in terms of the involved operations.  
  https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernance.sol#L596-L598
  
# PartyGovernanceNFT contract

- Similar to the case in the PartyGovernance contract, the new storage variable `rageQuitTimestamp` of type `uint40` is being accommodated below other short storage variables (`tokenCount` and `mintedVotingPower`) to have it tightly packed in a single slot. As this variable isn't used in combination with the others, this packing only incurs extra gas costs due to overhead.  
  https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L55

- The check for `newRageQuitTimestamp == DISABLE_RAGEQUIT_PERMANENTLY` can be moved at the start of the function and before line 271 to early exit and abort reading the storage variable in case of a revert.  
  https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L274
  
- When rage quitting multiple NTFs the same token transfer is executed multiple times to account for withdrawals associated to each NFT. Consider adding the amounts associated with each NFT first and then execute a single withdrawal for each token in `withdrawTokens`.  
  https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L337  
  https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L343

- When rage quitting multiple NTFs, each call to `burn()` will execute multiple checks and operations that are unneeded or are repeated with no other effect than consuming gas. For example:
  - The `(totalVotingPower != 0 || !authority)` check is unneeded as `totalVotingPower` should be greater than zero.
  - Updates to `lastBurnTimestamp` should only be needed once.
  - Updates to mintedVotingPower and totalVotingPower and repeated for each call, these can be batched and updated with a single SSTORE.  
  https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L320
