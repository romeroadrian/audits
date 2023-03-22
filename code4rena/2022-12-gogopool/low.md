## Incorrectly imported contract

`ERC20` contract is imported from the `ERC4626.sol` file. Occurrences:

- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/MinipoolManager.sol#L16
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Staking.sol#L11
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/tokens/TokenggAVAX.sol#L18

## Missing event for important parameter change	

Important parameter or configuration changes should trigger an event to allow being tracked off-chain:

Occurrences:

- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Ocyticus.sol#L22
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Ocyticus.sol#L27
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Ocyticus.sol#L32
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Oracle.sol#L28
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ProtocolDAO.sol#L190
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ProtocolDAO.sol#L198

## Function `setClaimingContractPct` should validate sum of percentages 

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ProtocolDAO.sol#L107

The function validates the parameter is at most 100%, but fails to verify that the sum of `ProtocolDAO.ClaimingContractPct.ClaimMultisig`, `ProtocolDAO.ClaimingContractPct.ClaimNodeOp` and `ProtocolDAO.ClaimingContractPct.ClaimProtocolDAO` is 100%.

## Missing field in `Staking.getStaker` function

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Staking.sol#L405

The function misses to assign the `avaxAssignedHighWater` field in the returned `Staker` struct.

## Unneeded imports

Imported contract not used in the current file:

- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Vault.sol#L7
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Vault.sol#L8
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/tokens/TokenggAVAX.sol#L7
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ClaimNodeOp.sol#L5
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ClaimNodeOp.sol#L10
- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Staking.sol#L5

## Follow "Checks Effects Interactions" pattern

- In `Vault.depositToken`, the external interaction `tokenContract.safeTransferFrom(..)` happens before the `tokenBalances` variable update.

## Unneeded variable cast

Variable is casted to the same type as it is currently defined.

Occurrences:

- https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Vault.sol#L157

## Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions	

Add some storage gap to base contracts to allow adding storage variables in future upgrades.

Occurrences:

- `ERC20Upgradeable` https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/tokens/upgradeable/ERC20Upgradeable.sol#L10
- `ERC4626Upgradeable` https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/tokens/upgradeable/ERC4626Upgradeable.sol#L11
- `BaseUpgradeable` https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/BaseUpgradeable.sol#L9
