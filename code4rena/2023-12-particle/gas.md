# Gas Report

## Summary

### Gas Optimizations

Total of **12 findings**:

|ID|Finding|
|:--:|:---|
| [G-1](#g-1-uniswapv3-pool-address-can-be-computed-locally) | UniswapV3 Pool address can be computed locally |
| [G-2](#g-2-unchecked-math-in-base-refund) | Unchecked math in `Base.refund()` |
| [G-3](#g-3-feegrowthinside-is-monotonic-increasing) | `feeGrowthInside` is monotonic increasing |
| [G-4](#g-4-optimize-packing-in-lien-struct) | Optimize packing in `Lien` struct |
| [G-5](#g-5-duplicate-storage-variable-in-particleinforeader) | Duplicate storage variable in ParticleInfoReader |
| [G-6](#g-6-lien-is-fetched-twice-in-the-implementation-of-getlien) | Lien is fetched twice in the implementation of `getLien()` |
| [G-7](#g-7-no-need-to-make-nextrecordid-shorter-than-the-size-of-a-word) | No need to make `_nextRecordId` shorter than the size of a word |
| [G-8](#g-8-duplicate-checks-in-closeposition) | Duplicate checks in `_closePosition()` |
| [G-9](#g-9-consider-using-the-off-chain-storage-pattern) | Consider using the "off-chain storage" pattern |
| [G-10](#g-10-cache-storage-variables-in-openposition) | Cache storage variables in `openPosition()` |
| [G-11](#g-11-lien-id-can-be-safely-incremented-using-unchecked-math) | Lien id can be safely incremented using unchecked math |
| [G-12](#g-12-cache-storage-variables-in-liquidateposition) | Cache storage variables in `liquidatePosition()` |

## Gas Optimizations

### <a name="G-1"></a>[G-1] UniswapV3 Pool address can be computed locally

UniswapV3 deploys pools deterministically, which means that their addresses can be computed locally by using the token addresses and the fee. 

Instead of externally querying `IUniswapV3Factory.getPool()`, pool addresses can be using the [`PoolAddress.computeAddress()`](https://github.com/Uniswap/v3-periphery/blob/697c2474757ea89fec12a4e6db16a574fe259610/contracts/libraries/PoolAddress.sol#L33) helper function.

Instances:

- https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/libraries/Base.sol#L182
- https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/libraries/Base.sol#L325
- https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L92
- https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L131

### <a name="G-2"></a>[G-2] Unchecked math in `Base.refund()`

```solidity
75:     function refund(address recipient, address token, uint256 amountExpected, uint256 amountActual) internal {
76:         if (amountExpected > amountActual) {
77:             TransferHelper.safeTransfer(token, recipient, amountExpected - amountActual);
78:         }
79:     }
```

Since `amountExpected` is guaranteed to be greater than `amountActual` by the check in line 76, the subtraction `amountExpected - amountActual` can be done using unchecked math.

### <a name="G-3"></a>[G-3] `feeGrowthInside` is monotonic increasing

The "feeGrowthInside" values that track fees in Uniswap are monotonic increasing (these never decrease with updated values).

The subtractions present in `getOwedFee()` (lines 363 and 368) can be done using unchecked math:

```solidity
354:     function getOwedFee(
355:         uint256 feeGrowthInside0X128,
356:         uint256 feeGrowthInside1X128,
357:         uint256 feeGrowthInside0LastX128,
358:         uint256 feeGrowthInside1LastX128,
359:         uint128 liquidity
360:     ) internal pure returns (uint128 token0Owed, uint128 token1Owed) {
361:         if (feeGrowthInside0X128 > feeGrowthInside0LastX128) {
362:             token0Owed = uint128(
363:                 FullMath.mulDiv(feeGrowthInside0X128 - feeGrowthInside0LastX128, liquidity, FixedPoint128.Q128)
364:             );
365:         }
366:         if (feeGrowthInside1X128 > feeGrowthInside1LastX128) {
367:             token1Owed = uint128(
368:                 FullMath.mulDiv(feeGrowthInside1X128 - feeGrowthInside1LastX128, liquidity, FixedPoint128.Q128)
369:             );
370:         }
371:     }
```

### <a name="G-4"></a>[G-4] Optimize packing in `Lien` struct

The `zeroForOne` field can be accommodated with the first set of values to make it fit in a single slot.

Taking the first 5 fields, we have `40 + 128 + 24 + 24 + 32 = 248` bits. A `bool` type, which is 8 bits, can be accommodated with these fields to fit everything in 256 bits.

```solidity
08:     struct Info {
09:         uint40 tokenId;
10:         uint128 liquidity;
11:         uint24 token0PremiumPortion;
12:         uint24 token1PremiumPortion;
13:         uint32 startTime;
14:         uint256 feeGrowthInside0LastX128;
15:         uint256 feeGrowthInside1LastX128;
16:         bool zeroForOne;
17:     }
```

### <a name="G-5"></a>[G-5] Duplicate storage variable in ParticleInfoReader

The reference to the ParticlePositionManager is duplicated in the storage space of ParticleInfoReader.

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L24-L26

```solidity
24:     address public PARTICLE_POSITION_MANAGER_ADDR;
25:     ParticlePositionManager internal _particlePositionManager;
26: 
```

Even though the declared types are different, these are both essentially an address that points to the same place. Consider removing one of these.

### <a name="G-6"></a>[G-6] Lien is fetched twice in the implementation of `getLien()`

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L302

The implementation of `getLien()` in the ParticleInfoReader contract fetches the lien from the ParticlePositionManager contract, and then calls the `getPremium()` which also fetches the same lien from the external contract.

Consider refactoring both functions so that lien is queried once in the implementation of `getLien()`.

### <a name="G-7"></a>[G-7] No need to make `_nextRecordId` shorter than the size of a word

In the current implementation of ParticlePositionManager, the data type for the `_nextRecordId` variable (the one that tracks the ids of liens) is `uint96`.

```solidity
37:     /* Variables */
38:     uint96 private _nextRecordId; ///@dev used for both lien and swap
39:     uint256 private _treasuryRate;
40:     // solhint-disable var-name-mixedcase
41:     address public DEX_AGGREGATOR;
42:     uint256 public FEE_FACTOR;
43:     uint128 public LIQUIDATION_REWARD_FACTOR;
44:     uint256 public LOAN_TERM;
```

As this variable isn't packed with any other adjacent storage (`_treasuryRate` will always be placed in the next storage slot since it is 256 bits long) there is no real need to define a shorter type then `uint256`. Using `uint96` will incur in unneeded gas costs as the variable needs to be sanitized while loading or saving it to storage.

### <a name="G-8"></a>[G-8] Duplicate checks in `_closePosition()`

The implementation of `_closePosition()` checks that the required amounts needed to recover the liquidity do not exceed the available tokens in the position:

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L413-L420

```solidity
413:         // the liquidity to add must be no less than the available amount
414:         /// @dev the max available amount contains the tokensOwed, will have another check in below at refundWithCheck
415:         if (
416:             cache.amountFromAdd > cache.collateralFrom + cache.tokenFromPremium - cache.amountSpent ||
417:             cache.amountToAdd > cache.amountReceived + cache.tokenToPremium
418:         ) {
419:             revert Errors.InsufficientRepay();
420:         }
```

However, the same checks are also performed later when refunding the borrower using `refundWithCheck()`:

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L460-L490

```solidity
460:         if (lien.zeroForOne) {
461:             cache.token0Owed = cache.token0Owed < cache.tokenToPremium ? cache.token0Owed : cache.tokenToPremium;
462:             cache.token1Owed = cache.token1Owed < cache.tokenFromPremium ? cache.token1Owed : cache.tokenFromPremium;
463:             Base.refundWithCheck(
464:                 borrower,
465:                 cache.tokenFrom,
466:                 cache.collateralFrom + cache.tokenFromPremium,
467:                 cache.amountSpent + cache.amountFromAdd + cache.token1Owed
468:             );
469:             Base.refundWithCheck(
470:                 borrower,
471:                 cache.tokenTo,
472:                 cache.amountReceived + cache.tokenToPremium,
473:                 cache.amountToAdd + cache.token0Owed
474:             );
475:         } else {
476:             cache.token0Owed = cache.token0Owed < cache.tokenFromPremium ? cache.token0Owed : cache.tokenFromPremium;
477:             cache.token1Owed = cache.token1Owed < cache.tokenToPremium ? cache.token1Owed : cache.tokenToPremium;
478:             Base.refundWithCheck(
479:                 borrower,
480:                 cache.tokenFrom,
481:                 cache.collateralFrom + cache.tokenFromPremium,
482:                 cache.amountSpent + cache.amountFromAdd + cache.token0Owed
483:             );
484:             Base.refundWithCheck(
485:                 borrower,
486:                 cache.tokenTo,
487:                 cache.amountReceived + cache.tokenToPremium,
488:                 cache.amountToAdd + cache.token1Owed
489:             );
490:         }
```

### <a name="G-9"></a>[G-9] Consider using the "off-chain storage" pattern

The ["off-chain storage" pattern](https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/off-chain-storage) can help reduce gas costs by moving the Lien structure off-chain and only storing the hash of the current state.

This pattern was present in the Particle Leverage Trading protocol.

### <a name="G-10"></a>[G-10] Cache storage variables in `openPosition()`

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L192

The implementation of `openPosition()` reads the `FEE_FACTOR` storage variable twice at lines 192 and 193.

### <a name="G-11"></a>[G-11] Lien id can be safely incremented using unchecked math

The `_nextRecordId` variable works as a counter that is incremented by one every time a new position is created. It would be impossible in practice for this variable to overflow.

### <a name="G-12"></a>[G-12] Cache storage variables in `liquidatePosition()`

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L350

The implementation of `liquidatePosition()` reads the `LIQUIDATION_REWARD_FACTOR` storage variable twice at lines 350 and 353.

