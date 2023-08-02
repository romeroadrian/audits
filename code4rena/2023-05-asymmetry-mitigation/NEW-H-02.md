# [adriro-NEW-H-02] Chainlink price feed responses are not validated

Link to changesets:

- https://github.com/asymmetryfinance/smart-contracts/pull/209/files
- https://github.com/asymmetryfinance/smart-contracts/pull/242/files

## Impact

The protocol team introduced Chainlink price feeds for the Reth and WstEth derivatives in order to mitigate price manipulation attacks. 

These changes introduce new issues, as the Chainlink responses are not validated at all. This is the implementation for Reth:

https://github.com/asymmetryfinance/smart-contracts/pull/209/files#diff-6abc8f2e4ad1647a12784e9fbf18e9c5f86c05668e3e89e2a51ab569992b214fR146-L216

```solidity
function ethPerDerivative() public view returns (uint256) {
    (, int256 chainLinkRethEthPrice, , , ) = chainLinkRethEthFeed
        .latestRoundData();
    return uint256(chainLinkRethEthPrice);
}
```

In the case of the WstEth derivative, additionally, the implementation even sets the price to zero if it is negative:

https://github.com/asymmetryfinance/smart-contracts/pull/242/files#diff-ac281bf63004ef9a825c084018c54f10b03233cd4f286398f5d5e993612308b5R90-R98

```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    uint256 stPerWst = IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    (, int256 chainLinkStEthEthPrice, , , ) = chainLinkStEthEthFeed
        .latestRoundData();
    if (chainLinkStEthEthPrice < 0) chainLinkStEthEthPrice = 0;
    uint256 ethPerWstEth = (stPerWst * uint256(chainLinkStEthEthPrice)) /
        10 ** 18;
    return ethPerWstEth;
}
```

Chainlink responses must be validated. The price may be invalid, the current round may not be finished, the response may be stale, among other issues. These outputs represent critical pieces in the protocol, as `ethPerDerivative()` is used in the `stake()` function to calculate the deposited amount, and also used to calculate slippage in the implementation of the derivative.

As a reference, these reports mention similar cases of missing validation in the Chainlink response:

- https://github.com/sherlock-audit/2022-09-knox-judging/issues/137
- https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/94
- https://solodit.xyz/issues/9795

The following report also mentions an important detail related to the freshness of the feed for stETH/ETH, as the heartbeat for this oracle is 24 hours, see https://github.com/sherlock-audit/2023-03-olympus-judging/issues/2 . Note that also the rETH/ETH price feed has a 2% deviation threshold, see https://data.chain.link/ethereum/mainnet/crypto-eth/reth-eth

- https://data.chain.link/ethereum/mainnet/crypto-eth/steth-eth
- https://data.chain.link/ethereum/mainnet/crypto-eth/reth-eth

## Recommendation

Validate the Chainlink response arguments. See the following article for a good set of recommendations https://0xmacro.com/blog/how-to-consume-chainlink-price-feeds-safely/
