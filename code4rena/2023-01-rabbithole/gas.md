## Duplicate call to `onlyOwner` in `withdrawRemainingTokens` function of `Erc20Quest` and `Erc1155Quest`

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150

The `withdrawRemainingTokens` function present in both contracts validates the owner using the `onlyOwner` modifier, but the super call to the `Quest` contract will validate it again. One of these modifiers can be removed.

## Unneeded data in `IERC1155` transfer callbacks

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L42
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L61

The functions `_transferRewards` and `withdrawRemainingTokens` execute an `IERC1155` transfer using the literal string `0x00` as data, which is then used in hooks and forwarded during callbacks. This string is the literal value "0x00" of length 4 and doesn't add anything. This can be removed to save gas.

## Redundant check in `Quest` constructor

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L35

The `Quest` constructor validates that `endTime_ <= block.timestamp`, but this condition is already implied by `startTime_ <= block.timestamp` and `endTime_ <= startTime_` (lines 36 and 37) and can be safely removed to save gas.

## Unneeded storage variable initialization in `Quest` constructor

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45

The default value for the `redeemedTokens` variable is 0 and doesn't need to be initialized.

## Redundant storage set in `Quest.start` function

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L51

The function sets the `isPaused` variable to false but this is redundant as this is the default value and a quest can't be paused before it has started.

## Duplicated getters in `Quest` contract

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L140
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L145

Getters `getRewardAmount` and `getRewardToken` are already implemented by the `public` visibility of the underlying variables.

## Optimize `Quest` struct fields

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L19-L24

`totalParticipants` and `numberMinted` can be safely defined as `uint48` (max value is `281474976710656`) as to pack these two with the `questAddress` slot and reduce the struct storage by 2 slots.

```solidity
struct Quest {
    mapping(address => bool) addressMinted;
    address questAddress;
    uint48 totalParticipants;
    uint48 numberMinted;
}
```

## Unneeded `hash` parameter in `mintReceipt` function of `QuestFactory` contract

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219

The `mintReceipt` function takes the signed hash as a parameter, only to recalculate it based on the caller and given quest id and compare it against the parameter. There's no point in doing all this, as this hash can be simply constructed and checked using the signature.

```solidity
function mintReceipt(string memory questId_, bytes memory signature_) public {
    if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
    if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
    
    bytes32 hash_ = keccak256(abi.encodePacked(msg.sender, questId_);
    if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

    quests[questId_].addressMinted[msg.sender] = true;
    quests[questId_].numberMinted++;
    emit ReceiptMinted(msg.sender, questId_);
    rabbitholeReceiptContract.mint(msg.sender, questId_);
}
```

## `timestampForTokenId` is not used in `RabbitHoleReceipt`

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34

The contract stores the timestamp every time a token is minted. Since this isn't used on-chain, consider indexing timestamp off-chain to save gas.

## Avoid copying token ids into a new array in `getOwnedTokenIdsOfQuest` function of `RabbitHoleReceipt` contract

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109-L135

This function will first construct an array with the token ids that correspond to the given quest id. Because the actual size of these tokens is unknown in the first place, the function will copy the token ids in a new array of the proper size. This isn't needed, as the array's length can be modified in place using assembly, and save gas by avoiding the creation and iteration of a new array.

```solidity
function getOwnedTokenIdsOfQuest(
    string memory questId_,
    address claimingAddress_
) public view returns (uint[] memory) {
    uint msgSenderBalance = balanceOf(claimingAddress_);
    uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
    uint foundTokens = 0;

    for (uint i = 0; i < msgSenderBalance; i++) {
        uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
        if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
            tokenIdsForQuest[foundTokens++] = tokenId;
        }
    }
    
    // Override length
    assembly {
      mstore(tokenIdsForQuest, foundTokens)
    }
    
    return tokenIdsForQuest;
}
```
