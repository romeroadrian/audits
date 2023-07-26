# Report

- Non Critical Issues (2)
- Low Issues (4)

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Import declarations should import specific symbols | 17 |
| [NC-2](#NC-2) | `JBFundingCycleMetadataResolver` library is not used and could be removed | - |

### <a name="NC-1"></a>[NC-1] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (17)*:
```solidity
File: contracts/JBXBuybackDelegate.sol

4: import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBController3_1.sol";

5: import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBFundingCycleDataSource.sol";

6: import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBPayDelegate.sol";

7: import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBPayoutRedemptionPaymentTerminal3_1.sol";

8: import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBFundingCycleBallot.sol";

9: import "@jbx-protocol/juice-contracts-v3/contracts/libraries/JBConstants.sol";

10: import "@jbx-protocol/juice-contracts-v3/contracts/libraries/JBFundingCycleMetadataResolver.sol";

11: import "@jbx-protocol/juice-contracts-v3/contracts/libraries/JBTokens.sol";

12: import "@jbx-protocol/juice-contracts-v3/contracts/structs/JBDidPayData.sol";

13: import "@jbx-protocol/juice-contracts-v3/contracts/structs/JBPayParamsData.sol";

15: import "@openzeppelin/contracts/access/Ownable.sol";

16: import "@openzeppelin/contracts/interfaces/IERC20.sol";

18: import "@paulrberg/contracts/math/PRBMath.sol";

20: import "@uniswap/v3-core/contracts/interfaces/IUniswapV3Pool.sol";

21: import "@uniswap/v3-core/contracts/interfaces/callback/IUniswapV3SwapCallback.sol";

22: import "@uniswap/v3-core/contracts/libraries/TickMath.sol";

24: import "./interfaces/external/IWETH9.sol";

```

### <a name="NC-2"></a>[NC-2] `JBFundingCycleMetadataResolver` library is not used and could be removed

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L40

The `JBFundingCycleMetadataResolver` library is not used and could be removed, along with the `JBFundingCycle` import.

## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Contract files should define a locked compiler version | 1 |
| [L-2](#L-2) | Unprotected access to `payParams()` may allow anyone to set `mintedAmount` and `reservedRate` | - |
| [L-3](#L-3) | Validate Uniswap pool is using WETH and the project token | - |
| [L-4](#L-4) | Converting a negative `int256` into a positive value may revert | 2 |

### <a name="L-1"></a>[L-1] Contract files should define a locked compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

*Instances (1)*:
```solidity
File: contracts/JBXBuybackDelegate.sol

2: pragma solidity ^0.8.16;

```

### <a name="L-2"></a>[L-2] Unprotected access to `payParams()` may allow anyone to set `mintedAmount` and `reservedRate`

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144

The `payParams()` function is intended to be called by the JuiceBox terminal, however its access is unprotected and anyone could call it. In particular, this can be used to arbitrarily modify the values of `mintedAmount` and `reservedRate`.

### <a name="L-3"></a>[L-3] Validate Uniswap pool is using WETH and the project token

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L125

Ensure the given Uniswap pool is using WETH and the project token as the pool's pair.

### <a name="L-4"></a>[L-4] Converting a negative `int256` into a positive value may revert

*Instances (2)*:

- https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224
- https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L271

Converting a negative value of variable of type `int256` into a positive one by doing `-variable` will revert the operation if the variable equals `type(int256).min` as the result overflows the capacity of the type.
