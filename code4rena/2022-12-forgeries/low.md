## Unused imports in VRFNFTRandomDraw.sol

- VRFCoordinatorV2

## Consider using Solidity time units in VRFNFTRandomDraw

The constants `HOUR_IN_SECONDS`, `WEEK_IN_SECONDS` and `MONTH_IN_SECONDS` can be expressed using [Solidity time units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units)

```solidity
uint256 immutable HOUR_IN_SECONDS = 1 hours;
uint256 immutable WEEK_IN_SECONDS = 1 weeks;
uint256 immutable MONTH_IN_SECONDS = 30 days;
```

## No need to cast `token` variable while emitting WinnerSentNFT 

`settings.token` is already of type `address`. No need to cast it again.

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L289

## Unused imports in VRFNFTRandomDrawFactory.sol

- IERC721EnumerableUpgradeable

## Add some storage padding in `OwnableUpgradeable``

The `OwnableUpgradeable` contract is an upgradeable contract that is used as a base contract. Consider adding storage padding in case it is needed to add storage variables to this contract.

```solidity
abstract contract OwnableUpgradeable is IOwnableUpgradeable, Initializable {
  
  ...

  uint256[48] private __gap;
}
```

## Prefer `_disableInitializers()` in implementation contract constructors

`VRFNFTRandomDrawFactory` and `VRFNFTRandomDraw` are contracts that are intended to be used as implementations, the former is the UUPS implementation contract for the factory proxy while the latter is the raffle implementation contract that is cloned in the factory. 

Both constructors have the `initializer` modifier which will have the effect of blocking the initializer, meaning an attacker can't call `initialize` on the implementations (particularly dangerous with the UUPS implementation since it can lead to destroying the contract). 

However, consider using `_disableInitializers()` instead, since it better fits the case and will also handle any eventual `reinitializer` added in the future.

## Different Solidity version specified in `Version` contract

Set `pragma solidity 0.8.16;` to match the other contracts.

