# [adriro-MR-M-03-ERROR]: Recovery transaction can be replayed after a cancellation

The mitigation of M-03 contains an error in the implementation of the fix. The original issue is still present.

## Impact

The report in M-03 describes an issue related to the replay of the recovery transaction. After a cancellation is executed, the same transaction that initiated the recovery procedure can be replayed since the nonce is not incremented after canceling the recovery.

The intended fix is present in commit [1c0b06fbbbdd9aac1285d4fc4949f5b84f923238](https://github.com/AmbireTech/ambire-common/commit/1c0b06fbbbdd9aac1285d4fc4949f5b84f923238). The updated implementation of `execute()` is as follows:

https://github.com/AmbireTech/ambire-common/blob/v2/contracts/AmbireAccount.sol#L131-L191

```solidity
131: 	function execute(Transaction[] calldata calls, bytes calldata signature) public payable {
132: 		uint256 currentNonce = nonce;
133: 		// NOTE: abi.encode is safer than abi.encodePacked in terms of collision safety
134: 		bytes32 hash = keccak256(abi.encode(address(this), block.chainid, currentNonce, calls));
135: 
136: 		address signerKey;
137: 		// Recovery signature: allows to perform timelocked calls
138: 		uint8 sigMode = uint8(signature[signature.length - 1]);
139: 
140: 		if (sigMode == SIGMODE_RECOVER || sigMode == SIGMODE_CANCEL) {
141: 			(bytes memory sig, ) = SignatureValidator.splitSignature(signature);
142: 			(RecoveryInfo memory recoveryInfo, bytes memory innerRecoverySig, address signerKeyToRecover) = abi.decode(
143: 				sig,
144: 				(RecoveryInfo, bytes, address)
145: 			);
146: 			signerKey = signerKeyToRecover;
147: 			bool isCancellation = sigMode == SIGMODE_CANCEL;
148: 			bytes32 recoveryInfoHash = keccak256(abi.encode(recoveryInfo));
149: 			require(privileges[signerKeyToRecover] == recoveryInfoHash, 'RECOVERY_NOT_AUTHORIZED');
150: 
151: 			uint256 scheduled = scheduledRecoveries[hash];
152: 			if (scheduled != 0 && !isCancellation) {
153: 				require(block.timestamp > scheduled, 'RECOVERY_NOT_READY');
154: 				nonce++;
155: 				delete scheduledRecoveries[hash];
156: 				emit LogRecoveryFinalized(hash, recoveryInfoHash, block.timestamp);
157: 			} else {
158: 				bytes32 hashToSign = isCancellation ? keccak256(abi.encode(hash, 0x63616E63)) : hash;
159: 				address recoveryKey = SignatureValidator.recoverAddrImpl(hashToSign, innerRecoverySig, true);
160: 				bool isIn;
161: 				for (uint256 i = 0; i < recoveryInfo.keys.length; i++) {
162: 					if (recoveryInfo.keys[i] == recoveryKey) {
163: 						isIn = true;
164: 						break;
165: 					}
166: 				}
167: 				require(isIn, 'RECOVERY_NOT_AUTHORIZED');
168: 				if (isCancellation) {
169: 					delete scheduledRecoveries[hash];
170: 					emit LogRecoveryCancelled(hash, recoveryInfoHash, recoveryKey, block.timestamp);
171: 				} else {
172: 					scheduledRecoveries[hash] = block.timestamp + recoveryInfo.timelock;
173: 					emit LogRecoveryScheduled(hash, recoveryInfoHash, recoveryKey, currentNonce, block.timestamp, calls);
174: 				}
175: 				return;
176: 			}
177: 		} else {
178: 			signerKey = SignatureValidator.recoverAddrImpl(hash, signature, true);
179: 			require(privileges[signerKey] != bytes32(0), 'INSUFFICIENT_PRIVILEGE');
180: 		}
181: 
182: 		// we increment the nonce to prevent reentrancy
183: 		// also, we do it here as we want to reuse the previous nonce
184: 		// and respectively hash upon recovery / canceling
185: 		// doing this after sig verification is fine because sig verification can only do STATICCALLS
186: 		nonce = currentNonce + 1;
187: 		executeBatch(calls);
188: 
189: 		// The actual anti-bricking mechanism - do not allow a signerKey to drop their own privileges
190: 		require(privileges[signerKey] != bytes32(0), 'PRIVILEGE_NOT_DOWNGRADED');
191: 	}
```

The patched line is 154. Note that this line belongs to a path that is executed when the recovery process is finally executed after the timelock, and has nothing to do with the cancellation of the process. Note also that the change is in fact a no operation, since the increment is overwritten by line 186:

1. Line 132 sets `currentNonce` to the actual value of `nonce`, say `N`.
2. Line 154 executes `nonce++` which sets the storage value of `nonce` to `N+1`.
3. Recovery bundle is executed and line 186 sets the storage value of `nonce` to `currentNonce + 1`, which means that `nonce` is still `N+1`.

The intended fix contains an error and fails to mitigate the issue. It is not clear if this was mistakenly placed in the wrong code path or if there was a confusion related to the different code paths that this function may take. In any case, the nonce is not incremented after a cancellation, which means that the replay attack is still possible.

## Recommendation

The `nonce` should be incremented in the path that executes the cancellation, as recommended in the report for M-03.

```solidity
...
  if (isCancellation) {
    delete scheduledRecoveries[hash];
+   nonce = currentNonce + 1;
    emit LogRecoveryCancelled(hash, recoveryInfoHash, recoveryKey, block.timestamp);
  } else {
    scheduledRecoveries[hash] = block.timestamp + recoveryInfo.timelock;
    emit LogRecoveryScheduled(hash, recoveryInfoHash, recoveryKey, currentNonce, block.timestamp, txns);
  }
  return;
  ...
```

