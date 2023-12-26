# Report

## Summary

### Low Issues

Total of **7 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-supported-assets-cannot-be-removed) | Supported assets cannot be removed |
| [L-2](#l-2-duplicate-access-list-in-rseth-contract) | Duplicate access list in RSETH contract |
| [L-3](#l-3-admin-could-get-locked-out-when-updating-lrtconfig) | Admin could get locked out when updating LRTConfig |
| [L-4](#l-4-missing-storage-gap-in-lrtconfigrolechecker) | Missing storage gap in LRTConfigRoleChecker |
| [L-5](#l-5-no-decimal-normalization-in-price-feeds) | No decimal normalization in price feeds |
| [L-6](#l-6-potential-denial-of-service-due-to-unbounded-gas-usage-in-gettotalassetdeposits) | Potential denial of service due to unbounded gas usage in `getTotalAssetDeposits()` |
| [L-7](#l-7-potential-overflow-in-getassetcurrentlimit) | Potential overflow in `getAssetCurrentLimit()` |

### Non Critical Issues

Total of **2 issues**:

|ID|Issue|
|:--:|:---|
| [NC-1](#nc-1-add-an-initializer-to-the-lrtconfigrolechecker-contract) | Add an initializer to the LRTConfigRoleChecker contract |
| [NC-2](#nc-2-use-uups-over-transparentproxy) | Use UUPS over TransparentProxy |

## Low Issues

### <a name="L-1"></a>[L-1] Supported assets cannot be removed

Configured assets in the LRTConfig contract can be added by calling `addNewSupportedAsset()`, but there is no current way of removing these assets if it is needed later.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L73-L75

```solidity
73:     function addNewSupportedAsset(address asset, uint256 depositLimit) external onlyRole(LRTConstants.MANAGER) {
74:         _addNewSupportedAsset(asset, depositLimit);
75:     }
```

Consider adding a function to let admins or managers to eventually remove an asset from the supported list.

### <a name="L-2"></a>[L-2] Duplicate access list in RSETH contract

The LRTConfig acts as an access list through the protocol, contracts inherit from LRTConfigRoleChecker that provides a link to the LRTConfig contract to check for access control.

However, the RSETH contract inherits from AccessControlUpgradeable and also uses LRTConfig, meaning this contract has two different access lists. Both of these access lists are mixed within the contract: mint and burn access go through the access list present in the contract itself, while pause is controlled via the `onlyLRTManager` modifier present in the LRTConfig access list.

This is confusing and error prone. Consider using just the access list defined by LRTConfig.

### <a name="L-3"></a>[L-3] Admin could get locked out when updating LRTConfig

Access control within the protocol is provided by an access list implemented in the LRTConfig contract.

This access list is used throughout the protocol by the LRTConfigRoleChecker mixin, which holds the reference to the current LRTConfig contract.

The config instance present in the LRTConfigRoleChecker contract can be updated by an admin using the `updateLRTConfig()` function:

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L47-L51

```solidity
47:     function updateLRTConfig(address lrtConfigAddr) external virtual onlyLRTAdmin {
48:         UtilLib.checkNonZeroAddress(lrtConfigAddr);
49:         lrtConfig = ILRTConfig(lrtConfigAddr);
50:         emit UpdatedLRTConfig(lrtConfigAddr);
51:     }
```

Once the new config takes place, the new access list from this instance will be effective. This means that if access control is not properly set up in the new instance, the admin could be locked out of the protocol.

Consider either:

- Using a two step process, where the change is made effective when the admin accepts the change in the config instance, or
- Check that the current admin is also an admin in the new config instance before applying the changes in `updateLRTConfig()`.

### <a name="L-4"></a>[L-4] Missing storage gap in LRTConfigRoleChecker

The LRTConfigRoleChecker contract is used as a base contract for upgradeable contracts but doesn't contain any storage gap to ensure enough space for future revisions.

```diff
  abstract contract LRTConfigRoleChecker {
      ILRTConfig public lrtConfig;
+     uint256[50] private __gap;
```

### <a name="L-5"></a>[L-5] No decimal normalization in price feeds

Chainlink feeds simply returns the price without checking for any decimal discrepancy.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L37-L39

```solidity
37:     function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
38:         return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
39:     }
```

In the case price feeds used by supported assets don't match in their decimals, the error will be carried forward during the calculation of the RSETH price in `getRSETHPrice()`, as numbers with different precision will be aggregated together.

Currently, feeds for all supported assets (stETH, cbETH and rETH) have 18 decimals, but caution must be taken if other assets are added.

### <a name="L-6"></a>[L-6] Potential denial of service due to unbounded gas usage in `getTotalAssetDeposits()`

The implementation of `getTotalAssetDeposits()` is composed of the asset balance in the DepositPool, and the aggregation of all balances and deposits in EigenLayer for each of the registered node delegators.

```solidity
47:     function getTotalAssetDeposits(address asset) public view override returns (uint256 totalAssetDeposit) {
48:         (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer) =
49:             getAssetDistributionData(asset);
50:         return (assetLyingInDepositPool + assetLyingInNDCs + assetStakedInEigenLayer);
51:     }
```

```solidity
71:     function getAssetDistributionData(address asset)
72:         public
73:         view
74:         override
75:         onlySupportedAsset(asset)
76:         returns (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer)
77:     {
78:         // Question: is here the right place to have this? Could it be in LRTConfig?
79:         assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));
80: 
81:         uint256 ndcsCount = nodeDelegatorQueue.length;
82:         for (uint256 i; i < ndcsCount;) {
83:             assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
84:             assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
85:             unchecked {
86:                 ++i;
87:             }
88:         }
89:     }
```

As node delegators are expected to grow over time, this may eventually cause a denial of service due to the unbounded gas usage required to compute the function.

Note that `getTotalAssetDeposits()` is a key function in the protocol, as it is used in `getRSETHPrice()` to calculate the RSETH price and in `getAssetCurrentLimit()`, which is then used to check for current deposit limits in `depositAsset()`.

A potential solution could be to track balances internally instead of aggregating all volume on the fly for each node delegator.

### <a name="L-7"></a>[L-7] Potential overflow in `getAssetCurrentLimit()`

The implementation of `getAssetCurrentLimit()` subtracts the configured limit for the asset to the current total deposits in the protocol:

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L51

```solidity
56:     function getAssetCurrentLimit(address asset) public view override returns (uint256) {
57:         return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
58:     }
```

As deposits held in EigenLayer are expected to grow over time, it is technically possible for `getTotalAssetDeposits(asset)` to be greater than `lrtConfig.depositLimitByAsset(asset)`, causing an overflow.

Consider returning `0` if this is this happens:

```solidity
    function getAssetCurrentLimit(address asset) public view override returns (uint256) {
        uint256 limit = lrtConfig.depositLimitByAsset(asset);
        uint256 balance = getTotalAssetDeposits(asset);
        
        if (limit > balance) {
          unchecked {
            return limit - balance;
          }
        }
        
        return 0;
    }
```

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] Add an initializer to the LRTConfigRoleChecker contract

All the contracts that inherit from LRTConfigRoleChecker have to initialize the `lrtConfig` variable in its own initializer, having this code duplicated across all contracts. 

Add an initializer to LRTConfigRoleChecker to have the derived contract use it during initialization.

```solidity
    function __LRTConfigRoleChecker_init(address lrtConfigAddr) internal onlyInitializing {
        __LRTConfigRoleChecker_init_unchained(lrtConfigAddr);
    }

    function __LRTConfigRoleChecker_init_unchained(address lrtConfigAddr) internal onlyInitializing {
        lrtConfig = ILRTConfig(lrtConfigAddr);
        emit UpdatedLRTConfig(lrtConfigAddr);
    }
```

### <a name="NC-2"></a>[NC-2] Use UUPS over TransparentProxy

The UUPS pattern is more flexible than the TransparentProxy as the upgrade logic resides in the implementation itself. It is also cheaper to use, since the Proxy doesn't need to check if the caller is the admin to switch between logics.

The general recommendation is to use UUPS over TransparentProxy, see [Transparent vs UUPS Proxies](https://docs.openzeppelin.com/contracts/4.x/api/proxy#transparent-vs-uups).

