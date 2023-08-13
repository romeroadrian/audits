# AmbireAccount.sol contract

- Use `calldata` for the `txns` parameter in the `executeBatch()` function.  
  https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/AmbireAccount.sol#L217

- Use `calldata` for the `txn` variable in the `executeBatch()` function.  
  https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/AmbireAccount.sol#L220

# AmbireAccountFactory.sol contract

- In `deploySafe()` the check in line 59 (`addr != address(0)`) is not needed as it would be already covered by the check in line 60 (`addr == expectedAddr`).  
  https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/AmbireAccountFactory.sol#L59

# SignatureValidator.sol contract

- Usages of `readBytes32()` helper don't need to check the index parameter as all these cases are guaranteed to be within bounds due to checks in surrounding code.  
  - Usages in line 55 and 56 are already validated by the `require` in line 54.  
    https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/SignatureValidator.sol#L55-L56
  - Usage in line 101 is already validated by the `require` in line 96.  
    https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/SignatureValidator.sol#L101
    
- The check for `signer != address(0)` in line 105 is not needed as the call to `wallet.isValidSignature()` would have failed if the wallet address were the zero address.  
  https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/SignatureValidator.sol#L105

- `modeRaw` variable is already checked by Solidity to be a valid value during the casting to `SignatureMode`. The `LastUnused` enum value is also unneeded.
  https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/SignatureValidator.sol#L49-L50

- Calls to `trimToSize()` in `recoverAddrImpl()` can be done using unchecked math in the calculation of `sig.length - 1` as the signature length has been already validated to be greater than 0.  
  https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/SignatureValidator.sol#L69  
  https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/SignatureValidator.sol#L84  
  