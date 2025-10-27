# Section 01. Review

## Tooling Prerequisites

- VSCode or [VSCodium](https://vscodium.com/): VSCode without the Microsoft telemetry.
- Foundry
- wsl

## Solidity Prerequisites

- Be familiar with compiling and deploying to local and testnet blockchains.
- Familiar with Foundry's working tree and basic commands.
- Familiarty with regular and fuzz testing
- Stateful Fuzzing/Invariant Tests (covered in next lesson)

## Fuzzing and invariant tests

Often, hacks result from scenarios you didn't anticipate or consider for testing. But what if you could write a test that checks for every possible scenario, not just one? This is known as **fuzz testing**.

An invariant is a property of the system that should always hold.

The Fundamental Principle: Testing Invariants

Each system, from a function to an entire program, has an integral property, often referred to as the _invariant_. This property must always hold true.

### Introducing Fuzz Tests and Invariant Tests

There are two popular methodologies when dealing with edge cases: using _fuzz tests/invariant tests_, or _symbolic execution_/_formal verification_ (which we'll save for another day).

Foundry will automatically randomize data and use numerous examples to run through the test script. This test will be supplied random data from 0 to uint256.max(), as many times as you've conifigured runs.

> Reminder: You can configure the number of runs in your foundry.toml under the \[fuzz] variable

The process should be:

1. Understand which are the invariants of the system
2. Write a fuzz test for those invariants

Stateless Fuzzing versus Stateful Fuzzing

_Stateful fuzzing_, instead of resetting the contract state for each new run, will use the ending state of your previous run as the starting state of your next.

- **Stateful fuzzing**: Fuzzing where the final state of your previous run is the starting state of your next run.

Try to understand how to set an invariant test in Foundry.

### Definitions

Again, in the context of Foundry:

- Fuzzing or Fuzz testing = Giving random data to one function
- Invariant testing = Giving random data & random function calls to many functions (also called stateful fuzzing)
- Symbolic execution and formal verification are the same.

## Installing Libraries

To install our OpenZeppelin libraries:

`forge install OpenZeppelin/openzeppelin-contracts --no-commit`

Add to our `foundry.toml`:

`remappings = ['@openzeppelin/contracts=lib/openzeppelin-contracts/contracts']`

## Advanced Solidity Prerequisites

The Core of Smart Contracts: Storage

When you create a state variable within your contract, an individual storage slot is assigned.

Constants or immutable variables do not occupy space in storage. This unique trait is due to their nature of being stored directly within the contract's bytecode.

`forge inspect Counter storage`

## Storage

- Each slot is 32 bytes long and is shown in its hex representation
- Constants or immutable variables don't occupy space in `Storage`, they're stored directly in the contract's bytecode.

Think of `Storage` as a huge array or list that contains all the variables we create in Solidity.

### Dynamic variables

Variables in the form of dynamic arrays or mapping are stored using some type of hashing function. The object itself does take up a storage slot, but it doesn't contain the whole array. Instead, the storage slot contains the length of the array.

### Temporary variables in function scope

For variables that are declared inside a function, their existence is ephemeral and scoped merely to the span of that function. These variables do not persist inside the contract and are not stored in `Storage`. Instead, they're stashed in a different memory data structure, which deletes them as soon as the function has finished execution.

### Memory Keyword

Finally, the `memory` keyword. Primarily used with strings, `memory` is needed because strings are dynamically sized arrays. By using this keyword, we tell Solidity that string operations are to be performed not in `Storage`, but in a separate memory location.

## Fallback and Receive

These two specific functions - `fallback` and `receive` - enable a contract to accept and react to native ETH sent to it. Both these functions can be made "**external payable**", indicating that they can receive and handle ETH.

To put it simply, consider the case of sending ETH to a smart contract without any data. In such an instance, the `receive` function would be called, resorting to `fallback` if the `receive` function does not exist.

On the other hand, if there _is_ data, Solidity will skip straight to the `fallback` function, bypassing the `receive` function entirely.

It is worthwhile to note that the `fallback` function may or may not be payable. If the contract lacks a `receive` function and the `fallback` function isn't payable, then the `fallback` function won't be called when you send ETH to the contract.

A contract that does not contain any of these functions will reject any ETH sent to it. In fact, Solidity will automatically compile this contract to reject ETH - with at least one notable exception we'll go over later.

## Encoding

Understand ABI.encode & ABI.encodePacked, abi.decode(), in Solidity

// TODO: Understand this section better.
// TODO: Also understand the difference between call(), staticcall() and delegatecall().

## Upgradeable contracts

Upgrades, proxies, delegatecalls...

// TODO: Understand this section better.

## Self Destruct

## Fork tests

`forge test --fork-url $MAINNET_ALCHEMY_URL`

## 🛠️ Links and Resources

- [Peera](https://app.peera.ai/)
- [Ethereum Stack Exchange](https://ethereum.stackexchange.com/)
