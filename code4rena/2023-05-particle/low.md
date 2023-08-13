# Report

- Non Critical Issues (3)
- Low Issues (10)


## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Use named parameters for mapping type declarations | 3 |
| [NC-2](#NC-2) | Unneeded explicit return | 1 |
| [NC-3](#NC-3) | Use bool type for togglable parameter | 1 |

### <a name="NC-1"></a>[NC-1] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18

*Instances (3)*:
```solidity
File: contracts/protocol/ParticleExchange.sol

24:     mapping(uint256 => bytes32) public liens;

25:     mapping(address => uint256) public interestAccrued;

26:     mapping(address => bool) public registeredMarketplaces;

```

### <a name="NC-2"></a>[NC-2] Unneeded explicit return

The explicit return can be omitted as the function is using named return variables.

*Instances (1)*:

- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/libraries/math/MathUtils.sol#L26

### <a name="NC-3"></a>[NC-3] Use bool type for togglable parameter

*Instances (1)*:

- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L343


## Low Issues

| |Issue|
|-|:-|
| [L-1](#L-1) | Contract files should define a locked compiler version |
| [L-2](#L-2) | `_withdrawAccountInterest()` can be front-runned to increase the treasury rate |
| [L-3](#L-3) | Relax amount check strictness while operating with marketplaces |
| [L-4](#L-4) | The `onERC721Received` callback can be used to create fake liens |
| [L-5](#L-5) | Accidental loss of NFTs due to misuse of push mechanism |
| [L-6](#L-6) | `auctionBuyNft()` should use current auction price instead of `amount` parameter |
| [L-7](#L-7) | Use Ownable2Step instead of Ownable for access control |
| [L-8](#L-8) | Provide safer limits for treasury rate |
| [L-9](#L-9) | Griefer can DoS lender NFT withdrawals |
| [L-10](#L-10) | Marketplace calls are too permissive |

### <a name="L-1"></a>[L-1] Contract files should define a locked compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

*Instances (1)*:
```solidity
File: contracts/protocol/ParticleExchange.sol

2: pragma solidity ^0.8.17;

```

### <a name="L-2"></a>[L-2] `_withdrawAccountInterest()` can be front-runned to increase the treasury rate

https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L238

A call to `_withdrawAccountInterest()` can be front-runned to increase the treasury rate and diminish (or nullify) the lender's portion of the loan interests.

```solidity
function _withdrawAccountInterest(address payable lender) internal {
    uint256 interest = interestAccrued[lender];
    if (interest == 0) return;

    interestAccrued[lender] = 0;

    if (_treasuryRate > 0) {
        uint256 treasuryInterest = MathUtils.calculateTreasuryProportion(interest, _treasuryRate);
        _treasury += treasuryInterest;
        interest -= treasuryInterest;
    }

    lender.transfer(interest);

    emit WithdrawAccountInterest(lender, interest);
}
```

The `treasuryInterest` variable is calculated as a proportion of the `interest` amount which is then subtracted to calculate the lender's share.

### <a name="L-3"></a>[L-3] Relax amount check strictness while operating with marketplaces

- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L331
- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L428

In both `sellNftToMarket()` and `buyNftFromMarket()` the given amount is validated using a strict equality check.

These conditions can be relaxed to account for extra fees, rounding or potential minimal differences when the transaction gets executed. For example, the sell operation can check if the difference in balance is greater or equal than the amount (i.e. it received at least `amount`) and the buy operation can check the the balance differences is lower or equal than the amount (i.e. it sent at most `amount`).

### <a name="L-4"></a>[L-4] The `onERC721Received` callback can be used to create fake liens

https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L74

Since the `onERC721Received` callback is a public function it can be called by anyone to create fake liens. The `from` parameter is user supplied to the function, which means that anyone can create fake liens on behalf of an arbitrary lender.

### <a name="L-5"></a>[L-5] Accidental loss of NFTs due to misuse of push mechanism

https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L74

An accidental loss of NFTs can happen if the sender doesn't use `safeTransferFrom()` or submits an incorrect payload in `safeTransferFrom()`.

### <a name="L-6"></a>[L-6] `auctionBuyNft()` should use current auction price instead of `amount` parameter

https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L688

The `amount` parameter in the `auctionBuyNft()` function should be used as a slippage check to ensure the caller gets at least an `amount` value from the action, but the effective value the offerer receives should be `currentAuctionPrice` (as this represents the maximum incentive the offerer gets while executing the action).

### <a name="L-7"></a>[L-7] Use Ownable2Step instead of Ownable for access control

- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L14

Use the [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) variant of the Ownable contract to better safeguard against accidental transfers of access control.

### <a name="L-8"></a>[L-8] Provide safer limits for treasury rate

https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L800

The current implementation of `setTreasuryRate()` only limits the rate parameter to `_BASIS_POINTS`, which if maxed represents the 100% of the lender's earnings.

### <a name="L-9"></a>[L-9] Griefer can DoS lender NFT withdrawals

https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L172

A malicious actor can front-runs calls to `withdrawNftWithInterest()` to prevent the lender from claiming the NFT and deleting the lien, by taking a loan of the lien and withdrawing the NFT, causing the transaction to `withdrawNftWithInterest()` to fail. The malicious borrower can then immediately repay the loan with zero or minimum interests.

### <a name="L-10"></a>[L-10] Marketplace calls are too permissive

- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L316
- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L414
- https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L420

The `sellNftToMarket()` and `buyNftFromMarket()` functions present in the ParticleExchange contract are used to execute a sell or a buy operation in a registered marketplace.

Even though marketplaces are whitelisted, both of these functions take an arbitrary `tradeData` payload that is supplied by the caller. This argument is then used as the calldata to the marketplace functions.

An attacker could use this to essentially call any function in the registered marketplace.

The recommendation here is to also whitelist function selectors and validate the right calls are being made. For example, this could be implemented as a `mapping(address => mapping(bytes4 => bool))` that indicates whether a particular function signature is enabled for the corresponding marketplace.
