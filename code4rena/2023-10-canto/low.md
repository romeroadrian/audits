# Report

## Summary

### Low Issues

Total of **5 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-claimconcentratedrewards-and-claimambientrewards-should-be-internal-function) | `claimConcentratedRewards()` and `claimAmbientRewards()` should be internal function |
| [L-2](#l-2-governance-check-is-commented-out) | Governance check is commented out |
| [L-3](#l-3-rewards-can-be-unintentionally-overridden) | Rewards can be unintentionally overridden |
| [L-4](#l-4-unused-rewards-cannot-be-claimed-back) | Unused rewards cannot be claimed back |
| [L-5](#l-5-missing-week-validation-while-claiming-rewards) | Missing week validation while claiming rewards |

### Non Critical Issues

Total of **1 issues**:

|ID|Issue|
|:--:|:---|
| [NC-1](#nc-1-use-solidity-time-units) | Use Solidity time units |

## Low Issues

### <a name="L-1"></a>[L-1] `claimConcentratedRewards()` and `claimAmbientRewards()` should be internal function

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L61

Both of these functions present in the LiquidityMiningPath contract are intended to be called via user commands whose entrypoint is the `userCmd()` function.

Consider changing the visibility of `claimConcentratedRewards()` and `claimAmbientRewards()` to internal or private.

### <a name="L-2"></a>[L-2] Governance check is commented out

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L66
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L75

Both `setConcRewards()` and `setAmbRewards()` have a privilege check that is commented out and currently ignored:

```solidity
// require(msg.sender == governance_, "Only callable by governance");
```

These functions are called as protocol commands, which undergo a privilege check in the entrypoint (see https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/CrocSwapDex.sol#L104), hence the low severity of this issue. 

However, it is not clear if there's a missing additional check to ensure these are called by the governance. Consider either removing the lines or uncommenting the check.

### <a name="L-3"></a>[L-3] Rewards can be unintentionally overridden

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L65
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L74

Lidiquity mining rewards are set up using `setConcRewards()` or `setAmbRewards()`. In both cases, the rewards are overridden instead of accumulated. Taking `setConcRewards()` as the example:

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L65-L72

```solidity
65:     function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
66:         // require(msg.sender == governance_, "Only callable by governance");
67:         require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
68:         while (weekFrom <= weekTo) {
69:             concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
70:             weekFrom += uint32(WEEK);
71:         }
72:     }
```

Line 69 assigns the new value `weeklyReward` to the storage mapping. If rewards were already set for this pool, i.e. `concRewardPerWeek_[poolIdx][weekFrom] != 0`, then the new assignment will override the existing value.

### <a name="L-4"></a>[L-4] Unused rewards cannot be claimed back

Although highly unlikely, there is no provided mechanism to claim back unused rewards assigned to periods of time of inactivity in the pool.

### <a name="L-5"></a>[L-5] Missing week validation while claiming rewards

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L156
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L256

In `claimConcentratedRewards()` and `claimAmbientRewards()`, the implementation takes an array of the weeks which are expected to be claimed. This is user input, and lacks any validation over the provided argument values.

Using `claimConcentratedRewards()` as the example, we can see each element in `weeksToClaim` is not checked to be an actual _week_, i.e. that `week % WEEK == 0`.

```solidity
174:         for (uint256 i; i < weeksToClaim.length; ++i) {
175:             uint32 week = weeksToClaim[i];
176:             require(week + WEEK < block.timestamp, "Week not over yet");
177:             require(
178:                 !concLiquidityRewardsClaimed_[poolIdx][posKey][week],
179:                 "Already claimed"
180:             );
181:             uint256 overallInRangeLiquidity = timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][week];
```

Consider adding an explicit check to ensure the elements in `weeksToClaim` are valid weeks.

```diff
    uint32 week = weeksToClaim[i];
    require(week + WEEK < block.timestamp, "Week not over yet");
+   require(week % WEEK == 0, "Invalid week");
```

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] Use Solidity time units

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L13

Instead of defining spans of time manually using seconds, consider using Solidity built-in [time units](https://docs.soliditylang.org/en/v0.8.21/units-and-global-variables.html#time-units).

