## Use internal calls to `add` in `mint` function of `CidNFT`

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L147

For each item in the `_addList` array the `mint` function is executing a delegatecall to a function in the same contract, which creates a new context and copies the arguments. Change this to an internal call in order to save gas, which will simply execute a jump within the contract and can benefit from reading parameters from calldata instead of copying each element in the array.

## Extract functionality out of the `remove` function to prevent duplicate execution

The `add` function present in the `CidNFT` contract will call the `remove` function in case there's an already associated token for the given params. The `remove` function will fetch data and run the checks again (fetch subprotocol data, check subprotocol data, check CID NFT ownership), wasting a lot of gas in duplicated execution.

Extract functionality out of the `remove` function into an internal function so the `add` function can call this internal function without fetching data or running the checks again.

## Use `push` instead of copying memory array when associating an `ACTIVE` subprotocol NFT

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L215-L220

The code will check if the current list of active NFTs is empty `lengthBeforeAddition == 0` and create an array in memory with a single element and copy it to storage. There's no need to do this, as a simple push to storage can handle the case, while improving gas costs.

```solidity
if (lengthBeforeAddition == 0) {
    activeData.values.push(_nftIDToAdd);
    activeData.positions[_nftIDToAdd] = 1; // Array index + 1
} else {
```
