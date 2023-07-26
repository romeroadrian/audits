# Report

## Low Issues

| |Issue|
|-|:-|
| [L-1](#L-1) | Division before multiplication during token amounts calculation |
| [L-2](#L-2) | Rage quit forfeits pending claims in TokenDistributor |
| [L-3](#L-3) | Follow CEI in `rageQuit()` function |
| [L-4](#L-4) | `rageQuit()` can be front-runned to diminish user's share of tokens |

### <a name="L-1"></a>[L-1] Division before multiplication during token amounts calculation

- https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L335
- https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L341

The calculation of the token amounts in the `rageQuit()` function first calculate the user's share using `getDistributionShareOf()`, which involves a division, and then projects this value over to the total available amount of assets, leading to a situation of division before multiplication.

The calculations can be simplified as `(token.balanceOf(address(this)) * votingPowerByTokenId[tokenId]) / totalVotingPower`.

### <a name="L-2"></a>[L-2] Rage quit forfeits pending claims in TokenDistributor

Pending distributions available in the TokenDistributor contract will be lost since the distributor contract uses the `getDistributionShareOf()` function to calculate the amount to be claimed.

https://github.com/code-423n4/2023-05-party/blob/main/contracts/distribution/TokenDistributor.sol#L234-L245

```solidity
function getClaimAmount(
    ITokenDistributorParty party,
    uint256 memberSupply,
    uint256 partyTokenId
) public view returns (uint128) {
    // getDistributionShareOf() is the fraction of the memberSupply partyTokenId
    // is entitled to, scaled by 1e18.
    // We round up here to prevent dust amounts getting trapped in this contract.
    return
        ((uint256(party.getDistributionShareOf(partyTokenId)) * memberSupply + (1e18 - 1)) /
            1e18).safeCastUint256ToUint128();
}
```

Burning an NFT sets the `votingPowerByTokenId[tokenId]` to zero. This means that calling `rageQuit()` while having pending unclaimed distributions in the TokenDistributor contract will lead to loss of funds, as the token is burned, making `getDistributionShareOf()` to be zero and causing `getClaimAmount()` to be zero too.

### <a name="L-3"></a>[L-3] Follow CEI in `rageQuit()` function

https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L312-L347

The new `rageQuit()` function loops through each NTF token id, and each iteration burns the token and executes the asset withdrawals associated with that NFT.

Consider following the ["Checks Effects Interactions"](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) pattern and first burn all tokens and then execute the token withdrawals. This can be coupled with the recommendation given in the gas report to batch token withdrawals to improve gas costs.

### <a name="L-4"></a>[L-4] `rageQuit()` can be front-runned to diminish user's share of tokens

The `rageQuit()` function calculates the user's share of tokens using the `getDistributionShareOf()` function, which calculates the proportion of the voting power associated with the NFT and the total voting power.

https://github.com/code-423n4/2023-05-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L150-L158

```solidity
function getDistributionShareOf(uint256 tokenId) public view returns (uint256) {
    uint256 totalVotingPower = _governanceValues.totalVotingPower;

    if (totalVotingPower == 0) {
        return 0;
    } else {
        return (votingPowerByTokenId[tokenId] * 1e18) / totalVotingPower;
    }
}
```

A transaction to `rageQuit()` can be front-runned with a call to `increaseTotalVotingPower()` to increase the value of the denominator in the calculation, causing a decrease in the amount of the share and a decrease in the amount of tokens sent to the user. 

For example, front-running the transaction with a call to `increaseTotalVotingPower()` such that `totalVotingPower` ends up being `type(uint256).max` will practically nullify the share of the user. 

The standard recommendation in these cases is to include a parameter that acts as a "minimum amount" and revert if the calculated amount is below this minimum.

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Import declarations should import specific symbols | 26 |
| [NC-2](#NC-2) | Use named parameters for mapping type declarations | 7 |
### <a name="NC-1"></a>[NC-1] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (26)*:
```solidity
File: contracts/party/PartyGovernance.sol

4: import "../distribution/ITokenDistributorParty.sol";

5: import "../distribution/ITokenDistributor.sol";

6: import "../utils/ReadOnlyDelegateCall.sol";

7: import "../tokens/IERC721.sol";

8: import "../tokens/IERC20.sol";

9: import "../tokens/IERC1155.sol";

10: import "../tokens/ERC721Receiver.sol";

11: import "../tokens/ERC1155Receiver.sol";

12: import "../utils/LibERC20Compat.sol";

13: import "../utils/LibRawResult.sol";

14: import "../utils/LibSafeCast.sol";

15: import "../globals/IGlobals.sol";

16: import "../globals/LibGlobals.sol";

17: import "../proposals/IProposalExecutionEngine.sol";

18: import "../proposals/LibProposal.sol";

19: import "../proposals/ProposalStorage.sol";

21: import "./IPartyFactory.sol";

```

```solidity
File: contracts/party/PartyGovernanceNFT.sol

4: import "../utils/ReadOnlyDelegateCall.sol";

5: import "../utils/LibSafeCast.sol";

6: import "../utils/LibAddress.sol";

7: import "openzeppelin/contracts/interfaces/IERC2981.sol";

8: import "../globals/IGlobals.sol";

9: import "../tokens/IERC721.sol";

10: import "../vendor/solmate/ERC721.sol";

11: import "./PartyGovernance.sol";

12: import "../renderers/RendererStorage.sol";

```

### <a name="NC-2"></a>[NC-2] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18

*Instances (7)*:
```solidity
File: contracts/party/PartyGovernance.sol

149:         mapping(address => bool) hasVoted;

209:     mapping(address => bool) public isHost;

211:     mapping(address => address) public delegationsByVoter;

215:     mapping(uint256 => ProposalState) private _proposalStateByProposalId;

217:     mapping(address => VotingPowerSnapshot[]) private _votingPowerSnapshotsByVoter;

```

```solidity
File: contracts/party/PartyGovernanceNFT.sol

57:     mapping(uint256 => uint256) public votingPowerByTokenId;

59:     mapping(address => bool) public isAuthority;

```

