<table>
    <tr>
        <th colspan="2">
            <img src="https://raw.githubusercontent.com/romeroadrian/audits/main/images/banner.png" width="800"/>
        </th>
    </tr>
    <tr>
        <td align="center"><img src="https://www.asymmetry.finance/images/afcvx/afcvx_btn1.svg" width="200" height="200" align="center"/></td>
        <td>
            <h1>Asymmetry Finance Report</h1>
            <h2>afCVX</h2>
            <p>Conducted by: adriro (<a href="https://twitter.com/adrianromero">@adrianromero</a>)</p>
            <p>Date: June 10 to 12, 2024</p>
        </td>
    </tr>
</table>

# afCVX Security Review

## Disclaimer

_The conducted security review represents an evaluation based on the information and code provided by the client. The author, employing a combination of automated tools and manual expertise, has endeavored to identify potential vulnerabilities. It is crucial to understand that this review does not ensure the absolute absence of vulnerabilities or errors within the smart contracts._

_Despite exercising due diligence, this assessment may not uncover all potential issues or undiscovered vulnerabilities present in the code. Findings and recommendations are based solely on the information available at the time of the review._

_This report is not to be considered as an endorsement or certification of the smart contract's absolute security. Authors cannot assume responsibility for any losses or damages that may arise from the utilization of the smart contracts._

_While this assessment aims to identify vulnerabilities, it cannot guarantee absolute security or eliminate all potential risks associated with smart contract usage._

## About afCVX

