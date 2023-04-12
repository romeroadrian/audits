# QA Report

- [Formal Verification](#FV)
- [Non Critical Issues](#NC)
- [Low Issues](#L)


## <a name="FV"></a>Formal Verification

### Validity of reconstructed tickets

The following Certora rule proves that reconstructed tickets from a random source are valid within the accepted range of `selectionSize` and `selectionMax` parameters.

Results: https://prover.certora.com/output/78195/0f1385d89b704d8cbe588829e162028c?anonymousKey=3f8b6d024580c29b4a4eb11e6efc776cb8e45615

```
methods {
  isValidTicket(uint256,uint8,uint8) returns (bool) envfree
  reconstructTicket(uint256,uint8,uint8) returns (uint120) envfree
}

rule reconstructedTicketIsValid() {
  uint256 random;
  uint8 selectionSize;
  uint8 selectionMax;

  require selectionMax <= 120;
  require selectionSize <= 16 || selectionSize <= (selectionMax -1);

  uint256 ticket = reconstructTicket(random, selectionSize, selectionMax);

  assert isValidTicket(ticket, selectionSize, selectionMax), "reconstructed ticket is not valid";
}
```

## <a name="NC"></a>Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Import declarations should import specific symbols | 53 |
| [NC-2](#NC-2) | Use named parameters for mapping type declarations | 17 |
| [NC-3](#NC-3) | Refactor common code across functions | 1 |
| [NC-4](#NC-4) | Unused local variables | 1 |
| [NC-5](#NC-5) | Use a modifier for access control | 1 |
| [NC-6](#NC-6) | Missing event for important parameter change | 2 |


### <a name="NC-1"></a>[NC-1] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (53)*:
```solidity
File: src/Lottery.sol

5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

6: import "@openzeppelin/contracts/utils/math/Math.sol";

7: import "src/ReferralSystem.sol";

8: import "src/RNSourceController.sol";

9: import "src/staking/Staking.sol";

10: import "src/LotterySetup.sol";

11: import "src/TicketUtils.sol";

```

```solidity
File: src/LotteryMath.sol

5: import "src/interfaces/ILottery.sol";

6: import "src/PercentageMath.sol";

```

```solidity
File: src/LotterySetup.sol

5: import "@openzeppelin/contracts/utils/math/Math.sol";

6: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

7: import "src/PercentageMath.sol";

8: import "src/LotteryToken.sol";

9: import "src/interfaces/ILotterySetup.sol";

10: import "src/Ticket.sol";

```

```solidity
File: src/LotteryToken.sol

5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

6: import "src/interfaces/ILotteryToken.sol";

7: import "src/LotteryMath.sol";

```

```solidity
File: src/RNSourceBase.sol

5: import "src/interfaces/IRNSource.sol";

```

```solidity
File: src/RNSourceController.sol

5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

6: import "src/interfaces/IRNSource.sol";

7: import "src/interfaces/IRNSourceController.sol";

```

```solidity
File: src/ReferralSystem.sol

5: import "@openzeppelin/contracts/utils/math/Math.sol";

6: import "src/interfaces/IReferralSystem.sol";

7: import "src/PercentageMath.sol";

```

```solidity
File: src/Ticket.sol

5: import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

6: import "src/interfaces/ITicket.sol";

```

```solidity
File: src/VRFv2RNSource.sol

5: import "@chainlink/contracts/src/v0.8/VRFV2WrapperConsumerBase.sol";

6: import "src/interfaces/IVRFv2RNSource.sol";

7: import "src/RNSourceBase.sol";

```

```solidity
File: src/interfaces/ILottery.sol

5: import "src/interfaces/ILotterySetup.sol";

6: import "src/interfaces/IRNSourceController.sol";

7: import "src/interfaces/ITicket.sol";

8: import "src/interfaces/IReferralSystem.sol";

```

```solidity
File: src/interfaces/ILotterySetup.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import "src/interfaces/ITicket.sol";

```

```solidity
File: src/interfaces/ILotteryToken.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

```solidity
File: src/interfaces/IRNSourceController.sol

5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

6: import "src/interfaces/IRNSource.sol";

```

```solidity
File: src/interfaces/IReferralSystem.sol

5: import "src/interfaces/ILotteryToken.sol";

```

```solidity
File: src/interfaces/ITicket.sol

5: import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

5: import "src/interfaces/IRNSource.sol";

```

```solidity
File: src/staking/StakedTokenLock.sol

5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

6: import "src/staking/interfaces/IStakedTokenLock.sol";

7: import "src/staking/interfaces/IStaking.sol";

```

```solidity
File: src/staking/Staking.sol

5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

7: import "src/interfaces/ILottery.sol";

8: import "src/LotteryMath.sol";

9: import "src/staking/interfaces/IStaking.sol";

```

```solidity
File: src/staking/interfaces/IStakedTokenLock.sol

5: import "src/staking/interfaces/IStaking.sol";

```

```solidity
File: src/staking/interfaces/IStaking.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import "src/interfaces/ILottery.sol";

```

### <a name="NC-2"></a>[NC-2] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18

*Instances (17)*:
```solidity
File: src/Lottery.sol

26:     mapping(address => uint256) private frontendDueTicketSales;

27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

36:     mapping(uint128 => uint120) public override winningTicket;

37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;

37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;

39:     mapping(uint128 => uint256) public override ticketsSold;

```

```solidity
File: src/RNSourceBase.sol

9:     mapping(uint256 => RandomnessRequest) internal requests;

```

```solidity
File: src/ReferralSystem.sol

17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;

17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;

19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;

21:     mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;

23:     mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;

25:     mapping(uint128 => uint256) public override minimumEligibleReferrals;

```

```solidity
File: src/Ticket.sol

14:     mapping(uint256 => ITicket.TicketInfo) public override ticketsInfo;

```

```solidity
File: src/staking/Staking.sol

19:     mapping(address => uint256) public override userRewardPerTokenPaid;

20:     mapping(address => uint256) public override rewards;

```

### <a name="NC-3"></a>[NC-3] Refactor common code across functions

- `claimRewards` and `unclaimedRewards` functions in the `Lottery` contract have a lot of duplicate functionality. Consider refactoring common code in a private function.

### <a name="NC-4"></a>[NC-4] Unused local variables

Unused variables should be removed.

```solidity
File: src/Lottery.sol

260:    uint256 winTier;

```

### <a name="NC-5"></a>[NC-5] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function's body.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L46-L49

```solidity
function onRandomNumberFulfilled(uint256 randomNumber) external override {
    if (msg.sender != address(source)) {
        revert RandomNumberFulfillmentUnauthorized();
    }
```

### <a name="NC-6"></a>[NC-6] Missing event for important parameter change	

Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

*Instances (2)*:

- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L24

- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L37

## <a name="L"></a>Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Contract files should define a locked compiler version | 2 |
| [L-2](#L-2) | `claimable` function doesn't validate ticket draw is finished | 1 |
| [L-3](#L-3) | `DRAWS_PER_YEAR` assumes one draw per week | 1 |
| [L-4](#L-4) | Simplify unpacking expression in `fixedReward` | 1 |
| [L-5](#L-5) | First win tier is always empty in `packFixedRewards` | 1 |
| [L-6](#L-6) | Limits in `getMinimumEligibleReferralsFactorCalculation` should be inclusive  | 1 |


### <a name="L-1"></a>[L-1] Contract files should define a locked compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

*Instances (2)*:
```solidity
File: src/VRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/staking/StakedTokenLock.sol

3: pragma solidity ^0.8.17;

```

### <a name="L-2"></a>[L-2] `claimable` function doesn't validate ticket draw is finished

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L159-L168

The function should validate that the draw associated with the ticket is already finished, as the function uses the winning ticket to run the calculation and this value will be undefined until the draw is finished and a ticket is selected.

### <a name="L-3"></a>[L-3] `DRAWS_PER_YEAR` assumes one draw per week

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L24

The `DRAWS_PER_YEAR` constant is fixed at 52 assuming one draw lasts one week, while the lottery can be configured with an arbitrary draw period.

### <a name="L-4"></a>[L-4] Simplify unpacking expression in `fixedReward`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L126-L127

The following calculation:

```solidity
uint256 mask = uint256(type(uint16).max) << (winTier * 16);
uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
```

Can be simplified using a single shift as:

```solidity
uint256 extracted = (nonJackpotFixedRewards >> (winTier * 16)) & type(uint16).max;
```

### <a name="L-5"></a>[L-5] First win tier is always empty in `packFixedRewards`

The rewards for the first win tier (tier 0) are always 0, meaning the lowest 16 bits of the packed rewards word are always 0 wasting this space. Consider offsetting the tier by 1 (store tier 1 in lowest 16 bits, tier 2 in next 16 bits, and so on) to take advantage of this space. This change also allows the protocol to support a max selection size of 17 instead of 16.

### <a name="L-6"></a>[L-6] Limits in `getMinimumEligibleReferralsFactorCalculation` should be inclusive

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L111-L130

```solidity
function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
    internal
    view
    virtual
    returns (uint256 minimumEligible)
{
    if (totalTicketsSoldPrevDraw < 10_000) {
        // 1%
        return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);
    }
    if (totalTicketsSoldPrevDraw < 100_000) {
        // 0.75%
        return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
    }
    if (totalTicketsSoldPrevDraw < 1_000_000) {
        // 0.5%
        return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
    }
    return 5000;
}
```

The [protocol documentation](https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token/rewards/referrals#referrers-allocation) specifies that the total ticket sold limits should be inclusive during the calculation of the minimum referral eligibility. However, the different conditions in the if statements use a strict inequality to define the bounds to calculate the eligibility factor.
