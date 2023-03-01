## Move `settings` assignment below checks in VRFNFTRandomDraw initializer

The `settings` variable assignment in line 80 can be moved below the checks to save gas in case something fails.

## Duplicated check in `startDraw` function

There's a check in lines 175-177 of the `VRFNFTRandomDraw` contract that is checked again in the `_requestRoll` function (lines 144-146).

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L175-L177

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L144-L146

## Use `msg.sender` instead of reading from storage in `lastResortTimelockOwnerClaimNFT`

`msg.sender` can be safely used in line 317 instead of reading from storage using the `owner()` accessor since the function implements the `onlyOwner` modifier.

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L317

## Optimize `CurrentRequest` struct

The `drawTimelock` field can be changed to `uint248` to accommodate it with the `hasChosenRandomNumber` (boolean, 8 bits long) in a single slot. This will reduce the required storage from 4 slots to 3. 

`uint248` precision can still represent timestamps until the year 14333223582026121799511060516560253983079097043344314535799629431672.

## Optimize `Settings` struct

The `subscriptionId` field can be moved to be next to the `token` variable (64 bits + 160 bits = 224 bits) to save one storage slot.

`drawBufferTime` is a duration that is at most a month (line 86 in VRFNFTRandomDraw), this means that a `uint32` is enough to represent this value. `recoverTimelock` precision can also be lowered safely to `uint64`, which can represent dates until the year 584554051223. These two fields can now be packed next to the `drawingToken` address (160 + 32 + 64 = 256) to save another 2 storage slots.


```solidity
struct Settings {
    /// @notice Token Contract to put up for raffle
    address token;
    /// @notice Chainlink subscription id
    uint64 subscriptionId;
    /// @notice Token ID to put up for raffle
    uint256 tokenId;
    /// @notice Token that each (sequential) ID has a entry in the raffle.
    address drawingToken;
    /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
    uint32 drawBufferTime;
    /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
    uint64 recoverTimelock;
    /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
    uint256 drawingTokenStartId;
    /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
    uint256 drawingTokenEndId;
    /// @notice Chainlink gas keyhash
    bytes32 keyHash;
}
```

## Avoid NFT owner check in `VRFNFTRandomDraw` initializer

When a new raffle is created, the initializer function will check that the admin account (caller to the factory) is the owner of the NFT being given away. This doesn't provide any warranty as the NFT can be borrowed, can later be transferred/sold, etc. Consider removing this check, as the `startDraw` function will properly transfer the NFT and will fail if the owner doesn't have it.
