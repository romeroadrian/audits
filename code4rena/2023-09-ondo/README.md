# Ondo Finance Audit - code4rena

Contest: https://code4rena.com/contests/2023-09-ondo-finance

## High

- [H-01 Bridged tokens could be lost if sender transfer concurrently from multiple source chains](./H-01.md)

## Medium

- [M-01 Design won't allow bridges with deterministic addresses](./M-01.md)
- [M-02 Approver could accidentally DoS their approval by executing the bridged message](./M-02.md)
- [M-03 Bridged messages can still be approved and executed even if contract is paused](./M-03.md)
- [M-04 Chain support cannot be removed or cleared in bridge contracts](./M-04.md)
- [M-05 Multicall feature in SourceBridge may be abused to send arbitrary messages to the Axelar service](./M-05.md)
- [M-06 Oracle could potentially return an invalid zero price](./M-06.md)
- [M-07 Overriding ranges in oracle may lead to gaps](./M-07.md)
- [M-08 Insufficient precision while deriving price in oracle may lead to innaccurate calculations](./M-08.md)
- [M-09 Rounding issues in rUSDY calculations](./M-09.md)

## Gas

[Gas Report](./gas.md)

## Low/QA

[Low/QA Report](./low.md)
