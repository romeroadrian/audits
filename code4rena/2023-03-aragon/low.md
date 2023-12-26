# QA Report

- [Non Critical Issues]()
- [Low Issues]()

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Import declarations should import specific symbols | 20 |
| [NC-2](#NC-2) | Use named parameters for mapping type declarations | 14 |

### <a name="NC-1"></a>[NC-1] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (20)*:
```solidity
File: src/core/dao/DAO.sol

5: import "@openzeppelin/contracts-upgradeable/utils/introspection/ERC165StorageUpgradeable.sol";

6: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

7: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

8: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

9: import "@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

10: import "@openzeppelin/contracts-upgradeable/token/ERC721/IERC721ReceiverUpgradeable.sol";

11: import "@openzeppelin/contracts-upgradeable/token/ERC1155/IERC1155Upgradeable.sol";

12: import "@openzeppelin/contracts-upgradeable/token/ERC1155/IERC1155ReceiverUpgradeable.sol";

13: import "@openzeppelin/contracts-upgradeable/utils/AddressUpgradeable.sol";

14: import "@openzeppelin/contracts/interfaces/IERC1271.sol";

```

```solidity
File: src/core/permission/PermissionManager.sol

5: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

7: import "./IPermissionCondition.sol";

8: import "./PermissionLib.sol";

```

```solidity
File: src/core/plugin/proposal/Proposal.sol

8: import "./IProposal.sol";

```

```solidity
File: src/core/plugin/proposal/ProposalUpgradeable.sol

8: import "./IProposal.sol";

```

```solidity
File: src/framework/utils/ens/ENSMigration.sol

7: import "@ensdomains/ens-contracts/contracts/registry/ENSRegistry.sol";

8: import "@ensdomains/ens-contracts/contracts/resolvers/PublicResolver.sol";

```

```solidity
File: src/framework/utils/ens/ENSSubdomainRegistrar.sol

5: import "@ensdomains/ens-contracts/contracts/registry/ENS.sol";

6: import "@ensdomains/ens-contracts/contracts/resolvers/Resolver.sol";

```

```solidity
File: src/utils/Proxy.sol

5: import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";

```

### <a name="NC-2"></a>[NC-2] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18

*Instances (14)*:
```solidity
File: src/core/permission/PermissionManager.sol

27:     mapping(bytes32 => address) internal permissionsHashed;

```

```solidity
File: src/core/utils/CallbackHandler.sol

11:     mapping(bytes4 => bytes4) internal callbackMagicNumbers;

```

```solidity
File: src/framework/plugin/repo/PluginRepo.sol

55:     mapping(uint8 => uint16) internal buildsPerRelease;

58:     mapping(bytes32 => Version) internal versions;

61:     mapping(address => bytes32) internal latestTagHashForPluginSetup;

```

```solidity
File: src/framework/plugin/setup/PluginSetupProcessor.sol

54:         mapping(bytes32 => uint256) preparedSetupIdToBlockNumber;

59:     mapping(bytes32 => PluginState) public states;

```

```solidity
File: src/framework/utils/InterfaceBasedRegistry.sol

25:     mapping(address => bool) public entries;

```

```solidity
File: src/plugins/governance/majority-voting/MajorityVotingBase.sol

142:         mapping(address => IMajorityVoting.VoteOption) voters;

189:     mapping(uint256 => Proposal) internal proposals;

```

```solidity
File: src/plugins/governance/multisig/Multisig.sol

38:         mapping(address => bool) approvers;

75:     mapping(uint256 => Proposal) internal proposals;

```

```solidity
File: src/plugins/token/MerkleDistributor.sol

29:     mapping(uint256 => uint256) private claimedBitMap;

```

```solidity
File: src/plugins/utils/Addresslist.sol

17:     mapping(address => CheckpointsUpgradeable.History) private _addresslistCheckpoints;

```

## Low Issues

### Missing calls to base initializers

Make sure to call base initializers, even if these are no-op.

- In `DAO` contract initializer:
  - `__ERC165Storage_init()`
  - `__UUPSUpgradeable_init()`
- In `PluginCloneable` initializer:
  - `__ERC165Upgradeable_init()`
- In `PluginUUPSUpgradeable` initializer:
  - `__ERC165Upgradeable_init()`
  - `__UUPSUpgradeable_init()`
- `ProposalUpgradeable` should define initializer and call `__ERC165Upgradeable_init()`

### `applyMultiTargetPermissions` function in `PermissionManager` should check that condition is not 

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/core/permission/PermissionManager.sol#L177-L184

If `item.operation == PermissionLib.Operation.GrantWithCondition` then the function should check that `item.condition` is not `PermissionLib.NO_CONDITION`, as this value is equivalent to `UNSET_FLAG` (both are `address(0)`).

### `createVersion` function in `PluginRepo` should validate `_release` to prevent overflow

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/framework/plugin/repo/PluginRepo.sol#L143

The `createVersion` function should validate that `_release >= latestRelease`, else subtraction in line 143 will overflow.

### Validate `initialOwner` address in `PluginRepoFactory.createPluginRepo`

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/framework/plugin/repo/PluginRepoFactory.sol#L31-L36

The function should validate that `_initialOwner != address(0)`.

### Validate `_maintainer` address in `PluginRepoFactory.createPluginRepoWithFirstVersion`

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/framework/plugin/repo/PluginRepoFactory.sol#L45-L59

The function should validate that `_maintainer != address(0)`.

### The `applyUpdate` function in `PluginSetupProcessor` should validate that the current version of the plugin

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/framework/plugin/setup/PluginSetupProcessor.sol#L500

The `applyUpdate` function should validate that the current version of the intended change is the actual current version at the time this function is being executed. This can be by computing the `appliedSetupId` with the current version and checking this against the stored `pluginState.currentAppliedSetupId` variable.

### TokenFactory should also grant the `CHANGE_DISTRIBUTOR_PERMISSION_ID` permission to the DAO

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/framework/utils/TokenFactory.sol#L137

After creating the `MerkleMinter` instance, the `createToken` function present in the `TokenFactory` contract should also grant the DAO the `CHANGE_DISTRIBUTOR_PERMISSION_ID` permission.

```
_managingDao.grant(merkleMinter, address(_managingDao), MerkleMinter(merkleMinter).CHANGE_DISTRIBUTOR_PERMISSION_ID);
```

### Multisig can early execute proposals when min approvals are reached

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/plugins/governance/multisig/Multisig.sol#L389

There's no need to wait until the proposal is closed as multisig approvals cannot be changed or revoked. The proposal can be early executed when the required approvals are met.

### CounterV2 setup example is missing "newVariable" in installation initializer

https://github.com/code-423n4/2023-03-aragon/blob/main/packages/contracts/src/plugins/counter-example/v2/CounterV2PluginSetup.sol#L48-L53

The setup should also include the `newVariable` parameter while encoding the initializer calldata.
