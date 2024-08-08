<table>
    <tr>
        <th colspan="2">
            <img src="https://raw.githubusercontent.com/romeroadrian/audits/main/images/banner.png" width="800"/>
        </th>
    </tr>
    <tr>
        <td align="center"><img src="https://asymmetry.finance/favicon.ico" width="200" height="200" align="center"/></td>
        <td>
            <h1>Asymmetry Finance Report</h1>
            <h2>Liquid Locker</h2>
            <p>Conducted by: adriro (<a href="https://twitter.com/adrianromero">@adrianromero</a>)</p>
            <p>Date: Jul 24 to 30, 2024</p>
        </td>
    </tr>
</table>

# Asymmetry Finance Security Review

## Disclaimer

_The conducted security review represents an evaluation based on the information and code provided by the client. The author, employing a combination of automated tools and manual expertise, has endeavored to identify potential vulnerabilities. It is crucial to understand that this review does not ensure the absolute absence of vulnerabilities or errors within the smart contracts._

_Despite exercising due diligence, this assessment may not uncover all potential issues or undiscovered vulnerabilities present in the code. Findings and recommendations are based solely on the information available at the time of the review._

_This report is not to be considered as an endorsement or certification of the smart contract's absolute security. Authors cannot assume responsibility for any losses or damages that may arise from the utilization of the smart contracts._

_While this assessment aims to identify vulnerabilities, it cannot guarantee absolute security or eliminate all potential risks associated with smart contract usage._

## About afLiquidLocker

Asymmetry Finance Liquid Locker is a set of smart contracts designed to tokenize locked voting positions of governance assets from various protocols.

In this initial implementation, afSTG serves as a liquid representation of Stargate's governance token (STG). Protocol-owned STG is permanently locked into veSTG. Fees earned from the locked position are distributed to afSTG stakers, who can choose between manually farming rewards (USDC) or compounding them using an ERC4626 vault.

## About adriro

