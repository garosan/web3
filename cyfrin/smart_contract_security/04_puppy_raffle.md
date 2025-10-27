# Section 04. Puppy Raffle

## Introduction

In this audit we will:

- Practice some manual review
- Use tools for static analysis

We'll see the differences between a private audit report and a competitive audit submission and be introduced to the process of competing in a CodeHawks First Flight!

Tools we will be using:

- **[Slither](https://github.com/crytic/slither)** - A pythonic static analysis tool compatible with Solidity and Vyper
- **[Aderyn](https://github.com/Cyfrin/aderyn)** - Built in Rust by _Alex Roan_, Aderyn traverses Abstract Syntax Trees to highlight suspected vulnerabilities.

### Case Studies

We'll dive into real world case studies detailing times these vulnerabilities were exploited in the wild.

At the end of the section, we'll have _even more_ exercises to challenge yourself beyond the course's teachings. These are opportunities to branch out, network and gain additional experience.

This includes participating in a CodeHawks First Flight or a competitive audit! Get ready!

### Prep for Puppy Raffle

If you take a look at the **[repo](https://github.com/Cyfrin/4-puppy-raffle-audit)** associated with this section, you'll see a fairly robust README already supplied. For this review, we're assuming the protocol has already undergone some degree of onboarding and they've provided us a respectable repo.

## Puppy Raffle Primer

1. Do **not** look at the `audit-data` branch of the course **[repo](https://github.com/Cyfrin/4-puppy-raffle-audit)**. This is our `answer key`.

2. Take some time to scope the codebase yourself before proceeding. Try to go through the process we just did with PasswordStore and challenge yourself to find what you can here.

## Phase 1: Scoping

Take a look at the **[Puppy Raffle Repo](https://github.com/Cyfrin/4-puppy-raffle-audit)**'s README.

What we're looking for:

- About section to tell us what's the contract or set of contracts' purpose
- Setup (If there's a makefile run `make` 1st unless otherwise instructed)
- Scope
- Compatibilities (This contract is solidity v 0.7.x, hmm strange)
- Roles
- Known issues

> Take a brief look at your `Makefile`. It's worthwhile to appreciate what it's actually doing. Our `Makefile` cleans our repo, installs necessary packages (Foundry, OpenZeppelin and base64) and then runs `forge build` to compile everything.

### Testing

Once we've run our `make` command, we should check out the protocol tests. I like to start by running `forge coverage` to see what kind of baseline we're starting with.

### Continuing...

By using the command `git checkout <commitHash>` we can assure our local repo is the correct version to be auditing.

### Roles

- Owner - Deployer of the protocol, has the power to change the wallet address to which fees are sent through the changeFeeAddress function.
- Player - Participant of the raffle, has the power to enter the raffle with the enterRaffle function and refund value through refund function.

## Tooling - Slither

https://github.com/crytic/slither

So I had to install slither using a global python environment:

```bash
python3 -m venv ~/slither-venv
source ~/slither-venv/bin/activate
# Install Slither once inside this global venv
pip install slither-analyzer
# Now, in any Foundry project, just activate this global venv before running Slither:
source ~/slither-venv/bin/activate
slither .
# To stop using slither and deactivate the venv
deactivate
```

## Tooling - Aderyn

https://github.com/Cyfrin/aderyn

Created by Alex Roan, Cyfrin's CTO.

You need to install rust for this one.

## Tooling - Solidity Visual Developer

This extension helps you add highlight box to variables to see if they're memory, storage variables, etc. Completely optional and I won't install it.

## Recon - Reading docs

One good tip would be to create your `.notes.md` file (for any project) and add the 'About' section in your own words. For this project this is what we're given:

### Puppy Raffle

This project is to enter a raffle to win a cute dog NFT. The protocol should do the following:

1. Call the `enterRaffle` function with the following parameters:
   1. `address[] participants`: A list of addresses that enter. You can use this to enter yourself multiple times, or yourself and a group of your friends.
2. Duplicate addresses are not allowed
3. Users are allowed to get a refund of their ticket & `value` if they call the `refund` function
4. Every X seconds, the raffle will be able to draw a winner and be minted a random puppy
5. The owner of the protocol will set a feeAddress to take a cut of the `value`, and the rest of the funds will be sent to the winner of the puppy.

## Recon: Reading the code

We can do `forge inspect PuppyRaffle methods`

For now we can see that immutable variables should be prefixed with `i_varName`.

## Recon: Reading the code II

We can start seeing more interesting things.

In `msg.value == entranceFee * newPlayers.length` what is one of them is 0?

There's a very important bug in here, can you spot it?

```solidity
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
}
```

## sc-exploits-minimized

Check out this Cyfrin [repo](https://github.com/Cyfrin/sc-exploits-minimized) which contains minimized examples of some of the bugs that we're going to see.

## Explot: Denial of service

https://www.damnvulnerabledefi.xyz/challenges/unstoppable/

This is the Remix example:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract DoS {
    address[] entrants;

    function enter() public {
        // Check for duplicate entrants
        for (uint256 i; i < entrants.length; i++) {
            if (entrants[i] == msg.sender) {
                revert("You've already entered!");
            }
        }
        entrants.push(msg.sender);
    }
}
```

Let's try to enter this with our 1st account:

For participant 0, execution cost 44633
participant 1, execution cost 30160 // Why you ask? The 1st one to use the contract has to pay more gas to 'initialize' the cct.
participant 2, cost 32787
participant 3, cost 35414

It would be super cool if you could:

1. Clone the repo and run the tests yourself to verify what was said
2. Be able to create your own tests to test this kind of scenarios

## Case Study: DoS

Owen Thurm from Guardian Audits.

unbounded for loops??

### Another finding Global-1 | Should unwrap native token DoS

// TODO: Come back to this part bcs I didnt understand sht.

## DoS PoC

Let's create a PoC to demonstrate this is actually an issue in our PuppyRaffle contract

First lets run forge test to make sure the test suite works.

// TODO: Come back and try to do this myself understanding each bit of the test code.

## DoS: Reporting

Here we're just creating the report. Circle back to this if you need inspiration on how to prepare and write your report.

## DoS: Mitigation

## Exploit: Business logic edge case

A little bug that was found:

```solidity
/// @notice a way to get the index in the array
/// @param player the address of a player in the raffle
/// @return the index of the player in the array, if they are not active, it returns 0
function getActivePlayerIndex(address player) external view returns (uint256) {
    for (uint256 i = 0; i < players.length; i++) {
        if (players[i] == player) {
            return i;
        }
    }
    return 0;
}
```

Can you spot why this is a bug? Patrick says its severity is _informational_ idk about that.

## Recon: Refund

The next bug is in the `refund()` function, see if you can spot it:

```solidity
/// @param playerIndex the index of the player to refund. You can find it externally by calling `getActivePlayerIndex`
/// @dev This function will allow there to be blank spots in the array
function refund(uint256 playerIndex) public {
    address playerAddress = players[playerIndex];
    require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
    require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

    payable(msg.sender).sendValue(entranceFee);

    players[playerIndex] = address(0);
    emit RaffleRefunded(playerAddress);
}
```

It seems to me from the hints that it has something to do with a reentrancy attack. Here CEI pattern could come in handy, maybe its something related to that we should first do `players[playerIndex] = address(0);` and then send the value, but we'll see.
Basically here we're doing it the other way around, CHECKS-INTERACTIONS-EFFECTS...

## Exploit: Reentrancy

An explanation of how a reentrancy attack works.

## Reentrancy: Remix example

PoC of Reentrancy Attack

## Reentrancy: Mitigation

CEI is falling behind as a concept and now there is CEII, FRE-PI.

## Menace to Society

- [Github repo](https://github.com/pcaversaccio/reentrancy-attacks) by Pascal showing all reentrancy attacks up to this date.
