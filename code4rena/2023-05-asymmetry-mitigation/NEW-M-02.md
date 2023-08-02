# [adriro-NEW-M-02] Introduced change in `stake()` may affect calculation of minted shares

Link to changeset: https://github.com/asymmetryfinance/smart-contracts/pull/209/files

## Impact

Pull request #209 (link above) introduces a change in how the `preDepositPrice` is calculated. Now, if the `underlyingValue` is zero, the `preDepositPrice` will be calculated as 1e18, similar to the case when `totalSupply == 0`:

https://github.com/asymmetryfinance/smart-contracts/pull/209/files#diff-badfabc2bc0d1b9ef5dbef737cd03dc2f570f6fd2074aea9514da9db2fff6e4eR78

```solidity
uint256 preDepositPrice; // Price of safETH in regards to ETH
if (totalSupply == 0 || underlyingValue == 0)
    preDepositPrice = 10 ** 18; // initializes with a price of 1
else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```

Before this change, if `underlyingValue == 0`, then `preDepositPrice == 0` and eventually the function will revert as the calculation of `mintAmount` divides by `preDepositPrice`. With this new change, it is possible to have a situation where `totalSupply != 0` but `underlyingValue == 0`, which will cause `preDepositPrice == 1e18`.

This should be considered an edge case, however it should be noted that with the introduction of disabled derivatives, it might be the case that derivatives are disabled, causing the `underlyingValue` variable to be zero. Given this scenario, `mintAmount` will be equal to `totalStakeValueEth`, without any restrictions to `totalSupply` (meaning there may be existing shareholders of SafEth):

```solidity
uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
                                ||
                                \/
uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / 1e18;
                                ||
                                \/
uint256 mintAmount = totalStakeValueEth;
```

## Recommendation

Revert the highlighted change.
