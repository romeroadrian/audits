# Report


## Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Contract files should define a locked compiler version | 4 |
| [L-2](#L-2) | Royalties are paid assuming all NFTs in the batch are equally priced | - |
| [L-3](#L-3) | Potential loss of funds when paying royalties | - |
| [L-4](#L-4) | NFT address is not validated in Factory `create` | - |
| [L-5](#L-5) | Missing event for important parameter change | 3 |
| [L-6](#L-6) | Factory `setProtocolFeeRate` should validate that the fee rate is within an acceptable range | - |
| [L-7](#L-7) | Potential overflow while updating reserves values in PrivatePool contract | - |
| [L-8](#L-8) | Zero amount ERC20 token transfers may fail some implementations | - |
| [L-9](#L-9) | `price` function will revert for ERC20 token with high decimals | - |


### <a name="L-1"></a>[L-1] Contract files should define a locked compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

*Instances (4)*:
```solidity
File: src/EthRouter.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/Factory.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/PrivatePool.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/PrivatePoolMetadata.sol

2: pragma solidity ^0.8.19;

```

### <a name="L-2"></a>[L-2] Royalties are paid assuming all NFTs in the batch are equally priced

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L115
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182

In the EthRouter and PrivatePool contracts, when buying or selling NFTs, royalty fees are calculated as if all tokens were equally priced, since the sale price is calculated by just taking the amount and dividing it by the numbers of tokens.

```solidity
uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
```

A more robust solution would be to factor the token weights in the calculation so each NFT is priced according to the weight it has in the pool.

### <a name="L-3"></a>[L-3] Potential loss of funds when paying royalties

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L121
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L188

In order to pay royalties, the EthRouter contract queries a registry in order to fetch the recipient and the amount of the royalty fee. In both of these cases, the implementation checks that `royaltyFee` is greater than zero but doesn't verify that `royaltyRecipient` is not the empty address. If this is the case, then funds will be sent to the `address(0)`.

```solidity
(uint256 royaltyFee, address royaltyRecipient) =
    getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);

if (royaltyFee > 0) {
    // transfer the royalty fee to the royalty recipient
    royaltyRecipient.safeTransferETH(royaltyFee);
}
```

The recommendation is to also check that `royaltyFee != address(0)`.

```solidity
(uint256 royaltyFee, address royaltyRecipient) =
    getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);

if (royaltyFee > 0 && royaltyRecipient != address(0)) {
    // transfer the royalty fee to the royalty recipient
    royaltyRecipient.safeTransferETH(royaltyFee);
}
```

### <a name="L-4"></a>[L-4] NFT address is not validated in Factory `create`

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71

The `create` function should validate that `_nft != address(0)`.

### <a name="L-5"></a>[L-5] Missing event for important parameter change

Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

*Instances (3)*:

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141

### <a name="L-6"></a>[L-6] Factory `setProtocolFeeRate` should validate that the fee rate is within an acceptable range

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141-L143

The `setProtocolFeeRate` should validate that the `_protocolFeeRate` value is within an acceptable range for the protocol fee.

### <a name="L-7"></a>[L-7] Potential overflow while updating reserves values in PrivatePool contract

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230-L231
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323-L324

In both the `buy` and `sell` functions of the PrivatePool contracts, reserves values are updated by downcasting an `uint256` to a `uint128`, which may cause an overflow of the values.

```solidity
virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
virtualNftReserves -= uint128(weightSum);
```

The implementation should use a "safe" casting variant in order to prevent the overflow, such as solmate's `SafeCastLib`.

### <a name="L-8"></a>[L-8] Zero amount ERC20 token transfers may fail some implementations

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L423
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L502
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L527
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651

In the PrivatePool contract there are several occurrences of a potential ERC20 transfer of zero value. Some ERC20 token implementations revert the operation when transferring a zero amount, which will end up reverting the transaction.

In all of these cases, the implementation should check that the transferred value is greater than zero before calling the transfer function.

### <a name="L-9"></a>[L-9] `price` function will revert for ERC20 token with high decimals

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744

The `price` function present in the PrivatePool factory will revert if the base token is an ERC20 with more than 36 decimals, as the subtraction will cause an underflow.

```solidity
uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
```
