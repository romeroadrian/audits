## Protocol

AfEth is a new protocol that is part of the Asymmetry Finance ecosystem. This new product is essentially a vault, collateralized by two strategies, SafEth and Votium.

SafEth is an existing product of Asymmetry Finance. It provides a diversified Ethereum staking through difference LSD (Liquid Staking Derivatives) protocols. Supported derivatives are:

- Lido Finance
- Rocket Pool
- Frax Finance
- Swell Network
- ANKR
- staFI

The Votium strategy is a new strategy introduced in AfEth. This contract holds Convex tokens (CVX) and delegates them to the Votium protocol, which provides rewards coming from incentives to direct voting power to certain Curve Gauges.

Deposits in AfEth are split according to a configurable ratio. Users deposit ETH and the protocol will stake a percentage of it in SafEth, and deposit the rest in the Votium strategy.

## Contracts

There are 4 core contracts:

- AfEth.sol: main contract and main entrypoint for the AfEth protocol.
- AbstractStrategy.sol: base abstract contract for creating strategies.
- VotiumStrategyCore.sol: implements the core interactions with Votium and Convex.
- VotiumStrategy.sol: strategy implementation to handle deposit, withdrawals and rewards in Votium.

The main interactions are:

- Deposit: users deposit ETH into the protocol and get AfEth tokens.
- Request Withdraw: holders of AfEth can request a withdrawal to remove their funds. This action needs to undergo a waiting period in the underlying assets can be withdrawn from the protocols beneath AfEth. 
- Withdraw: after waiting until the release time, users can execute the queued withdrawal and get back the ETH.
- Rewards: the rewarded can claim rewards from Votium and Convex and deposit them back into the protocol, compounding the assets held by the contract and increasing value for holders.

The VotiumStrategy contract can also act as a vault on its own, meaning users can deposit and interact with it directly, not necessarily going through AfEth. 

All these contracts are upgradeable and follow the Transparent Proxy pattern.

![class-diagram](https://i.ibb.co/d0n5jXM/class-Diagram.png)

# Actors

- Owner: controlled by the DAO multisig. Has access to modify system configuration. Can withdraw stuck tokens from the Votium strategy.
- Rewarder: privileged role that claims rewards from Votium and Convex, swaps them to ETH and compounds the assets back into the protocol. This role wil probably be managed by a Gelato bot.
- End User: deposits ETH into AfEth, can withdraw by following a queue process and recover ETH from their position.

# Flows

The are 3 main flows in the system that govern most of the interactions in the contracts:

- Deposit
- Withdraw
- Rewards

## Deposit

![deposit](https://i.ibb.co/1G98Fww/deposit.png)

## Withdraw

![withdraw](https://i.ibb.co/GRhbTnx/withdraw.png)

## Reward

![reward](https://i.ibb.co/QXrnpHP/reward.png)

# Risks

- Oracle: the system heavily relies on oracles to determine prices which are used in deposits and withdrawals. Invalid or stale data here can be critical to the protocol.
- Swaps and MEV: the Votium strategy depends on exchanging tokens to manage deposits, withdrawals and rewards. Improper handling of swap operations can cause damage to end users or the protocol in general.
- Integrations: there are multiple integrations in the design of the protocol. SafEth on multiple LSD protocols. Chainlink for price feeds. The Votium strategy depends on Convex, Votium and Curve Pools. Any unexpected precondition or misunderstood postcondition while interfacing with any third-party function can break the integration, cause a denial of service or end in unexpected results for the protocol.
