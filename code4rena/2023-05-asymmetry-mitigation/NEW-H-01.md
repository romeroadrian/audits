# [adriro-NEW-H-01] Protocol assumes a 1:1 peg of frxETH to ETH

Link to changeset: https://github.com/asymmetryfinance/smart-contracts/pull/262/files

## Impact

The `ethPerDerivative()` function in the SfrxEth now assumes a peg of frxETH to ETH, and reverts if the price difference (queried through the Curve pool) is more than 0.1%. Presumably, this decision was likely caused by the fact that frxETH doesn't have a Chainlink price feed, so the sponsor decided to continue using the Curve oracle but now assuming there is a peg between both tokens.

This assumption is very dangerous and could potentially cause a DoS in the derivative if frxETH de-pegs as the `ethPerDerivative()` function will always revert.

Similarly to the scenarios described in this issue https://github.com/code-423n4/2023-03-asymmetry-findings/issues/588 for stETH, it may be the case that a sell pressure on frxETH tanks the price down, or a buy pressure raises the prices above the expected limit (see charts here https://www.coingecko.com/en/coins/frax-ether/eth). Especially with just a margin of 0.1%, as this represents a difference of just 0.001 ETH per unit.

It could also be the case that the protocol behind frxETH fails or gets compromised. If this happens then it is highly likely the price will fall as people will start exiting their position. As the SfrxEth `withdraw()` functions depends on the `ethPerDerivative()` function to calculate slippage, this will effectively DoS users that want to exit their position using `unstake()`, and will also DoS protocol admins that need to call `rebalanceToWeights()` to remove the position on the protocol level.

## Recommendation

Remove the 1:1 peg assumption. Regarding the price manipulation, remember that the Curve oracle price represents a moving average of the price. Alternatively, look for another TWAP price feed or introduce additional checks to guard against price manipulation attacks (see https://code4rena.com/reports/2022-02-redacted-cartel#m-17-thecosomataeth-oracle-price-can-be-better-secured-freshness--tamper-resistance).