adriro is an independent security researcher currently focused on security reviews of smart contracts. He is a top warden at [code4rena](https://code4rena.com/) and serves as a resident auditor at [yAudit](https://yaudit.dev/).

You can follow him on X at [@adrianromero](https://x.com/adrianromero) or browse his [portfolio](https://github.com/romeroadrian/audits).

## Scope

The scope for the current review targets the [afLiquidLocker codebase](https://github.com/asymmetryfinance/afLiquidLocker) at revision [c82ff88ffe0eb214372465f65e8dd9c50a0426e6](https://github.com/asymmetryfinance/afLiquidLocker/commit/c82ff88ffe0eb214372465f65e8dd9c50a0426e6) and is limited to the following files:

```
src
├── Compounder.sol
├── LiquidLocker.sol
├── Locker.sol
├── interfaces
│   ├── ICurveCryptoswap.sol
│   ├── ICurveStableswapNG.sol
│   ├── ILiquidLocker.sol
│   ├── ILocker.sol
│   ├── IRewardDistributor.sol
│   ├── ISwapper.sol
│   └── IYearnBoostedStaker.sol
├── proxies
│   └── StargateProxy.sol
└── utils
    ├── Zap.sol
    └── swappers
        └── StargateSwapper.sol
```

## Summary

| Identifier | Title | Severity | Status |
| ---------- | ----- | ---------| ------ |
| [C-1](#c-1-compounder-needs-to-unstake-tokens-during-withdrawals) | Compounder needs to unstake tokens during withdrawals | Critical | Fixed |
| [H-1](#h-1-unclear-reward-token-flow) | Unclear reward token flow | High | Fixed |
| [M-1](#m-1-missing-slippage-control-when-output-token-is-afstg) | Missing slippage control when output token is afSTG | Medium | Fixed |
| [L-1](#l-1-missing-token-approval-when-changing-the-reward-token-in-compounder) | Missing token approval when changing the reward token in Compounder | Low | Fixed |
| [L-2](#l-2-sweep-recipient-can-be-empty-when-locker-contract-is-created) | Sweep recipient can be empty when Locker contract is created | Low | Fixed |
| [I-1](#i-1-claiming-stargates-fees-is-permissionless) | Claiming Stargate's fees is permissionless | Informational | Ack |
| [I-2](#i-2-confusing-logic-in-keeper-trigger) | Confusing logic in keeper trigger | Informational | Fixed |
| [I-3](#i-3-consider-adding-a-slippage-parameter-in-the-harvest-trigger) | Consider adding a slippage parameter in the harvest trigger | Informational | Fixed |
| [I-4](#i-4-missing-events-in-compounder-setters) | Missing events in Compounder setters | Informational | Ack |
| [I-5](#i-5-consider-delegating-stargate-voting-power-from-locker-contract) | Consider delegating Stargate voting power from Locker contract | Informational | Ack |
| [G-1](#g-1-gas-improvements-in-zap-contract) | Gas improvements in Zap contract | Gas | Partial |
| [G-2](#g-2-cache-storage-variable-in-harvest) | Cache storage variable in `harvest()` | Gas | Fixed |
| [G-3](#g-3-duplicate-mint-event-in-liquidlocker) | Duplicate mint event in LiquidLocker | Gas | Fixed |

## Critical Findings

### <a name="C-1"></a>[C-1] Compounder needs to unstake tokens during withdrawals

#### Summary

afSTG held by the Compounder is staked in the YearnBoostedStaker contract. To process withdrawals, it first needs to unstake the required tokens.

#### Details

The Compounder contract periodically stakes deposited afSTG tokens in the YearnBoostedStaker contract. To process withdrawals, the implementation will eventually need to unstake these assets first.

```solidity
213:     function _withdraw(
214:         address _caller,
215:         address _receiver,
216:         address _owner,
217:         uint256 _assets,
218:         uint256 _shares
219:     ) internal override {
220:         if (_caller != _owner) _spendAllowance(_owner, _caller, _shares);
221: 
222:         _totalAssets -= _assets;
223:         _burn(_owner, _shares);
224: 
225:         IERC20(asset()).safeTransfer(_receiver, _assets);
226: 
227:         emit Withdraw(_caller, _receiver, _owner, _assets, _shares);
228:     }
```

#### Impact

afSTG held by the Compounder cannot be unstaked, preventing withdrawals.

#### Recommendation

The Compounder should unstake the missing assets from the YearnBoostedStaker.

```diff
    function _withdraw(
        address _caller,
        address _receiver,
        address _owner,
        uint256 _assets,
        uint256 _shares
    ) internal override {
        if (_caller != _owner) _spendAllowance(_owner, _caller, _shares);

        _totalAssets -= _assets;
        _burn(_owner, _shares);
        
+       uint256 _balance = IERC20(asset()).balanceOf(address(this));
+       
+       if (_assets > _balance) {
+           unchecked {
+               uint256 _missing = _assets - _balance;
+               ybs.unstake(_missing, address(this));
+           }
+       }

        IERC20(asset()).safeTransfer(_receiver, _assets);

        emit Withdraw(_caller, _receiver, _owner, _assets, _shares);
    }
```

## High Findings

### <a name="H-1"></a>[H-1] Unclear reward token flow

#### Summary

The deployment script indicates that reward tokens are swept directly to the token distributor, which is incompatible with the distribution mechanism implemented by the contract.

#### Details

Earned fees from STG locks are claimed from the proxy and sent to the locker contract. 

```solidity
72:     function claim(address _asset) external onlyApprovedCaller {
73:         locker.execute(FEE_DISTRIBUTOR, 0, abi.encodeWithSignature("claimToken(address,address)", address(locker), _asset));
74:     }
```

From there, these are swept to the configured `sweepRecipient`.

```solidity
61:     function sweep(address _asset) external {
62:         if (_asset == asset) revert Unauthorized();
63: 
64:         if (_asset == address(0)) {
65:             payable(owner).sendValue(address(this).balance);
66:         } else {
67:             IERC20(_asset).safeTransfer(sweepRecipient, IERC20(_asset).balanceOf(address(this)));
68:         }
69:     }
```

According to the [deployment script](https://github.com/asymmetryfinance/afLiquidLocker/blob/c82ff88ffe0eb214372465f65e8dd9c50a0426e6/script/Deploy.s.sol#L111), the sweep recipient is the reward distributor:

```solidity
111:         locker.setSweepRecipient(address(rewardDistributor));
```

However, sending tokens directly to the contract is not how the distributor works, which expects an explicit call to the `depositReward()` function in order to pull the tokens and assign the distribution for the current week.

```solidity
54:     function depositReward(uint _amount) external {
55:         if (_amount > 0) {
56:             uint week = getWeek();
57:             weeklyRewardAmount[week] += _amount;
58:             rewardToken.safeTransferFrom(msg.sender, address(this), _amount);
59:             emit RewardDeposited(week, msg.sender, _amount);
60:         }
61:     }
```

Additionally, note that Stargate's [FeeDistributor](https://etherscan.io/address/0xAF667811A7eDcD5B0066CD4cA0da51637DB76D09#code) contract can support more than one token, while the SingleTokenRewardDistributor contract used in the protocol only deals with a single reward token.

#### Impact

Sweeping rewards directly to the distributor contract is incompatible with the distribution mechanism and could lead to loss of funds if used incorrectly.

#### Recommendation

The locker contract can act as the direct depositor of the distribution contract. If the locker needs to remain agnostic to how rewards are distributed, then create an additional contract that acts as the depositor and configure it as the sweep recipient in the locker. This also brings flexibility in deciding how and when reward tokens are deposited in the distributor.

## Medium Findings

### <a name="M-1"></a>[M-1] Missing slippage control when output token is afSTG

#### Summary

The Zap contract fails to check the minimum output amount for the afSTG case.

#### Details

In the [`zap()`](https://github.com/asymmetryfinance/afLiquidLocker/blob/c82ff88ffe0eb214372465f65e8dd9c50a0426e6/src/utils/Zap.sol#L83) function, the given `_minOut` amount is compared with the actual output for all cases except when the output token is afSTG.

```solidity
118:         if (_outputToken == YBS) {
119:             _amount = IYearnBoostedStaker(YBS).stakeFor(_receiver, _amount);
120:             if (_amount + 1 < _minOut) revert NotEnoughOutputAmount();
121:             return _amount;
122:         } else if (_outputToken == A_AF_TOKEN) {
123:             _amount = IERC4626(A_AF_TOKEN).deposit(_amount, _receiver);
124:             if (_amount < _minOut) revert NotEnoughOutputAmount();
125:             return _amount;
126:         } else if (_outputToken == LP_TOKEN) {
127:             uint256[] memory _amounts = new uint256[](2);
128:             _amounts[uint256(uint128(poolAFTokenIdx))] = _amount;
129:             _amount = curvePool.add_liquidity(_amounts, 0, _receiver);
130:             if (_amount < _minOut) revert NotEnoughOutputAmount();
131:             return _amount;
132:         } else if (_outputToken == TOKEN) {
133:             _amount = curvePool.exchange(poolAFTokenIdx, poolTokenIdx, _amount, 0, _receiver);
134:             if (_amount < _minOut) revert NotEnoughOutputAmount();
135:             return _amount;
136:         }
137: 
138:         if (_outputToken != AF_TOKEN) revert InvalidState();
139:         IERC20(AF_TOKEN).safeTransfer(_receiver, _amount);
140:         return _amount;
```

#### Impact

Slippage control is not applied when the user selects afSTG as the output option.

#### Recommendation

Check `_amount >= _minOut` when `_outputToken == AF_TOKEN`.

```diff
    if (_outputToken != AF_TOKEN) revert InvalidState();
+   if (_amount < _minOut) revert NotEnoughOutputAmount();
    IERC20(AF_TOKEN).safeTransfer(_receiver, _amount);
    return _amount;
```

## Low Findings

### <a name="L-1"></a>[L-1] Missing token approval when changing the reward token in Compounder

The [`setRewardToken()`](https://github.com/asymmetryfinance/afLiquidLocker/blob/c82ff88ffe0eb214372465f65e8dd9c50a0426e6/src/Compounder.sol#L147) function can be used to change the associated reward token in the Compounder contract. This token is expected to keep an active approval with the swapper contract, which needs to pull the tokens when rewards are compounded.

When changing the reward token, it is recommended to first revoke the approval to the old reference and update the approval to the new address.

### <a name="L-2"></a>[L-2] Sweep recipient can be empty when Locker contract is created

The Locker [constructor](https://github.com/asymmetryfinance/afLiquidLocker/blob/c82ff88ffe0eb214372465f65e8dd9c50a0426e6/src/Locker.sol#L32) does not check if the given `_sweepRecipient` argument is different from `address(0)`. 

An empty reference could lead to loss of funds if the recipient is not properly configured before rewards are swept.

## Informational Findings

### <a name="I-1"></a>[I-1] Claiming Stargate's fees is permissionless

The action to claim Stargate's fees can be executed via the proxy using the `claim()` function, restricted to approved callers.

```solidity
72:     function claim(address _asset) external onlyApprovedCaller {
73:         locker.execute(FEE_DISTRIBUTOR, 0, abi.encodeWithSignature("claimToken(address,address)", address(locker), _asset));
74:     }
```

However, Stargate's [FeeDistributor](https://etherscan.io/address/0xAF667811A7eDcD5B0066CD4cA0da51637DB76D09#code#F1#L348) contract allows anyone to claim fees on behalf of another account by default.

```solidity
function claimToken(address user, IERC20 token) external override nonReentrant userAllowedToClaim(user) tokenCanBeClaimed(token) returns (uint256) {
    _checkpointTotalSupply();
    _checkpointUserBalance(user);
    _checkpointToken(token, false);

    return _claimToken(user, token);
}

modifier userAllowedToClaim(address user) {
    if (_onlyVeHolderClaimingEnabled[user]) {
        require(msg.sender == user, "Claiming is not allowed");
    }
    _;
}
```

If the intention is to restrict claiming only through the proxy, then make sure to call `enableOnlyVeHolderClaiming()` to turn on the flag to only allow the holder to claim fees.

### <a name="I-2"></a>[I-2] Confusing logic in keeper trigger

The Compounder contract exposes a `harvestTrigger()` function to determine when keepers should call the `harvest()`.

```solidity
096:     function harvestTrigger() external view returns (bool) {
097:         uint256 _claimable = rewardDistributor.getClaimable(address(this));
098:         if (_claimable > swapThresholds.max) return true; // if the rewards are above the max threshold
099:         if (block.timestamp - lastSwapTimestamp < thresholdTimeBetweenSwaps) return false; // if not enough time has passed since the last swap
100:         if (block.basefee > acceptableBaseFee) return false; // if the base fee is too high
101:         if (_claimable > 0) return true; // if there are rewards to claim
102:         if (IERC20(asset()).balanceOf(address(this)) > minStakeAmount) return true; // if there are assets to stake
103:         if (rewardToken.balanceOf(address(this)) > swapThresholds.min) return true; // if there are rewards to swap
104:         return false;
105:     }
```

There are different aspects that are either not clear or are conflicting with the harvest implementation.

- Staking and swapping should be treated as independent actions. The check in line 102 happens after other short-circuit checks that can prevent staking even if the threshold is reached.
- This function will likely be called off-chain, which means the base fee will potentially be different when the actual harvest call is made.
- The check in line 101 seems weak and unnecessary if the threshold is not met. This could be combined with the check in line 103 using something similar to `_claimable + rewardToken.balanceOf(address(this)) > swapThresholds.min`.

### <a name="I-3"></a>[I-3] Consider adding a slippage parameter in the harvest trigger

Since the harvest function is expected to be called by whitelisted bots, it should be possible to add a slippage parameter in `harvest()` to be forwarded and used in the underlying swap of rewards. 

This would improve the on-chain guarantees instead of relying solely on a private mempool. 

### <a name="I-4"></a>[I-4] Missing events in Compounder setters

None of the setters present in the Compounder contract emit events to reflect the corresponding change.

### <a name="I-5"></a>[I-5] Consider delegating Stargate voting power from Locker contract

In addition to fees, veSTG lockers are entitled to governance features, including [creating proposals and voting](https://stargateprotocol.gitbook.io/stargate/v/user-docs/governance/proposals-and-voting).

Stargate manages proposals and voting through the [Snapshot](https://snapshot.org/#/stgdao.eth) platform. Voting power held by smart contracts can be delegated using the delegate registry, as explained in Snapshot's [documentation](https://docs.snapshot.org/user-guides/delegation#smart-contract-interaction).

## Gas Findings

### <a name="G-1"></a>[G-1] Gas improvements in Zap contract

In the [`zap()`](https://github.com/asymmetryfinance/afLiquidLocker/blob/c82ff88ffe0eb214372465f65e8dd9c50a0426e6/src/utils/Zap.sol#L83) function, the amount is checked to be different from zero two times (lines 94 and 97). The first check can be removed, to have a single check after the `type(uint256).max` case is handled.

The check in line 116 that deals with the received amount of afSTG should not be needed. It would either fail when converting it to the output token or ultimately fail due to the slippage check.

When the input token is a-afSTG, the implementation first transfers the shares to the Zap contract (line 100) and then redeems the shares from the contract (line 106). Instead of pulling the tokens, it should be possible to directly redeem those from the caller to save one step.

The [`_isInList()`](https://github.com/asymmetryfinance/afLiquidLocker/blob/c82ff88ffe0eb214372465f65e8dd9c50a0426e6/src/utils/Zap.sol#L166) function used to check if the input or output token is supported relies on a storage array that may need multiple loads from storage to resolve the result. Since the tokens are a fixed set and are already present as immutable variables, this check could be implemented using a `if (...) ... else if (...) ...` structure that completely avoids loading from storage.

### <a name="G-2"></a>[G-2] Cache storage variable in `harvest()`

The `rewardDistributor` variable is fetched twice from storage when there are rewards to be claimed. Consider storing its reference in a local variable.

### <a name="G-3"></a>[G-3] Duplicate mint event in LiquidLocker

The [`mint()`](https://github.com/asymmetryfinance/afLiquidLocker/blob/c82ff88ffe0eb214372465f65e8dd9c50a0426e6/src/LiquidLocker.sol#L42) function emits a `Mint` event as part of its implementation, but a second event is also fired in the underlying implementation of `_mint()` given by the ERC20 base contract.

