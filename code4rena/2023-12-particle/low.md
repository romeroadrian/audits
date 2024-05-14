# Report

## Summary

### Low Issues

Total of **7 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-lp-tokens-cannot-be-removed) | LP tokens cannot be removed |
| [L-2](#l-2-lp-owners-cannot-selectively-choose-which-loans-not-to-renew) | LP owners cannot selectively choose which loans not to renew |
| [L-3](#l-3-particlepositionmanager-should-not-accept-erc721-transfer-from-nft-collections-other-than-uniswapv3) | ParticlePositionManager should not accept ERC721 transfer from NFT collections other than UniswapV3 |
| [L-4](#l-4-incompatibility-with-fee-on-transfer-tokens) | Incompatibility with fee-on-transfer tokens |
| [L-5](#l-5-missing-validations-in-particleinforeader-initializer) | Missing validations in ParticleInfoReader initializer |
| [L-6](#l-6-particleinforeader-getowedinfo-misses-to-return-information-about-the-trade-side) | `ParticleInfoReader.getOwedInfo()` misses to return information about the trade side |
| [L-7](#l-7-missing-validations-in-particlepositionmanager-initializer) | Missing validations in ParticlePositionManager initializer |

### Non Critical Issues

Total of **7 issues**:

|ID|Issue|
|:--:|:---|
| [NC-1](#nc-1-unused-parameter-in-openpositionparams-struct) | Unused parameter in OpenPositionParams struct |
| [NC-2](#nc-2-unnecessary-multicall-feature-in-particleinforeader) | Unnecessary multicall feature in ParticleInfoReader |
| [NC-3](#nc-3-uniswap-fee-tiers-can-be-dynamic) | Uniswap fee tiers can be dynamic |
| [NC-4](#nc-4-owner-variable-is-shadowed-in-getliquidityposition) | `owner` variable is shadowed in `getLiquidityPosition()` |
| [NC-5](#nc-5-unused-getliencache-struct) | Unused `GetLienCache` struct |
| [NC-6](#nc-6-unused-swapposition-library-in-particlepositionmanager) | Unused SwapPosition library in ParticlePositionManager |
| [NC-7](#nc-7-incorrect-call-to-base-contract-initializer) | Incorrect call to base contract initializer |

## Low Issues

### <a name="L-1"></a>[L-1] LP tokens cannot be removed

Besides minting positions in Uniswap, the protocol accepts existing positions by transferring the UniV3 NFT to the Particle contract. The `onERC721Received()` callback is triggered when executing a `safeTransferFrom()` and will correctly create the `lp` position in ParticlePositionManager.

However, while the position can be managed through the Particle contract, there's no existing functionality to recover the NFT back. 

### <a name="L-2"></a>[L-2] LP owners cannot selectively choose which loans not to renew

LP owners can reclaim liquidity from their loans, but calling `reclaimLiquidity()` will update the `renewalCutoffTime` variable, affecting all existing positions.

Owners of liquidity positions cannot selectively decide which loans not to renew, needing to reclaim all active positions even if just a single one needs to be pulled off.

### <a name="L-3"></a>[L-3] ParticlePositionManager should not accept ERC721 transfer from NFT collections other than UniswapV3

The implementation of ParticlePositionManager accepts NFT transfers via the `onERC721Received()` that is triggered 

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L98-L111

```solidity
098:     function onERC721Received(
099:         address,
100:         address from,
101:         uint256 tokenId,
102:         bytes calldata
103:     ) external override returns (bytes4) {
104:         if (msg.sender == Base.UNI_POSITION_MANAGER_ADDR) {
105:             // matched with Uniswap v3 position NFTs
106:             lps[tokenId] = LiquidityPosition.Info({owner: from, renewalCutoffTime: 0, token0Owed: 0, token1Owed: 0});
107:             (, , , , , , , uint128 liquidity, , , , ) = Base.UNI_POSITION_MANAGER.positions(tokenId);
108:             emit LiquidityPosition.SupplyLiquidity(tokenId, from, liquidity);
109:         }
110:         return this.onERC721Received.selector;
111:     }
```

As we can see, the function registers the liquidity position if the caller is the Uniswap V3 NFT collection, but also accepts other NFT collections as the correct selector value is returned for all cases.

### <a name="L-4"></a>[L-4] Incompatibility with fee-on-transfer tokens

The protocol is mostly incompatible with ERC20 implementations that take a fee or tax on transfer.

There are different occurrences across the codebase in which ERC20 transfers are expected to receive the same amount as tokens transferred, an invariant that doesn't hold in fee-on-transfer tokens.

Examples of this are `mint()` and `increaseLiquidity()` in the LiquidityPosition library, the `swap()` function of the SwapPosition library, or the margin transfers in `openPosition()` and `addPremium()` of the ParticlePositionManager contract.

### <a name="L-5"></a>[L-5] Missing validations in ParticleInfoReader initializer

The initializer of the ParticleInfoReader contract doesn't validate that the given address is not the empty address, a check that is present in the associated setter `updateParticleAddress()`.

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L37-L42

```solidity
37:     function initialize(address particleAddr) external initializer {
38:         __UUPSUpgradeable_init();
39:         __Ownable_init();
40:         PARTICLE_POSITION_MANAGER_ADDR = particleAddr;
41:         _particlePositionManager = ParticlePositionManager(particleAddr);
42:     }
```

Consider adding a check to ensure that `particleAddr != address(0)`.

### <a name="L-6"></a>[L-6] `ParticleInfoReader.getOwedInfo()` misses to return information about the trade side

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L353

The implementation of `getOwedInfo()` returns information using the `token0`/`token1` which doesn't indicate which is the "long" side of the position.

Consider using the `tokenFrom`/`tokenTo` naming scheme (and internally flipping these depending on the value of `zeroForOne`), or returning the lien's `zeroForOne` value so that the caller can interpret the returned values.

### <a name="L-7"></a>[L-7] Missing validations in ParticlePositionManager initializer

None of the arguments in the ParticlePositionManager initializer are checked to be valid, though all of these are validated in their respective setters.

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L60-L74

```solidity
60:     function initialize(
61:         address dexAggregator,
62:         uint256 feeFactor,
63:         uint128 liquidationRewardFactor,
64:         uint256 loanTerm,
65:         uint256 treasuryRate
66:     ) external initializer {
67:         __UUPSUpgradeable_init();
68:         __Ownable_init();
69:         DEX_AGGREGATOR = dexAggregator;
70:         FEE_FACTOR = feeFactor;
71:         LIQUIDATION_REWARD_FACTOR = liquidationRewardFactor;
72:         LOAN_TERM = loanTerm;
73:         _treasuryRate = treasuryRate;
74:     }
```

These missing validation could lead to having an initial set of invalid values after the contract is initialized.

Consider adding the following checks:

- `dexAggregator != address(0)`
- `feeFactor <= _FEE_FACTOR_MAX`
- `liquidationRewardFactor <= _LIQUIDATION_REWARD_FACTOR_MAX`
- `LOAN_TERM <= _LOAN_TERM_MAX`
- `treasuryRate <= _TREASURY_RATE_MAX`

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] Unused parameter in OpenPositionParams struct

The OpenPositionParams struct contains a field called `marginPremiumRatio` that is not used in the implementation.

```solidity
06:     struct OpenPositionParams {
07:         uint256 tokenId;
08:         uint256 marginFrom;
09:         uint256 marginTo;
10:         uint256 amountSwap;
11:         uint128 liquidity;
12:         uint24 tokenFromPremiumPortionMin;
13:         uint24 tokenToPremiumPortionMin;
14:         uint8 marginPremiumRatio;
15:         bool zeroForOne;
16:         bytes data;
17:     }
```

Consider removing this field from the struct to avoid any confusion.

### <a name="NC-2"></a>[NC-2] Unnecessary multicall feature in ParticleInfoReader

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L21

The ParticleInfoReader contract inherits the Multicall mixin, but since this contract is mostly composed of read only functions to query the state of the Position Manager, it is not entirely clear the purpose of having a multicall feature in this type of contract.

Consider removing the Multicall feature in ParticleInfoReader.

### <a name="NC-3"></a>[NC-3] Uniswap fee tiers can be dynamic

The implementation of `getDeepPool()` loops through the default fee tiers in order to fetch the pool with the deepest liquidity.

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L102-L117

```solidity
102:     function getDeepPool(address token0, address token1) external view returns (address deepPool) {
103:         uint24[3] memory feeTiers = [uint24(500), uint24(3000), uint24(10000)];
104:         uint128 maxLiquidity = 0;
105: 
106:         for (uint256 i = 0; i < feeTiers.length; i++) {
107:             address poolAddress = Base.UNI_FACTORY.getPool(token0, token1, feeTiers[i]);
108:             if (poolAddress != address(0)) {
109:                 IUniswapV3Pool pool = IUniswapV3Pool(poolAddress);
110:                 uint128 liquidity = pool.liquidity();
111:                 if (liquidity > maxLiquidity) {
112:                     maxLiquidity = liquidity;
113:                     deepPool = poolAddress;
114:                 }
115:             }
116:         }
117:     }
```

Even though the implementation is using the default fee tiers (500, 3000 and 10000), it is important to note that fee tiers are dynamic and can be configured in Uniswap. See the implementation of [`enableFeeAmount()`](https://github.com/Uniswap/v3-core/blob/d8b1c635c275d2a9450bd6a78f3fa2484fef73eb/contracts/UniswapV3Factory.sol#L61).

### <a name="NC-4"></a>[NC-4] `owner` variable is shadowed in `getLiquidityPosition()`

The implementation of `getLiquidityPosition()` uses the `owner` variable name to refer to the owner of the liquidity position.

This naming is clashing with the `owner` variable coming from the Ownable2StepUpgradeable base contract. 

### <a name="NC-5"></a>[NC-5] Unused `GetLienCache` struct

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticleInfoReader.sol#L254

The `GetLienCache` struct defined as part of the ParticleInfoReader contract is not used in any part of the codebase and can be safely removed to avoid any potential confusion.

### <a name="NC-6"></a>[NC-6] Unused SwapPosition library in ParticlePositionManager

The ParticlePositionManager contract is importing the SwapPosition library with a `using` construct.

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L29

```solidity
29:     using SwapPosition for mapping(bytes32 => SwapPosition.Info);
```

This import is not used in the implementation of ParticlePositionManager and can be safely removed.

### <a name="NC-7"></a>[NC-7] Incorrect call to base contract initializer

The initializer of the ParticlePositionManager contract is calling the `__Ownable_init()` corresponding to the Ownable contract, even though the contract is actually inheriting from Ownable2StepUpgradeable.

https://github.com/code-423n4/2023-12-particle/blob/a3af40839b24aa13f5764d4f84933dbfa8bc8134/contracts/protocol/ParticlePositionManager.sol#L60-L68

```solidity
60:     function initialize(
61:         address dexAggregator,
62:         uint256 feeFactor,
63:         uint128 liquidationRewardFactor,
64:         uint256 loanTerm,
65:         uint256 treasuryRate
66:     ) external initializer {
67:         __UUPSUpgradeable_init();
68:         __Ownable_init();
```

Consider using the proper call to the `__Ownable2Step_init()` base initializer.

