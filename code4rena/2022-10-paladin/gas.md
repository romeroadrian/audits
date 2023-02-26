## Unchecked math in `_pledge` function

The update in line 268 can be done using unchecked math since `rewardAmount <= pledgeAvailableRewardAmounts[pledgeId]` due to the check in line 267.

## Unneeded storage read in `createPledge` function

In line 340 there's an unnecessary read from storage since `pledgeAvailableRewardAmounts[vars.newPledgeID]` should be 0 since we are creating a new pledge with a new unused id. Consider changing this line to be a simple assignment:

```
pledgeAvailableRewardAmounts[vars.newPledgeID] = vars.totalRewardAmount;
```

## `pledgeParams.endTimestamp` is read twice in `extendPledge` and `increasePledgeRewardPerVote`

This storage variable is readed twice in lines 380 and 382 in the `extendPledge` function and similarly in lines 426 and 430 in the `increasePledgeRewardPerVote` function. Consider storing the first read from storage locally to prevent a re-read.

## `pledgeParams.rewardToken` is read twice in `extendPledge` and `increasePledgeRewardPerVote`

This storage variable is readed twice in lines 394 and 396 in the `extendPledge` function and similarly in lines 438 and 440 in the `increasePledgeRewardPerVote` function. Consider storing the first read from storage locally to prevent a re-read.
