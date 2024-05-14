# Report

## Summary

### Low Issues

Total of **4 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-values-cannot-be-set-to-zero-in-setgovernanceparameterproposal) | Values cannot be set to zero in SetGovernanceParameterProposal |
| [L-2](#l-2-inconsistent-revert-behavior-in-signature-validators) | Inconsistent revert behavior in signature validators |
| [L-3](#l-3-a-proposal-cannot-be-accepted-by-host-when-one-abdicates) | A proposal cannot be accepted by host when one abdicates |
| [L-4](#l-4-burn-doesnt-decrement-the-total-voting-power) | Burn doesn't decrement the total voting power |

## Low Issues

### <a name="L-1"></a>[L-1] Values cannot be set to zero in SetGovernanceParameterProposal

The SetGovernanceParameterProposal allows Parties to update their governance values. The implementation of the proposal allows configuring the `voteDuration`, `executionDelay` and `passThresholdBps`.

https://github.com/code-423n4/2023-10-party/blob/0ce3819de173f7688c9c834ce2cc758dd03c9bd2/contracts/proposals/SetGovernanceParameterProposal.sol#L25-L66

```solidity
25:     function _executeSetGovernanceParameter(
26:         IProposalExecutionEngine.ExecuteProposalParams memory params
27:     ) internal returns (bytes memory) {
28:         SetGovernanceParameterProposalData memory proposalData = abi.decode(
29:             params.proposalData,
30:             (SetGovernanceParameterProposalData)
31:         );
32:         if (proposalData.voteDuration != 0) {
33:             if (proposalData.voteDuration < 1 hours) {
34:                 revert InvalidGovernanceParameter(proposalData.voteDuration);
35:             }
36:             emit VoteDurationSet(
37:                 _getSharedProposalStorage().governanceValues.voteDuration,
38:                 proposalData.voteDuration
39:             );
40:             _getSharedProposalStorage().governanceValues.voteDuration = proposalData.voteDuration;
41:         }
42:         if (proposalData.executionDelay != 0) {
43:             if (proposalData.executionDelay > 30 days) {
44:                 revert InvalidGovernanceParameter(proposalData.executionDelay);
45:             }
46:             emit ExecutionDelaySet(
47:                 _getSharedProposalStorage().governanceValues.executionDelay,
48:                 proposalData.executionDelay
49:             );
50:             _getSharedProposalStorage().governanceValues.executionDelay = proposalData
51:                 .executionDelay;
52:         }
53:         if (proposalData.passThresholdBps != 0) {
54:             if (proposalData.passThresholdBps > 10000) {
55:                 revert InvalidGovernanceParameter(proposalData.passThresholdBps);
56:             }
57:             emit PassThresholdBpsSet(
58:                 _getSharedProposalStorage().governanceValues.passThresholdBps,
59:                 proposalData.passThresholdBps
60:             );
61:             _getSharedProposalStorage().governanceValues.passThresholdBps = proposalData
62:                 .passThresholdBps;
63:         }
64: 
65:         return "";
66:     }
```

As we can see in the previous snippet of code, the default value (zero) is used to discern if the value should be updated. 

While this allows the option to skip certain parameters when using the proposal, it is important to note that this will actually prevent setting those values to zero, if these values are intentionally being set to zero.

### <a name="L-2"></a>[L-2] Inconsistent revert behavior in signature validators

In the implementation of ProposalExecutionEngine, the function `isValidSignature()` returns `0` when the signature is invalid:

https://github.com/code-423n4/2023-10-party/blob/0ce3819de173f7688c9c834ce2cc758dd03c9bd2/contracts/proposals/ProposalExecutionEngine.sol#L225-L245

```solidity
225:     function isValidSignature(bytes32 hash, bytes memory signature) external view returns (bytes4) {
226:         IERC1271 validator = getSignatureValidatorForHash(hash);
227:         if (address(validator) == address(1)) {
228:             // Signature set by party to be always valid
229:             return IERC1271.isValidSignature.selector;
230:         }
231:         if (address(validator) != address(0)) {
232:             return validator.isValidSignature(hash, signature);
233:         }
234:         if (tx.origin == address(0)) {
235:             validator = getSignatureValidatorForHash(0);
236:             if (address(validator) == address(0)) {
237:                 // Use global off-chain signature validator
238:                 validator = IERC1271(
239:                     _GLOBALS.getAddress(LibGlobals.GLOBAL_OFF_CHAIN_SIGNATURE_VALIDATOR)
240:                 );
241:             }
242:             return validator.isValidSignature(hash, signature);
243:         }
244:         return 0;
245:     }
```

However, in OffChainSignatureValidator, `isValidSignature()` will revert due to different reasons when the signature cannot be validated (lines 54, 64 and 79).

https://github.com/code-423n4/2023-10-party/blob/0ce3819de173f7688c9c834ce2cc758dd03c9bd2/contracts/signature-validators/OffChainSignatureValidator.sol#L28-L80

```solidity
28:     function isValidSignature(bytes32 hash, bytes memory signature) external view returns (bytes4) {
29:         uint8 v;
30:         bytes32 r;
31:         bytes32 s;
32:         assembly {
33:             // First word of signature after size contains r
34:             r := mload(add(signature, 0x20))
35:             s := mload(add(signature, 0x40))
36:             // v is one byte which starts after s. type is uint8 so extra data will be ignored
37:             v := mload(add(signature, 0x41))
38:         }
39: 
40:         bytes memory message;
41:         assembly {
42:             // Raw message data begins after v. Overwriting part of s and v with size of `message`
43:             message := add(signature, 0x41)
44:             mstore(message, sub(mload(signature), 0x41))
45:         }
46: 
47:         // Recreate the message pre-hash from the raw data
48:         bytes memory encodedPacket = abi.encodePacked(
49:             "\x19Ethereum Signed Message:\n",
50:             Strings.toString(message.length),
51:             message
52:         );
53:         if (keccak256(encodedPacket) != hash) {
54:             revert MessageHashMismatch();
55:         }
56: 
57:         Party party = Party(payable(msg.sender));
58:         address signer = ecrecover(hash, v, r, s);
59:         uint96 signerVotingPowerBps = party.getVotingPowerAt(signer, uint40(block.timestamp)) *
60:             10000;
61: 
62:         if (signerVotingPowerBps == 0 && party.balanceOf(signer) == 0) {
63:             // Must own a party card or be delegatated voting power
64:             revert NotMemberOfParty();
65:         }
66: 
67:         uint96 totalVotingPower = party.getGovernanceValues().totalVotingPower;
68:         uint96 thresholdBps = signingThersholdBps[party];
69: 
70:         // Either threshold is 0 or signer votes above threshold
71:         if (
72:             thresholdBps == 0 ||
73:             (signerVotingPowerBps > totalVotingPower &&
74:                 signerVotingPowerBps / totalVotingPower >= thresholdBps)
75:         ) {
76:             return IERC1271.isValidSignature.selector;
77:         }
78: 
79:         revert InsufficientVotingPower();
80:     }
```

### <a name="L-3"></a>[L-3] A proposal cannot be accepted by host when one abdicates

Proposals in PartyGovernance can now be accepted by hosts to skip the veto period. This is implemented by snapshotting the number of hosts at the moment the proposal is created (line 571):

https://github.com/code-423n4/2023-10-party/blob/0ce3819de173f7688c9c834ce2cc758dd03c9bd2/contracts/party/PartyGovernance.sol#L553-L582

```solidity
553:     function propose(
554:         Proposal memory proposal,
555:         uint256 latestSnapIndex
556:     ) external returns (uint256 proposalId) {
557:         _assertActiveMember();
558:         proposalId = ++lastProposalId;
559:         // Store the time the proposal was created and the proposal hash.
560:         (
561:             _proposalStateByProposalId[proposalId].values,
562:             _proposalStateByProposalId[proposalId].hash
563:         ) = (
564:             ProposalStateValues({
565:                 proposedTime: uint40(block.timestamp),
566:                 passedTime: 0,
567:                 executedTime: 0,
568:                 completedTime: 0,
569:                 votes: 0,
570:                 totalVotingPower: _getSharedProposalStorage().governanceValues.totalVotingPower,
571:                 numHosts: numHosts,
572:                 numHostsAccepted: 0
573:             }),
574:             getProposalHash(proposal)
575:         );
576:         emit Proposed(proposalId, msg.sender, proposal);
577:         accept(proposalId, latestSnapIndex);
578: 
579:         // Notify third-party platforms that the governance NFT metadata has
580:         // updated for all tokens.
581:         emit BatchMetadataUpdate(0, type(uint256).max);
582:     }
```

When a host accepts the proposal, the variable `numHostsAccepted` is incremented to track how many hosts have accepted the proposal. The implementation then compares both variables to determine if all hosts have accepted the proposal:

https://github.com/code-423n4/2023-10-party/blob/0ce3819de173f7688c9c834ce2cc758dd03c9bd2/contracts/party/PartyGovernance.sol#L1122-L1127

```solidity
1122:     function _hostsAccepted(
1123:         uint8 snapshotNumHosts,
1124:         uint8 numHostsAccepted
1125:     ) private pure returns (bool) {
1126:         return snapshotNumHosts > 0 && snapshotNumHosts == numHostsAccepted;
1127:     }
```

If a host abdicates their role without transferring it to another account, the current number of hosts will be decremented, which means that ongoing proposals won't be able to reach the required number of hosts even if all current hosts vote for it.

### <a name="L-4"></a>[L-4] Burn doesn't decrement the total voting power

The new implementation of `burn()` in PartyGovernanceNFT doesn't decrement the total voting power when tokens are burned.

The `_burnAndUpdateVotingPower()` function (called from `burn()`) decrements the `mintedVotingPower`, clears `votingPowerByTokenId[tokenId]` and adjusts the voting power of the owner of the token:

https://github.com/code-423n4/2023-10-party/blob/0ce3819de173f7688c9c834ce2cc758dd03c9bd2/contracts/party/PartyGovernanceNFT.sol#L268-L305

```solidity
268:     function _burnAndUpdateVotingPower(
269:         uint256[] memory tokenIds,
270:         bool checkIfAuthorizedToBurn
271:     ) private returns (uint96 totalVotingPowerBurned) {
272:         for (uint256 i; i < tokenIds.length; ++i) {
273:             uint256 tokenId = tokenIds[i];
274:             address owner = ownerOf(tokenId);
275: 
276:             // Check if caller is authorized to burn the token.
277:             if (checkIfAuthorizedToBurn) {
278:                 if (
279:                     msg.sender != owner &&
280:                     getApproved[tokenId] != msg.sender &&
281:                     !isApprovedForAll[owner][msg.sender]
282:                 ) {
283:                     revert NotAuthorized();
284:                 }
285:             }
286: 
287:             // Must be retrieved before updating voting power for token to be burned.
288:             uint96 votingPower = votingPowerByTokenId[tokenId].safeCastUint256ToUint96();
289: 
290:             totalVotingPowerBurned += votingPower;
291: 
292:             // Update voting power for token to be burned.
293:             delete votingPowerByTokenId[tokenId];
294:             emit PartyCardIntrinsicVotingPowerSet(tokenId, 0);
295:             _adjustVotingPower(owner, -votingPower.safeCastUint96ToInt192(), address(0));
296: 
297:             // Burn token.
298:             _burn(tokenId);
299: 
300:             emit Burn(msg.sender, tokenId, votingPower);
301:         }
302: 
303:         // Update minted voting power.
304:         mintedVotingPower -= totalVotingPowerBurned;
305:     }
```

However, the total voting power `_getSharedProposalStorage().governanceValues.totalVotingPower` is not modified.

The implementation of `rageQuit()` (which burns tokens internally) adjusts the total voting power after calling `_burnAndUpdateVotingPower`, but in both versions of `burn()` the implementations don't make this adjustment.

