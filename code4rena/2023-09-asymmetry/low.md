# Report

## Summary

### Low Issues

Total of **18 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-hardcoded-addresses-may-not-be-compatible-with-a-multi-chain-deployment) | Hardcoded addresses may not be compatible with a multi-chain deployment |
| [L-2](#l-2-use-ownable2step-instead-of-ownable-for-access-control) | Use Ownable2Step instead of Ownable for access control |
| [L-3](#l-3-missing-name-and-symbol-for-afeth-token) | Missing name and symbol for AfEth token |
| [L-4](#l-4-validate-ratio-argument-in-setratio) | Validate ratio argument in `setRatio()` |
| [L-5](#l-5-price-calculations-may-experience-precision-loss-due-to-division-before-multiplication) | Price calculations may experience precision loss due to division before multiplication |
| [L-6](#l-6-validate-withdrawals-in-afeth-have-been-already-executed) | Validate withdrawals in AfEth have been already executed |
| [L-7](#l-7-prevent-afeth-token-transfers-to-its-own-contract) | Prevent AfEth token transfers to its own contract |
| [L-8](#l-8-public-relock-function-can-be-used-to-grief-withdrawal-requests) | Public relock function can be used to grief withdrawal requests |
| [L-9](#l-9-missing-call-to-base-initializers) | Missing call to base initializers |
| [L-10](#l-10-missing-check-for-zero-address-value-in-constructor-or-setter) | Missing check for zero address value in constructor or setter |
| [L-11](#l-11-function-canwithdraw-in-votiumstrategy-doesnt-check-if-withdrawal-has-been-already-executed) | Function `canWithdraw()` in VotiumStrategy doesn't check if withdrawal has been already executed |
| [L-12](#l-12-votiumstrategy-allows-to-recover-erc20-tokens-but-not-native-eth) | VotiumStrategy allows to recover ERC20 tokens but not native ETH |
| [L-13](#l-13-missing-usage-of-safe-wrappers-to-handle-erc20-operations) | Missing usage of safe wrappers to handle ERC20 operations |
| [L-14](#l-14-zero-token-allowance-can-cause-denial-of-service-in-applyrewards) | Zero token allowance can cause denial of service in `applyRewards()` |
| [L-15](#l-15-low-level-calls-to-account-with-no-code-will-not-fail) | Low level calls to account with no code will not fail |
| [L-16](#l-16-protocol-fees-are-not-collected-when-rewards-are-not-routed-through-afeth) | Protocol fees are not collected when rewards are not routed through AfEth |
| [L-17](#l-17-protocol-doesnt-collect-fees-from-safeth) | Protocol doesn't collect fees from SafEth |
| [L-18](#l-18-potential-rounding-to-zero-issue-in-afeth-deposit-could-cause-loss-of-value) | Potential rounding to zero issue in AfEth deposit could cause loss of value |

### Non Critical Issues

Total of **4 issues**:

|ID|Issue|
|:--:|:---|
| [NC-1](#nc-1-remove-debug-symbols) | Remove debug symbols |
| [NC-2](#nc-2-missing-event-for-important-parameter-change) | Missing event for important parameter change |
| [NC-3](#nc-3-unused-constants) | Unused constants |
| [NC-4](#nc-4-use-constants-for-literal-or-magic-values) | Use constants for literal or magic values |

### Informational Issues

Total of **1 issues**:

|ID|Issue|
|:--:|:---|
| [INFO-1](#info-1-consider-using-the-erc4626-standard) | Consider using the ERC4626 standard |

## Low Issues

### <a name="L-1"></a>[L-1] Hardcoded addresses may not be compatible with a multi-chain deployment

The codebase is full of hardcoded addresses that refer to other parts of the protocol (SafEth, for example) or third-party protocols (e.g. Votium, Convex, Curve Pools).

Assumptions of the presence of third-party protocol, their deployment addresses and/or their arguments will prevent or complicate deployments in layer 2 chains. This is stated as a possibility in the documentation: 

> This will only be deployed to Ethereum Mainnet, with the chance of being deployed on L2's on a future date

### <a name="L-2"></a>[L-2] Use Ownable2Step instead of Ownable for access control

Use the [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) variant of the Ownable contract to better safeguard against accidental transfers of access control.

*Instances (2)*:

- [AfEth](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L10)
- [VotiumStrategyCore](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L24)

### <a name="L-3"></a>[L-3] Missing name and symbol for AfEth token

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L72

The AfEth contract inherits from ERC20Upgradeable but doesn't call its base initializer, which is in charge of setting the name and symbol for the ERC20 token.

### <a name="L-4"></a>[L-4] Validate ratio argument in `setRatio()`

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L90

The `ratio` configuration parameter in AfEth measures the portion of deposited value that goes into the SafEth contract. This value is intended to range between `0` (0%) and `1e18` (100%).

The `setRatio()` function should validate that the new value is within bounds, i.e. `require(_newRatio <= 1e18)`.

### <a name="L-5"></a>[L-5] Price calculations may experience precision loss due to division before multiplication

There are several places across the codebase in which intermediate results that depend on a division are then used as part of calculations that involve a multiple.

For example, in `requestWithdraw()` the calculation of `votiumWithdrawAmount` depends on the intermediate result of `withdrawRatio`. The expression can be simplified as:

```solidity
uint256 votiumWithdrawAmount = (_amount * votiumBalance) / (totalSupply() - afEthBalance);
```

And similarly, `safEthWithdrawAmount`:

```solidity
uint256 safEthWithdrawAmount = (_amount * safEthBalance) / (totalSupply() - afEthBalance);
```

### <a name="L-6"></a>[L-6] Validate withdrawals in AfEth have been already executed

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L243

Withdrawals in AfEth are first enqueued when requested, and executed later when the vAfEth tokens are ready to be withdrawn. The `withdraw()` function in AfEth validates that the withdrawal is ready (by using `canWithdraw()`) but doesn't validate if the withdrawal has been already executed.

This isn't currently exploitable since the withdrawal in AfEth also depends on the withdrawal in VotiumStrategy, which correctly checks if the withdrawal associated to the `vEthWithdrawId` has been already claimed. Consider adding a similar check to `AfEth::withdraw()`.

### <a name="L-7"></a>[L-7] Prevent AfEth token transfers to its own contract

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L183

When a user requests a withdrawal in AfEth, their tokens are transferred to the contract and "locked" until the withdrawal is made effective, at which point the tokens are burned.

Given this mechanism, AfEth tokens held by the same contract are considered as locked tokens but anyone can technically transfer tokens to the contract by calling `transfer()` or `transferFrom()`.

Consider adding a check to the base ERC20 functionality to prevent AfEth tokens from being sent to the contract by external actors.

### <a name="L-8"></a>[L-8] Public relock function can be used to grief withdrawal requests

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L135

A malicious user can call `relock()` to front-run a transaction to `requestWithdraw()`. 

Relocking will take any available CVX in the contract and expired locks in CvxLocker and relock them. Griefed users will need to wait more, as any of the available balance that could have been used for the withdrawal has been relocked as part of the front-running.

### <a name="L-9"></a>[L-9] Missing call to base initializers

Upgradeable contracts that inherit from other base contracts should call the corresponding base initializers during initialization.

*Instances (4)*:

- [AfEth::OwnableUpgradeable](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L72)
- [AfEth::ERC20Upgradeable](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L72)
- [AbstractStrategy::ReentrancyGuardUpgradeable](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/AbstractStrategy.sol#L10)
- [VotiumStrategyCore::OwnableUpgradeable](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L100)

### <a name="L-10"></a>[L-10] Missing check for zero address value in constructor or setter

Address parameters should be validated to guard against the default value `address(0)`.

*Instances (6)*:

- `vEthAddress` in [`AfEth::setStrategyAddress()`](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L81)
- `feeAddress` in [`AfEth::setFeeAddress()`](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L98)
- `rewarder` in [`VotiumStrategyCore::initialize()`](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L112)
- `manager` in [`VotiumStrategyCore::initialize()`](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L113)
- `chainlinkCvxEthFeed` in [`VotiumStrategyCore::setChainlinkCvxEthFeed()`](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L76)
- `rewarder` in [`VotiumStrategyCore::setRewarder()`](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L125)

### <a name="L-11"></a>[L-11] Function `canWithdraw()` in VotiumStrategy doesn't check if withdrawal has been already executed

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategy.sol#L155

The implementation of `canWithdraw()` in the VotiumStrategy contract just checks if the required epoch has been reached, but doesn't validate if the request has been already fulfilled (`WithdrawRequestInfo.withdrawn`).

Consider also checking for the `withdrawn` condition as part of the implementation of `canWithdraw()`.

```solidity
function canWithdraw(
    uint256 _withdrawId
) external view virtual override returns (bool) {
    uint256 currentEpoch = ILockedCvx(VLCVX_ADDRESS).findEpochId(
        block.timestamp
    );
    return
        withdrawIdToWithdrawRequestInfo[_withdrawId].epoch <= currentEpoch && !withdrawIdToWithdrawRequestInfo[_withdrawId].withdrawn;
}
```

### <a name="L-12"></a>[L-12] VotiumStrategy allows to recover ERC20 tokens but not native ETH

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L215

The implementation of `withdrawStuckTokens()` allows the owner of the protocol to recover any ERC20 token, but fails to consider native ETH transfers.

### <a name="L-13"></a>[L-13] Missing usage of safe wrappers to handle ERC20 operations

- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L216
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L282
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L287

ERC20 operations on arbitrary tokens should be [safely wrapped](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20) to account for incompatible implementations.

### <a name="L-14"></a>[L-14] Zero token allowance can cause denial of service in `applyRewards()`

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L282-L285

```solidity
IERC20(_swapsData[i].sellToken).approve(
    address(_swapsData[i].spender),
    0
);
```

During `applyRewards()`, token allowances are first reset to zero before being increased to infinity. This could cause issues with some ERC20 implementations that revert on zero value approvals, such as [BNB](https://etherscan.io/token/0xB8c77482e45F1F44dE1745F52C74426C631bDD52#code#L92).

### <a name="L-15"></a>[L-15] Low level calls to account with no code will not fail

- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L292

Low level calls (i.e. `address.call(...)`) to account with no code will silently succeed without reverting or throwing any error. Quoting the reference for the CALL opcode in evm.codes:

> Creates a new sub context and execute the code of the given account, then resumes the current one. Note that an account with no code will return success as true.

### <a name="L-16"></a>[L-16] Protocol fees are not collected when rewards are not routed through AfEth

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L302-L304

Protocol fees are collected in `AfEth::depositRewards()`, just before rewards are being compounded in the protocol.

The reward flow is initiated in `VotiumStrategyCore::applyRewards()`, and will deposit rewards in AfEth only if the manager address is defined:

```solidity
302:         if (address(manager) != address(0))
303:             IAfEth(manager).depositRewards{value: ethReceived}(ethReceived);
304:         else depositRewards(ethReceived);
```

It is important to note that if rewards aren't channeled through AfEth, the protocol will not receive any fees.

### <a name="L-17"></a>[L-17] Protocol doesn't collect fees from SafEth

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L272

Protocol fees are only collected as part of the deposited rewards coming from the VotiumStrategy contract. Votium and Convex rewards are claimed and deposited back in the protocol as an explicit compound action.

On the other hand, SafEth accrues value by the passive appreciation of the underlying LSD tokens backing the protocol. There is no explicit process for claiming or compounding this increase in SafEth token value. The protocol isn't collecting fees from this side of the split.

It is not clear if this is by design or a potential oversight in the implementation.

### <a name="L-18"></a>[L-18] Potential rounding to zero issue in AfEth deposit could cause loss of value

Deposits in the AfEth contract are split based on a configured `ratio`. One portion of the split goes to SafEth, while the other is deposited in the Votium strategy.

https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L148-L169

```solidity
148:     function deposit(uint256 _minout) external payable virtual {
149:         if (pauseDeposit) revert Paused();
150:         uint256 amount = msg.value;
151:         uint256 priceBeforeDeposit = price();
152:         uint256 totalValue;
153: 
154:         AbstractStrategy vStrategy = AbstractStrategy(vEthAddress);
155: 
156:         uint256 sValue = (amount * ratio) / 1e18;
157:         uint256 sMinted = sValue > 0
158:             ? ISafEth(SAF_ETH_ADDRESS).stake{value: sValue}(0)
159:             : 0;
160:         uint256 vValue = (amount * (1e18 - ratio)) / 1e18;
161:         uint256 vMinted = vValue > 0 ? vStrategy.deposit{value: vValue}() : 0;
162:         totalValue +=
163:             (sMinted * ISafEth(SAF_ETH_ADDRESS).approxPrice(true)) +
164:             (vMinted * vStrategy.price());
165:         if (totalValue == 0) revert FailedToDeposit();
166:         uint256 amountToMint = totalValue / priceBeforeDeposit;
167:         if (amountToMint < _minout) revert BelowMinOut();
168:         _mint(msg.sender, amountToMint);
169:     }
```

The amounts that go into each are calculated in lines 156 and 160. Both `sValue` and `vValue` calculations could be rounded down to zero if the numerator is lower than the denominator.

Even with a low `ratio` value, the amounts lost are negligible, hence the low severity.

For example, given a small `ratio` of 1% (i.e. `1e16`) we have:

```
amount * ratio < 1e18
amount < 1e18 / ratio
amount < 1e18 / 1e16
amount < 100
```

Amount should be lower than 100 wei in order to be rounded down to zero.

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] Remove debug symbols

- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L18

Remove any code related to debug functionality.

### <a name="NC-2"></a>[NC-2] Missing event for important parameter change

Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

*Instances (8)*:

- [AfEth::setStrategyAddress()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L81)
- [AfEth::setRatio()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L90)
- [AfEth::setFeeAddress()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L98)
- [AfEth::setProtocolFee()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L106)
- [AfEth::setPauseDeposit()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L116)
- [AfEth::setPauseWithdraw()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L124)
- [VotiumStrategyCore::setChainlinkCvxEthFeed()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L76)
- [VotiumStrategyCore::setRewarder()](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L125)

### <a name="NC-3"></a>[NC-3] Unused constants

Unreferenced private constants in contracts can be removed.

*Instances (2)*:

- [AfEth::CVX_ADDRESS](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L50)
- [AfEth::VLCVX_ADDRESS](https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/AfEth.sol#L52)

### <a name="NC-4"></a>[NC-4] Use constants for literal or magic values

Consider defining constants for literal or magic values as it improves readability and prevents duplication of config values.

*Instances (9)*:

- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L105
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L106
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L107
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L117
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L180
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L230
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L253
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L314
- https://github.com/code-423n4/2023-09-asymmetry/blob/main/contracts/strategies/votium/VotiumStrategyCore.sol#L323

## Informational Issues

### <a name="INFO-1"></a>[INFO-1] Consider using the ERC4626 standard

Both AfEth and VotiumStrategy behave like vaults. They take deposits, mint ERC20 tokens in representation of this deposit, and have a withdraw function to recover the assets back.

This is exactly the use case of the [ERC4626 standard](https://eips.ethereum.org/EIPS/eip-4626). Consider using this standard to improve composability.

