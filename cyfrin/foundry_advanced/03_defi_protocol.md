# Section 03. Develop a DeFi Protocol

## DeFi Introduction

Basic walkthrough of DeFi, main apps (AAVE, Uniswap, MakerDAO) and MEV.

## Project Walkthrough

## Introduction to Stablecoins

## DecentralizedStableCoin.sol

```bash
mkdir defi-stablecoin
cd defi-stablecoin
forge init
touch README.md # https://youtu.be/wUjYK5gwNZs?t=2550
touch src/DecentralizedStableCoin.sol
```

Start coding the DecentralizedStableCoin.sol contract
https://youtu.be/wUjYK5gwNZs?t=2886

## Project Setup: DSCEngine

https://youtu.be/wUjYK5gwNZs?t=3293

## Create the deposit collateral function

## Creating the mint function

## Creating and retrieving the health factor

## Finish the mint function

## Creating the deployment script

## Test the DSCEngine Smart Contract

## Create the depositAndMint function

## Create the redeem collateral function

## Setup liquidations

## Refactor liquidations

## DSCEngine advanced testing

## Create fuzz tests handler

## DeFi Handler Deposit Collateral

## Create the collateral redeem handler

## Create the mint handler

## Debugging the fuzz tests handler

## Create the price feed handler

## Manage your oracles connections

## Links

- [Leveraged Trading in DeFi | Python, Brownie, Aave, Uni/SushiSwap](https://www.youtube.com/watch?v=TmNGAvI-RUA)
- [Become a DEFI QUANT | Python, Chainlink, and Aave](https://www.youtube.com/watch?v=x0YDcZly_PU)
- [Aave Flash Loan Tutorial - Finding Arbitrage](https://www.youtube.com/watch?v=Aw7yvGFtOvI)
- [Flashbots - New to MEV?](https://docs.flashbots.net/new-to-mev)
