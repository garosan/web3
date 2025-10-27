# Section 06. Upgradeable Smart Contracts

## Introduction

Upgradable smart contracts are a complex subject with potential drawbacks, which isn't the best route to default on. They sound great in theory, promising flexibility and adaptability, but a lot of problems may arise when there's too much centralized control over contracts.

Remember: The priority should always be minimizing the deployment of upgradeable smart contracts.

From hacks to lost funds, the risks are real.

Let's look at three different patterns or philosophies we can use:

1. Parameterizing (Not really upgrading, just adding a bunch of setter functions)
2. Social migration
3. Proxy (with subcategories like metamorphic contracts, transparent upgradable proxies, and universal upgradable proxies)

### Not really upgrading

The idea here is parameterizing everything, having setter functions that can change certain parameters. For instance, if you have a set reward that distributes a token at a 1% rate every year, you can have a setter function to adjust that distribution rate. The obvious drawback is, unless you anticipated all possible future functionality when writing the contract, you won't be able to add it in the future.

Another question that arises is—who gets access to these functions? If a single person holds the key, it becomes a centralized smart contract, going against decentralization's core principle. To address this, you can add a governance contract to your protocol, allowing proportional control.

### Social Migration

Basically like being AAVE and saying 'hey everyone we just launched V2, go here to interact with V2!'. It's a social convention that everyone agrees to use the new contract.

Drawbacks:

- It has a new address so you have to tell everyone to use the new contract (including exchanges and other platforms that reference your contract)
- You have to move all your state to the new contract (i.e. if you're a ERC-20, you need to move all the current balances to the new contract)

Moving the state of the first contract to the second one is also a challenging task. You need to devise a migration method to transport the storage from one contract to the other. You can learn more about the social migration method from [this blog post](https://blog.trailofbits.com/2018/09/05/contract-upgrade-anti-patterns/) written by Trail of Bits.

### Proxies

Proxies are the real way to make upgradeable contracts. The idea is that you deploy a proxy and an implementation contract. Users interact with the proxy which calls the implementation contract. If you the dev want, you can change the implementation without users
ever realizing they're interacting with a new version of the contract.

There are 3 main proxy methodologies:

- Transparent Proxies
- Universal Upgradable Proxies (UUPS)
- The Diamond Pattern

Each has its benefits and drawbacks, but the focus is on maintaining contract functionality and decentralization.

## Using delegatecall

In this section we're going to be learning how to build our own proxies and in order to do this we first need an understanding of the delegateCall function.

## Overview of EIP-1967

In this lesson we'll apply what we've learnt and get our hands dirty with a small proxy example. The code for this exercise can be found **[here](https://github.com/Cyfrin/foundry-upgrades-f23/tree/main/src/sublesson)**.

## ❓ Questions and 💪 Exercises

## 🛠️ Links and Resources

- [Trail of Bits - Contract upgrade anti-patterns](https://blog.trailofbits.com/2018/09/05/contract-upgrade-anti-patterns/)
- [Yul Docs](https://docs.soliditylang.org/en/latest/yul.html)
