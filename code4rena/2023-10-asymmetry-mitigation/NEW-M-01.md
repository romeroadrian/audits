# [ADRIRO-NEW-M-01] Manager authorization in VotiumStrategy still leaves room for unprotected access

## Summary

Access control has been added to the VotiumStrategy contract with the intention of restricting functionality only to AfEth. However, an error in the implementation still leaves the contract publicly accessible.

## Impact

In the updated codebase, the sponsor has introduced access control to the VotiumStrategy contract. This authorization is implemented in the `onlyManager` modifier:

https://github.com/asymmetryfinance/afeth/blob/74f340568480aa03d043e970fcf2578bea037cf6/contracts/strategies/votium/VotiumStrategyCore.sol#L95-L99

```solidity
95:     modifier onlyManager() {
96:         if (address(manager) != address(0) && msg.sender != manager)
97:             revert NotManager();
98:         _;
99:     }
```

As we can see in line 96, the check is only executed if `manager` is different from `address(0)`. If this is not the case, then the check is not enforced and the revert in line 97 can never be triggered. This means that when `manager == address(0)` access is still granted for any caller.

This modifier has been added to `depositRewards()`, `deposit()`, `requestWithdraw()` and `withdraw()`, which means that potentially all these functions can still be publicly accessible.

This is particularly relevant in relation to issue [M-07](https://github.com/code-423n4/2023-09-asymmetry-findings/issues/38) of the original report, as access control was introduced to mitigate this issue.

Note, additionally, that the current issue potentially affects [H-05](https://github.com/code-423n4/2023-09-asymmetry-findings/issues/23), since `depositRewards()` could still be publicly accessible and be used to purchase CVX with an arbitrary value for the `_cvxMinout` slippage parameter.

## Recommendation

Remove the condition to allow access if `manager` is `address(0)`. Additionally, check that `manager` is correctly initialized in `initialize()`.

```diff
    modifier onlyManager() {
-       if (address(manager) != address(0) && msg.sender != manager)
+       if (msg.sender != manager)
            revert NotManager();
        _;
    }
```

```diff
    function initialize(
        address _owner,
        address _rewarder,
        address _manager
    ) external initializer {
        bytes32 VotiumVoteDelegationId = 0x6376782e65746800000000000000000000000000000000000000000000000000;
        address DelegationRegistry = 0x469788fE6E9E9681C6ebF3bF78e7Fd26Fc015446;
        address votiumVoteProxyAddress = 0xde1E6A7ED0ad3F61D531a8a78E83CcDdbd6E0c49;
        ISnapshotDelegationRegistry(DelegationRegistry).setDelegate(
            VotiumVoteDelegationId,
            votiumVoteProxyAddress
        );
        rewarder = _rewarder;
+       require(_manager != address(0));
        manager = _manager;
        __ERC20_init("Votium AfEth Strategy", "vAfEth");
        _transferOwnership(_owner);
        chainlinkCvxEthFeed = AggregatorV3Interface(
            0xC9CbF687f43176B302F03f5e58470b77D07c61c6
        );
    }
```
