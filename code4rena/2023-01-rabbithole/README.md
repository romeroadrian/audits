# RabbitHole Quest Protocol Audit - code4rena

Contest: https://code4rena.com/contests/2023-01-rabbithole-quest-protocol-contest

## High

- [H-01 User may loose rewards if the receipt is minted after quest end time](./H-01.md)
- [H-02 Quest owner will receive less funds when withdrawing remaining tokens from the `Erc20Quest` if protocol fees are withdraw first](./H-02.md)
- [H-03 Protocol fees can be withdrawn multiple times in `Erc20Quest`](./H-03.md)
- [H-04 `withdrawRemainingTokens` fails to consider unclaimed receipts in `Erc1155Quest`](./H-04.md)
- [H-05 Changing the `RabbitHoleReceipt` contract in the `QuestFactory` will break rewards for existing quests](./H-05.md)
- [H-06 Bad implementation in minter access control for `RabbitHoleReceipt` and `RabbitHoleTickets` contracts](./H-06.md)

## Medium

- [M-01 Protocol fee recipient address is copied into `Erc20Quest` contract](./M-01.md)
- [M-02 Users can be tricked with claimed receipt tokens - Receipts can be claimed via flash loans](./M-02.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
