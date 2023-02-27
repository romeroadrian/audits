# Gas

## Use `uint256` values in `mutualConsents` mapping in `MutualConsent` contract

Having `bool` values will make the compiler emit extra code to sanitize them. See https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol#L23

## Use `uint256` values in `whitelistedFunctions` mapping in `SpigotState` struct

Having `bool` values will make the compiler emit extra code to sanitize them. See https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol#L23

## Unchecked math in `claimRevenue` function of `SpigotLib` library

Line 96 can be done using unchecked math due to the if guard in line 95 (`claimed > escrowedAmount`).

## Use `uint256` values in the `enabled` mapping in `EscrowState` struct

Having `bool` values will make the compiler emit extra code to sanitize them. See https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol#L23

## Avoid unnecessary storage read in function `enableCollateral` in EscrowLib 

The `IEscrow.Deposit` struct is initialized in line 108 by reading it from storage and copying it to memory. This is unnecessary since nothing is read from this struct during the function (and it doesn't make sense either, since this function enables the collateral meaning anything stored there should be empty/zeroed).

## Unchecked math in `releaseCollateral` function of `EscrowLib` library

Line 164 can be done using unchecked math due to the if guard in line 163 (`self.deposited[token].amount >= amount` holds true).

## Unchecked math in `liquidate` function of `EscrowLib` library

Line 202 can be done using unchecked math due to the if guard in line 200 (`self.deposited[token].amount >= amount` holds true).

## Simplify storage in `_updateOutstandingDebt` in LineOfCredit contract

Line 195 overwrites the full Credit struct, but only the `interestAccrued` member gets modified. Consider updating just this variable.

## Simplify storage in `accrueInterest` in LineOfCredit contract

Line 206 overwrites the full Credit struct, but only the `interestAccrued` member gets modified. Consider updating just this variable.

## Unchecked math in `useAndRepay` function of `SpigotedLine` contract

Line 144 can be done using unchecked math due to the if guard in line 143 (`amount <= unusedTokens[credit.token]` holds true).

## Unchecked math in `claimAndTrade` function of `SpigotedLineLib` contract

Line 101 can be done using unchecked math due to the if guard in line 100 (`oldClaimTokens > newClaimTokens` holds true).

Line 109 can be done using unchecked math due to the if guard in line 105 (`diff <= unused` holds true).

## Rate struct can be tightly packed if timestamp size is shortened

The Rate struct takes 2 storage slots. It can be shortened to a single slot if the `lastAccrued` timestamp gets shortened to, for example, uint40 which should be enough until the year 36812. Rates will need to be shortened to uint108, but that should also be plenty of precision for credit rates.

## Several contracts use delegatecalls to libraries in order to provide functionality which incurs in an extra gas cost

All of the libraries used in the codebase are defined as external functions, which will generate a delegatecall whenever they are used. 

This is probably used to fight code size, which may be understandable in the line of credit contracts, since it groups the LineOfCredit, EscrowedLine and SpigotedLine under the SecuredLine contract.

But in the case of the Spigot and Escrow contracts, these almost map 1-1 with respect to their libraries (SpigotLib and EscrowLib), which makes no sense and incurs in extra gas costs.

Also the `LineLib` library is a good candidate to be moved to internal functions, as the implementations are short and frequently used.
