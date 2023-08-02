# [adriro-NEW-M-01] Ability to disable derivatives currently in use generates new potential issues

Link to changeset: https://github.com/asymmetryfinance/smart-contracts/pull/264/files

## Impact

Since originally derivatives could not be removed from the system, the protocol team decided to implement an enable/disable feature to address potential problems that arise in individual derivatives.

However, disabling a derivative while in use may negatively impact other users and block certain operations that will have consequences in the protocol at different levels causing new potential issues.

For example, if the derivative needs to be disabled because of an issue in the `deposit()` function, which may affect the SafEth `stake()` operation, this will affect the `unstake()` too:

https://github.com/asymmetryfinance/smart-contracts/pull/264/files#diff-badfabc2bc0d1b9ef5dbef737cd03dc2f570f6fd2074aea9514da9db2fff6e4eR114-R126

```solidity
function unstake(
    uint256 _safEthAmount,
    uint256 _minOut
) external nonReentrant {
    require(pauseUnstaking == false, "unstaking is paused");
    require(_safEthAmount > 0, "amount too low");
    require(_safEthAmount <= balanceOf(msg.sender), "insufficient balance");
    uint256 safEthTotalSupply = totalSupply();
    uint256 ethAmountBefore = address(this).balance;

    for (uint256 i = 0; i < derivativeCount; i++) {
        if (!settings[i].enabled) continue;
        
    ....
```

If the derivative is currently in use, protocol users that have funds deposited in the derivative that call `unstake()` will suffer a loss of funds, as the derivative will simply be skipped. 

Similarly, disabled derivatives are not taken into account while rebalancing the protocol. If the derivative needs to disabled due to any reason, the `rebalanceToWeights()` function which simply ignore it:

https://github.com/asymmetryfinance/smart-contracts/pull/264/files#diff-badfabc2bc0d1b9ef5dbef737cd03dc2f570f6fd2074aea9514da9db2fff6e4eR159-R163

```solidity
function rebalanceToWeights() external onlyOwner {
    for (uint i = 0; i < derivativeCount; i++) {
        if (derivatives[i].balance() > 0)
        if (settings[i].enabled && derivatives[i].balance() > 0)
            derivatives[i].withdraw(derivatives[i].balance());
    }
```

## Recommendation

Implement a more granular disable feature, that provides separate control of the 3 main actions of SafEth.sol (stake, unstake and weight rebalance). This would allow protocol admins to have better control to mitigate potential issues or emergencies that depend on the underlying protocols of each derivative.
