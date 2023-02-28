## Redundant validation in `_checkAndUpdateRedeemLimit`

https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L642

The function `_checkAndUpdateRedeemLimit` in the `CashManager` contract validates that the amount is not 0, but this is redundant since the same amount has been validated against in the `minimumRedeemAmount` variable as part of the `requestRedemption` function.

## Usage of safe unchecked math

- CashManager line 865 can be unchecked due to the if guard in line 864.  
https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L865

- CashManager line 867 can be unchecked due to the if guard in line 866.  
https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/CashManager.sol#L867 

## Cash factories deploy a new implementation contract for each proxy

https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/factory/CashFactory.sol#L79
https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/factory/CashKYCSenderFactory.sol#L81
https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/factory/CashKYCSenderReceiverFactory.sol#L81

All three factories (`CashFactory`, `CashKYCSenderFactory` and `deployCashKYCSenderReceiver`) deploy an individual implementation contract when deploying a new proxy. 

This is not needed and can be safely removed. New proxies can point to the same implementation to save gas.

## Cash factories store a reference to the latest set of deployed contract in storage

https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/factory/CashFactory.sol#L49-L51
https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/factory/CashKYCSenderFactory.sol#L49-L51
https://github.com/code-423n4/2023-01-ondo/blob/main/contracts/cash/factory/CashKYCSenderReceiverFactory.sol#L49-L51

Each of the three factories (`CashFactory`, `CashKYCSenderFactory` and `deployCashKYCSenderReceiver`) will store a reference to each deployed contract (implementation, proxy and admin) in storage when a new token is deployed.

This shouldn't be needed and probably doesn't make any sense, and can be removed to save gas.

## Simplify KYC group roles in KYCRegistry

The `KYCRegistry` contract keeps a mapping between group level and the role identifier assigned to that group.

This logic may be simplified by taking the role identifier as a predefined function of the group level. For example, using `bytes32 GROUP_ROLE = keccak(kycRequirementGroup)`.

This eliminates the need to maintain the `kycGroupRoles`, which can be removed, and saves one storage lookup when calling the function of the KYCRegistry contract.


