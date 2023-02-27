## State is stored by duplicate in `Escher` contract

The `Escher` contract is an ERC1155 NFT that represents roles (admin, creator, curator) as soulbound NFTs. However, this contract also inherits from OZ's `AccessControl` which also tracks roles using storage. When an account gets a role, it is minted an NFT and also granted the role using the `AccessControl` mechanism.

This means that state is duplicated, each account that has a role in the Escher contract will have a representation as an NFT (`_balances` mapping in OZ ERC1155) and as a role (`_roles` mapping in OZ AccessControl).

One of these is unneeded. For example, this can be implemented by using just the ERC1155 representation. The `IAccessControl` interface could be implemented by minting the NFT to grant a role, burning the NFT to revoke a role, and checking the NFT balance to implement the "has role" behavior. 

## Unneeded `sale` variable constructor initialization in Sales contract

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L46
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L53
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L45

Each of the three sales contracts (`FixedPrice`, `LPDA` and `OpenEdition`) will initialize the sale variable in the constructor. This is not needed as these contracts are used with cloned implementations (rest of the body of the constructor is ok).

## `sale` variable is re-read from storage when ending the sale in the `FixedPrice` contract

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L73

The `sale` variable is re-read from storage (previously read at line 58) when the sale is ended during the buy process.

## Optimize struct fields in `LPDA.Sale`

Timestamps (`startTime` and `endTime`) can be defined as `uint48` to save one storage slot. Using `uint48` would allow a precision up to the year 8921556.

```solidity
struct Sale {
    // slot 1
    uint48 currentId;
    uint48 finalId;
    address edition;
    // slot 2
    uint80 startPrice;
    uint80 finalPrice;
    uint80 dropPerSecond;
    // slot 3
    uint48 startTime;
    uint48 endTime;
    address payable saleReceiver;
}
```

## Optimize struct fields in `OpenEdition.Sale`

Similarly, timestamps here (`startTime` and `endTime`) can be defined as `uint48` to save one storage slot. Using `uint48` would allow a precision up to the year 8921556.

```solidity
struct Sale {
    // slot 1
    uint72 price;
    uint24 currentId;
    address edition;
    // slot 2
    uint48 startTime;
    uint48 endTime;
    address payable saleReceiver;
}
```
