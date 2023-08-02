# Report

- Non Critical Issues (5)
- Low Issues (5)

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Bool expression compared to literal value | 2 |
| [NC-2](#NC-2) | Import declarations should import specific symbols | 31 |
| [NC-3](#NC-3) | Use `uint256` instead of the `uint` alias | 16 |
| [NC-4](#NC-4) | Use constants for literal or magic values | 5 |
| [NC-5](#NC-5) | Use `rethAddress` function instead of duplicating functionality | 2 |

### <a name="NC-1"></a>[NC-1] Bool expression compared to literal value
Bool expressions do not need to be compared against a literal value. For example, `aBoolExpression == true` can be directly stated as `aBoolExpression`, or `aBoolExpression == false` as `!aBoolExpression`.

*Instances (2)*:
```solidity
File: contracts/SafEth/SafEth.sol

65:         require(pauseStaking == false, "staking is paused");

117:         require(pauseUnstaking == false, "unstaking is paused");

```

### <a name="NC-2"></a>[NC-2] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (31)*:
```solidity
File: contracts/SafEth/SafEth.sol

4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import "../interfaces/IWETH.sol";

6: import "../interfaces/uniswap/ISwapRouter.sol";

7: import "../interfaces/lido/IWStETH.sol";

8: import "../interfaces/lido/IstETH.sol";

9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

10: import "./SafEthStorage.sol";

11: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

4: import "../../interfaces/IDerivative.sol";

5: import "../../interfaces/frax/IsFrxEth.sol";

6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

7: import "../../interfaces/rocketpool/RocketStorageInterface.sol";

8: import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";

9: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";

10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";

11: import "../../interfaces/IWETH.sol";

12: import "../../interfaces/uniswap/ISwapRouter.sol";

13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

14: import "../../interfaces/uniswap/IUniswapV3Factory.sol";

15: import "../../interfaces/uniswap/IUniswapV3Pool.sol";

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

4: import "../../interfaces/IDerivative.sol";

5: import "../../interfaces/frax/IsFrxEth.sol";

6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

8: import "../../interfaces/curve/IFrxEthEthPool.sol";

9: import "../../interfaces/frax/IFrxETHMinter.sol";

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

4: import "../../interfaces/IDerivative.sol";

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

7: import "../../interfaces/curve/IStEthEthPool.sol";

8: import "../../interfaces/lido/IWStETH.sol";

```

### <a name="NC-3"></a>[NC-3] Use `uint256` instead of the `uint` alias
Prefer using the `uint256` type definition over its `uint` alias.

*Instances (16)*:
```solidity
File: contracts/SafEth/SafEth.sol

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);

27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

28:     event WeightChange(uint indexed index, uint weight);

28:     event WeightChange(uint indexed index, uint weight);

31:         uint weight,

32:         uint index

72:         for (uint i = 0; i < derivativeCount; i++)

89:         for (uint i = 0; i < derivativeCount; i++) {

99:             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

149:         for (uint i = 0; i < derivativeCount; i++) {

158:         for (uint i = 0; i < derivativeCount; i++) {

219:         uint _derivativeIndex,

220:         uint _slippage

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

183:             uint rethPerEth = (10 ** 36) / poolPrice();

```

### <a name="NC-4"></a>[NC-4] Use constants for literal or magic values

Consider defining constants for literal or magic values as it improves readability and prevents duplication of config values.

*Instances (5)*:

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L55
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L38
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35

### <a name="NC-5"></a>[NC-5] Use `rethAddress` function instead of duplicating functionality to retrieve the RETH token address

*Instances (2)*:

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L187-L193
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L229-L235

## Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Contract files should define a locked compiler version | 4 |
| [L-2](#L-2) | Rounding down when dividing by weight will leave some ETH leftovers  | 2 |
| [L-3](#L-3) | `adjustWeight` function should validate derivative index is within range | - |
| [L-4](#L-4) | `ethPerDerivative` function has confusing semantics | - |
| [L-5](#L-5) | Protect access to `receive()` function | - |

### <a name="L-1"></a>[L-1] Contract files should define a locked compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

*Instances (4)*:
```solidity
File: contracts/SafEth/SafEth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;

```

### <a name="L-2"></a>[L-2] Rounding down when dividing by weight will leave some ETH leftovers

When splitting ETH amounts by weight in order to determine how much corresponds to each derivative, there will be some leftovers that won't be considered due to the rounding. Consider assigning these leftovers to any arbitrary derivative, an easy strategy would be to keep track of how much balance has been already deposited in the derivatives and assign the remaining balance from the deposit to the last derivative.

This happens in the `stake` and `rebalanceToWeights` functions:

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149

### <a name="L-3"></a>[L-3] `adjustWeight` function should validate derivative index is within range

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165

As the underlying data structure is a mapping instead of an array, it is possible to adjust the weight of an out-of-bounds index that will correspond to a invalid or non-existent derivative. The function should validate that the given index is within the valid range.

```solidity
  function adjustWeight(
      uint256 _derivativeIndex,
      uint256 _weight
  ) external onlyOwner {
+     require(_derivativeIndex < derivativeCount);
      weights[_derivativeIndex] = _weight;
      uint256 localTotalWeight = 0;
      for (uint256 i = 0; i < derivativeCount; i++)
          localTotalWeight += weights[i];
      totalWeight = localTotalWeight;
      emit WeightChange(_derivativeIndex, _weight);
  }
```

### <a name="L-4"></a>[L-4] `ethPerDerivative` function has confusing semantics

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/interfaces/IDerivative.sol#L15

The `ethPerDerivative` function implemented by the different derivatives takes an amount as the argument which hints that this function should return the ETH value corresponding to the amount sent as a parameter of the derivate. This is not the case in neither of the 3 derivatives implemented, all of these return the ETH value for one unit (1e18, due to all tokens having 18 decimals) of the underlying derivative asset (wstETH, sfrxETH and RETH). 

The amount paramater is mostly ignored by the implementations, except for the Reth derivative which uses it to switch between using the Rocker Pool protocol or the Uniswap pool.

Consider renaming the function to something more appropiate that reflects the semantics of the current implementation and less confusing in terms of its interface.

### <a name="L-5"></a>[L-5] Protect access to `receive()` function

All major contracts implement the `receive` function to allow incoming ETH payments: derivatives need to accept ETH when unstaked or swapped, and SafEth needs to accept ETH coming from derivatives when unstaking or rebalancing.

As a precautionary measure, the different instances of this function can be protected to allow only transfers coming from the intended callers. In the case of SafEth these are the configured derivatives, and in the case of each derivative it should be the address of the corresponding protocol or exchange pool.