[afCVX](https://medium.com/@asymmetryfin/introducing-afcvx-fb744bd24d85) is a new protocol by Asymmetry Finance built to maximize yield on CVX tokens. The design works as a hybrid CVX wrapper, in which a share of the tokens remain liquid in the Convex staking rewards pool, while the rest is deposited at CLever CVX, a protocol that enables CVX locking with the option to leverage on future yield. Rewards coming from both of these underlying platforms are compounded back into the protocol.

## About adriro

adriro is an independent security researcher currently focused on security reviews of smart contracts. He is a top warden at [code4rena](https://code4rena.com/) and serves as a resident auditor at [yAudit](https://yaudit.dev/).

You can follow him on X at [@adrianromero](https://x.com/adrianromero) or browse his [portfolio](https://github.com/romeroadrian/audits).

## Scope

The scope for the current review targets the afCVX refactor present in the [fix-accounting](https://github.com/asymmetryfinance/afCVX/tree/fix-accounting) branch  at revision [a8f0820050abc47e85e8ef1ed943363f5e85e3bd](https://github.com/asymmetryfinance/afCVX/tree/a8f0820050abc47e85e8ef1ed943363f5e85e3bd) and includes the following files:

```
src
├── AfCvx.sol
└── strategies
    └── CLeverCVXStrategy.sol
```

## Summary

| Identifier | Title | Severity | Status |
| ---------- | ----- | ---------| ------ |
| [C-1](#c-1-afcvx-withdrawal-ignores-the-portion-of-unlocked-assets) | afCVX withdrawal ignores the portion of unlocked assets | Critical | Fixed |
| [M-1](#m-1-previewwithdraw-should-round-in-favor-of-the-vault) | `previewWithdraw()` should round in favor of the vault | Medium | Fixed |
| [M-2](#m-2-incorrect-fee-calculation-in-previewredeem) | Incorrect fee calculation in `previewRedeem()` | Medium | Fixed |
| [M-3](#m-3-clevercvxstrategy-cannot-be-paused) | CleverCvxStrategy cannot be paused | Medium | Fixed |
| [L-1](#l-1-set-maintenance-window-even-if-there-are-no-pending-obligations) | Set maintenance window even if there are no pending obligations | Low | Fixed |
| [I-1](#i-1-missing-token-allowance-revocation-on-pausing) | Missing token allowance revocation on pausing | Informational | Ack |
| [I-2](#i-2-unused-constants-and-events) | Unused constants and events | Informational | Fixed |
| [I-3](#i-3-rounding-up-clever-repayment-fee-is-not-needed) | Rounding up CLever repayment fee is not needed | Informational | Ack |
| [I-4](#i-4-locked-cvx-in-clever-cannot-be-fully-withdrawn-when-having-debt) | Locked CVX in CLever cannot be fully withdrawn when having debt | Informational | Ack |
| [G-1](#g-1-optimizations-in-calculatedistribute) | Optimizations in `_calculateDistribute()` | Gas | Partial |
| [G-2](#g-2-optimizations-in-clevercvxstrategy-contract) | Optimizations in CleverCvxStrategy contract | Gas | Partial |

## Critical Findings

### <a name="C-1"></a>[C-1] afCVX withdrawal ignores the portion of unlocked assets

###### Summary

Users withdrawing from afCVX would receive less assets than expected as the implementation fails to transfer the portion of unlocked assets.

###### Details

Immediate withdrawals in afCVX are handled using unlocked assets (free CVX balance in the contracts) and assets staked in the CVX rewards pool. When a withdrawal is requested, the implementation uses any available unlocked balance and withdraws from the rewards pool if needed.

```solidity
373:     function _withdraw(address _caller, address _receiver, address _owner, uint256 _assets, uint256 _shares) internal override {
374:         unchecked {
375:             weeklyWithdrawalLimit -= uint128(_assets);
376:         }
377: 
378:         if (_assets != 0) {
379:             uint256 _idle = CVX.balanceOf(address(this));
380:             if (_idle < _assets) {
381:                 unchecked {
382:                     _assets -= _idle;
383:                 }
384:                 CVX_REWARDS_POOL.withdraw(
385:                     _assets, // amount
386:                     false // claim
387:                 );
388:             }
389:         }
390: 
391:         if (_caller != _owner) _spendAllowance(_owner, _caller, _shares);
392: 
393:         // Need to transfer after minting or ERC777s could reenter
394:         _burn(_owner, _shares);
395:         CVX.safeTransfer(_receiver, _assets);
396: 
397:         emit Withdraw(_caller, _receiver, _owner, _assets, _shares);
398:     }
```

We can see that lines 379-388 handle this logic. If the idle balance is not enough (line 380) the implementation tries to withdraw the missing part from the rewards pool.

However, the function incorrectly reuses the `_assets` variable and overwrites it with the required amount to withdraw from the rewards pool. This variable is later used in line 395 to transfer the assets back to the user, meaning the user will only receive the amount that comes from the unstaked assets from the pool.

###### Impact

Withdrawals that are covered by both unlocked and staked assets will miss the unlocked part, causing a loss of funds to the user.

###### Recommendation

Use a different variable to represent the amount of assets to unstake.

```diff
    if (_assets != 0) {
        uint256 _idle = CVX.balanceOf(address(this));
        if (_idle < _assets) {
+           uint256 _unstakeAmount;
            unchecked {
-               _assets -= _idle;
+               _unstakeAmount = _assets - _idle;
            }
            CVX_REWARDS_POOL.withdraw(
-               _assets, // amount
+               _unstakeAmount, // amount
                false // claim
            );
        }
    }
```

## High Findings

None.

## Medium Findings

### <a name="M-1"></a>[M-1] `previewWithdraw()` should round in favor of the vault

###### Summary

The implementation of [`previewWithdraw()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/AfCvx.sol#L182) incorrectly rounds resulting shares down.

###### Details

When withdrawing assets from the vault, the implementation converts the desired amount of assets to shares, rounding down on the result.

```solidity
182:     function previewWithdraw(uint256 _assets) public view override returns (uint256) {
183:         return _convertToShares(
184:             _assets + Math.mulDiv(_assets, withdrawalFeeBps, PRECISION, Math.Rounding.Ceil),
185:             Math.Rounding.Floor
186:         );
187:     }
```

###### Impact

Rounding in favor of the user could lead to severe consequences if the value of a share is inflated.

###### Recommendation

The call to `_convertToShares()` should round up.

```diff
    function previewWithdraw(uint256 _assets) public view override returns (uint256) {
        return _convertToShares(
            _assets + Math.mulDiv(_assets, withdrawalFeeBps, PRECISION, Math.Rounding.Ceil),
-           Math.Rounding.Floor
+           Math.Rounding.Ceil
        );
    }
```

### <a name="M-2"></a>[M-2] Incorrect fee calculation in `previewRedeem()`

###### Summary

The [`previewRedeem()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/AfCvx.sol#L190) function incorrectly calculates the fee as it were exclusive of the given amount.

###### Details

When redeeming shares, fees should be taken on the total amount of assets resulting from the conversion of the shares. The current implementation calculates the fees on top of the given amount.

```solidity
190:     function previewRedeem(uint256 _shares) public view override returns (uint256) {
191:         uint256 _assets = _convertToAssets(_shares, Math.Rounding.Floor);
192:         return _assets - Math.mulDiv(_assets, withdrawalFeeBps, PRECISION, Math.Rounding.Ceil);s
193:     }
```

Since the intention here is to redeem a specific number of assets, resulting from the conversion of the given shares, the calculation should take fees as included in the total assets. 

###### Impact

Users redeeming shares would receive less assets than intended due to a higher than expected fee.

###### Recommendation

Fee calculation in `previewRedeem()` should be `(assets * fee / (fee + bps))`, rounding up.

```diff
    uint256 _assets = _convertToAssets(_shares, Math.Rounding.Floor);
-   return _assets - Math.mulDiv(_assets, withdrawalFeeBps, PRECISION, Math.Rounding.Ceil);
+   uint256 _withdrawalFeeBps = withdrawalFeeBps;
+   return _assets - Math.mulDiv(_assets, _withdrawalFeeBps, _withdrawalFeeBps + PRECISION, Math.Rounding.Ceil);
```

### <a name="M-3"></a>[M-3] CleverCvxStrategy cannot be paused

###### Summary

The CleverCvxStrategy contract includes the pause logic but no way to enable it.

###### Details

The emergency shutdown functionality has been removed in the refactor. While the new version of the AfCVX contract contains a [`setPause()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/AfCvx.sol#L116) function, there is currently no way to modify the `paused` variable in the CleverCvxStrategy contract.

###### Impact

CleverCvxStrategy would still be accessible even if the main AfCVX contract is paused.

###### Recommendation

Either add a function in CleverCvxStrategy to pause the contract, or trigger the pause from `AfCVX::setPaused()`.

## Low Findings

### <a name="L-1"></a>[L-1] Set maintenance window even if there are no pending obligations

The maintenance window ensures that no further requests are made in the current epoch after the operator has processed the pending obligations during its routine processing.

The implementation of [`unlock()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L279) sets the `maintenanceWindowEnd` variable, but only if pending obligations are not zero.

```solidity
279:     function unlock() external onlyOperatorOrOwner {
280:         uint256 _unlockObligations = unlockObligations;
281:         if (_unlockObligations != 0) {
282:             unlockObligations = 0;
283:             CLEVER_CVX_LOCKER.unlock(_unlockObligations);
284: 
285:             // The start of the next epoch. Until then unlock requests are blocked
286:             maintenanceWindowEnd = block.timestamp / REWARDS_DURATION * REWARDS_DURATION + REWARDS_DURATION;
287:         }
288:         unlockInProgress = false;
289:     }
```

This could lead to a situation in which there are no unlocks requested up until the moment the operator executes the processing, but new requests are still allowed between this processing and the end of the epoch.

It is recommended to set `maintenanceWindowEnd` even if `unlockObligations == 0` and ensure this process is executed in all epochs, regardless of whether there are pending requests or not.

## Informational Findings

### <a name="I-1"></a>[I-1] Missing token allowance revocation on pausing

The previous version of the contracts implemented a `emergencyShutdown()` function which not only paused the contracts, but also removed the token allowances by calling `_emergencyRevokeAllAllowances()`.

The new version has the pause, but leaves the token approvals for third party contracts.

### <a name="I-2"></a>[I-2] Unused constants and events

The following definitions are not used and could be removed:

- [AfCvx.sol#L419](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/AfCvx.sol#L419)
- [AfCvx.sol#L435](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/AfCvx.sol#L435)
- [CLeverCVXStrategy.sol#L48](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L48)
- [CLeverCVXStrategy.sol#L323](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L323)
- [CLeverCVXStrategy.sol#L330](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L330)

### <a name="I-3"></a>[I-3] Rounding up CLever repayment fee is not needed

The [`maxTotalUnlock()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L118) and [`_calculateRepayAmount()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L295) functions calculate the CLever repayment fee by rounding up the result.

This is not needed since the implementation of [`repay()`](https://github.com/AladdinDAO/aladdin-v3-contracts/blob/63198430f5e4cfd981808b35ef5002a2b290807f/contracts/clever/CLeverCVXLocker.sol#L451) in CLeverCVXLocker always rounds down the fee calculation.

```solidity
483:       uint256 _fee = _clevCVXAmount.mul(repayFeePercentage) / FEE_PRECISION;
```

### <a name="I-4"></a>[I-4] Locked CVX in CLever cannot be fully withdrawn when having debt

The refactored implementation of [`maxTotalUnlock()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L118) always reserves the repayment fee associated with the current debt from the maximum amount that can be removed from CLever.

```solidity
121:         uint256 _repayFeePercentage = CLEVER_CVX_LOCKER.repayFeePercentage();
122:         uint256 _repayFee =
123:                 _borrowed * _repayFeePercentage
124:                 / CLEVER_PRECISION + (_borrowed * _repayFeePercentage % CLEVER_PRECISION == 0 ? 0 : 1);
125: 
126:         uint256 _unlockObligations = unlockObligations;
127:         return _unlockObligations + _repayFee >= _deposited ? 0 : _deposited - _repayFee - _unlockObligations;
```

This means that when `_borrowed > 0` (i.e. there is outstanding debt), not all locked assets in CLever can be withdrawn. Eventually, the last portion of CVX tokens in CLever will need to remain locked unless the system winds down the debt. Note also that the repay fee, along with the debt itself, is paid using clevCVX from the Furnace, which is a balance separate from the deposited assets in the CLever locker.

## Gas Findings

### <a name="G-1"></a>[G-1] Optimizations in `_calculateDistribute()`

There are multiple gas optimizations available in the implementation of [`_calculateDistribute()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/AfCvx.sol#L310).

- The call to `totalAssets()` in line 317 should be already available by adding `_totalDeposit + _assetsInConvex + _assetsInCLever`.
- `_targetAssetsInConvex` calculation in line 320 could be done using `_totalAssets - _targetAssetsInCLever`.
- Subtractions in lines 322, 323 and 332 could be done using unchecked math.
- `_convexDeposit` calculation in line 335 could be done by subtracting `_remainingDeposit` from the result of line 334 (should be split from `_cleverDeposit`).
- `_convexDeposit` calculation in line 340 could be done using `_totalDeposit - _cleverDeposit`

These should also help with potential leftovers due to rounding on share calculations.

### <a name="G-2"></a>[G-2] Optimizations in CleverCvxStrategy contract

- In line 181, the addition could use the current value of `unlockObligations` available in the `_currentUnlockObligations` variable.
- In the [`requestUnlock()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#L176) function, subtractions in lines 193, 196 and 203 could be done using unchecked math.
- In [`_calculateRepayAmount()`](https://github.com/asymmetryfinance/afCVX/blob/a8f0820050abc47e85e8ef1ed943363f5e85e3bd/src/strategies/CLeverCVXStrategy.sol#295), the subtraction present in line 305 could be done using unchecked math.

