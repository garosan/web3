# Section 01. Overview

## Intro

Prerequisites:

- Medium understanding of Solidity
- Experience with Foundry
- High school math

Why take the course:

- Competitive edge in audit contests
- Extend knowledge to other AMMs

What will we learn?

- Math of constant product AMMs
- Uniswap V2 contracts
- Testing with Foundry on a mainnet fork

## Looking at a graph

We are going to learn about constant product AMMs and the concept of _liquidity_.

The function to learn is: `x * y = L ^ 2`, where:

- `x` is amount of token X
- `y` is amount of token Y
- `L` is liquidity

Note: Research later why the formula is not `XY = K`

## Contracts

There are 3 important contracts:

- Factory
- Router Contract
- Pair Contract

The factory contract is a simple contract that's used to deploy the pair contracts. We can have a pair contract of ETH/USDT, or DAI/ETH, or DAI/MKR. The factory contract allows us to create these pairs.

We can also interact with the pair contracts directly, to add or remove liquidity or to swap tokens. This is a bit error-prone so we also have the router contract.

The router contract is used as an intermediary to add or remove liquidity from the pairs and also to swap tokens. If we swap DAI to ETH the router will interact with the DAI/ETH pair contract. If we want to swap ETH for MKR the router will swap ETH to DAI, then DAI to MKR, then send the MKR over to us.

The user can interact with the factory contract directly to deploy a pair contract, but the user can also interact with the router, which will then call the factory to deploy the pair contract.

## 🛠️ Links and Resources

- [RareSkills Uniswap V2 Book](https://www.rareskills.io/uniswap-v2-book)
- [Constant Product Automated Market Maker: Everything you need to know](https://medium.com/@tomarpari90/constant-product-automated-market-maker-everything-you-need-to-know-5bfeb0251ef2)
