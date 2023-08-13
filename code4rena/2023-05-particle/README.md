# Particle Protocol Audit - code4rena

Contest: https://code4rena.com/contests/2023-05-particle-protocol-invitational

## High

- [H-01 Particle Exchange can be used to swap NFTs within the collection for free](./H-01.md)
- [H-02 Borrowers can still close loan normally while being defaulted](./H-02.md)

## Medium

- [M-01 Treasury fee is not collected in `withdrawEthWithInterest()`](./M-01.md)
- [M-02 Borrower can block being defaulted or auctioned](./M-02.md)
- [M-03 Gas limited ETH transfers can lead to a denial of service](./M-03.md)
- [M-04 Function `buyNftFromMarket()` should not be payable](./M-04.md)
- [M-05 Unspent WETH is not considered in `buyNftFromMarket()`](./M-05.md)
- [M-06 Lender can front-run calls to `auctionBuyNft()` to DoS auctions](./M-06.md)
- [M-07 Lender can auction the loan without any restriction to cause losses to the borrower](./M-07.md)
- [M-08 Risk of accidental DoS while receiving NFTs from marketplaces](./M-08.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
