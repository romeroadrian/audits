## Define `WEEK` constant using native units

The `WEEK` constant can be defined by using `7 days`.

## Validate `chestAddress` in constructor

Validate `chestAddress != address(0)` during contract initialization.

## Validate `minTargetVotes` in constructor

Validate that `minTargetVotes > 0` in the contract's constructor to match the semantics in the associated `updateMinTargetVotes` setter.

## Consider adding a "paginated" version of `getAllPledges`

The `pledges` array will eventually grow in size as the contract is used and will keep all historic pledges. Consider adding a paginated (offset + length) version of this accessor.

## Validate `endTimestamp` in the `_pledge` function

Validate that the `endTimestamp` parameter is not in the past. This isn't a big issue since `uint256 boostDuration = endTimestamp - block.timestamp;` will underflow and revert, but consider adding a validation and returning a proper error.
