# [ADRIRO-NEW-H-02] Users loses their share of rewards while waiting for withdrawal

## Summary

Withdrawals in AfEth undergo a delay until the underlying CVX tokens can be withdrawn. Depositors need to request a withdrawal and wait until the required withdrawal epoch before making their withdrawal effective. During this period of time, they will lose their share of any compounded reward into SafEth.

## Impact

Withdrawals in AfEth are first requested using the `requestWithdraw()`. As we can see in the implementation, the function will calculate the number of SafEth tokens corresponding to the user's share (`safEthWithdrawAmount`) and store that information to be used when the withdrawal can be finally executed.

https://github.com/asymmetryfinance/afeth/blob/74f340568480aa03d043e970fcf2578bea037cf6/contracts/AfEth.sol#L222-L236

```solidity
222:         uint256 safEthBalance = safEthBalanceMinusPending();
223: 
224:         uint256 safEthWithdrawAmount = (withdrawRatio * safEthBalance) / 1e18;
225: 
226:         pendingSafEthWithdraws += safEthWithdrawAmount;
227: 
228:         withdrawIdInfo[latestWithdrawId]
229:             .safEthWithdrawAmount = safEthWithdrawAmount;
230:         withdrawIdInfo[latestWithdrawId]
231:             .votiumWithdrawAmount = votiumWithdrawAmount;
232:         withdrawIdInfo[latestWithdrawId].vEthWithdrawId = vEthWithdrawId;
233: 
234:         withdrawIdInfo[latestWithdrawId].owner = msg.sender;
235:         withdrawIdInfo[latestWithdrawId].amount = _amount;
236:         withdrawIdInfo[latestWithdrawId].withdrawTime = withdrawTimeBefore;
```

Line 224 calculates the amount based on the user's share and the total number of SafEth tokens held by the AfEth contract. This amount **gets fixed at this moment** and is stored in the `withdrawIdInfo` mapping, which is later used by `withdraw()` to send the funds to the user:

https://github.com/asymmetryfinance/afeth/blob/74f340568480aa03d043e970fcf2578bea037cf6/contracts/AfEth.sol#L284-L290

```solidity
284:         if (withdrawInfo.safEthWithdrawAmount > 0) {
285:             ISafEth(SAF_ETH_ADDRESS).unstake(
286:                 withdrawInfo.safEthWithdrawAmount,
287:                 0
288:             );
289:             trackedsafEthBalance -= withdrawInfo.safEthWithdrawAmount;
290:         }
```

Protocol rewards coming from the VotiumStrategy in AfEth are compounded back into the protocol. Depending on the current state of SafEth and vAfEth TVL, and the desired target ratio, rewards can be compounded into SafEth if the current ratio is below the target value, in order to increase the SafEth side. This can be seen in the implementation of `depositRewards()`:

https://github.com/asymmetryfinance/afeth/blob/74f340568480aa03d043e970fcf2578bea037cf6/contracts/AfEth.sol#L322-L333

```solidity
322:         uint256 safEthTvl = (ISafEth(SAF_ETH_ADDRESS).approxPrice(true) *
323:             safEthBalanceMinusPending()) / 1e18;
324:         uint256 votiumTvl = ((votiumStrategy.cvxPerVotium() *
325:             votiumStrategy.ethPerCvx(true)) * trackedvStrategyBalance) / 1e36;
326:         uint256 totalTvl = (safEthTvl + votiumTvl);
327:         uint256 safEthRatio = (safEthTvl * 1e18) / totalTvl;
328:         if (safEthRatio < ratio) {
329:             uint256 safEthReceived = ISafEth(SAF_ETH_ADDRESS).stake{
330:                 value: amount
331:             }(_safEthMinout);
332:             trackedsafEthBalance += safEthReceived;
333:         } else {
```

As we can see in lines 329-331, rewards compounded into SafEth are executed by staking into the protocol which **involves the minting of new SafEth tokens**.

Combining this with the fact that the user's share of SafEth tokens are calculated when the user requests the withdrawal, it means that any rewards earned and compounded into SafEth during the period of time the user is waiting to execute their withdrawal will be lost. As the user is still locked into the platform, and knowing that their share of CVX tokens are still generating revenue to the protocol, it is unfair that they don't receive their share of it.

This only happens with the SafEth strategy, as compounded rewards into SafEth increment the number of SafEth tokens the AfEth contract holds, but compounded rewards into the Votium strategy are directly considered by the held tokens (which behaves more like a vault).

Note that, with the new changes, even if the CVX balance is enough to cover the withdrawal, the request is delayed until the next epoch, which implies a minimum of one week. The user will be forced to wait at least one week, up to the maximum of 16 weeks imposed by Convex in the worst case, and will lose any rewards compounded into SafEth during this period.

## Proof of Concept 

Let's assume a user owns 10% of the total share of AfEth. The current amount of SafEth tokens held in the AfEth contract is 1000. Let's also say that the target ratio is 50% and the current TVL of SafEth is 40% of the total.

1. The user requests the withdrawal of all their owned AfEth. The protocol has 1000 SafEth tokens, hence their share is calculated at 100 tokens.
2. The withdrawal is scheduled for a future epoch since it also needs to withdraw from Votium, let's say this is scheduled for 2 epochs (nearly two weeks).
3. In between this period of time, rewards are applied in the protocol. Since the SafEth TVL is below the target ratio, rewards are compounded into the SafEth side. Let's say the total amount of new minted SafEth tokens is 100.
4. The withdrawal period is finished and the user can finally withdraw. The user will receive 100 SafEth tokens, since this amount was calculated and fixed in step 1. The user doesn't receive the 10 tokens from their share of the applied rewards in step 3.

## Recommendation

The easiest path to solve the issue would be to delay the calculation of the user's SafEth share until the time the withdraw is made effective. This way any potential reward that is applied during the waiting time is considered in the resulting amount that is sent when `withdraw()` is called.
