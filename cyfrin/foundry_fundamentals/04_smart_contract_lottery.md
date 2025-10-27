# Section 4. Smart Contract Lottery

- [Github Repo](https://github.com/Cyfrin/foundry-smart-contract-lottery-cu)

We will learn:

- Events
- On-chain randomness
- Modulo
- Chainlink automation
- Using natspec

## Setup

```shell
mkdir foundry-smart-contract-lottery
cd foundry-smart-contract-lottery

forge init
```

Delete all the `Counter` files.

Let's start with adding the details of this project to the `README.md`:

```markdown
# Proveably Random Raffle Contracts

## About

This code is to create a proveably random smart contract lottery.

## What we want it to do?

1. Users should be able to enter the raffle by paying for a ticket. The ticket fees are going to be the prize the winner receives.
2. The lottery should automatically and programmatically draw a winner after a certain period.
3. Chainlink VRF should generate a provably random number.
4. Chainlink Automation should trigger the lottery draw regularly.
```

Let's now create a file named `Raffle.sol`:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

contract Raffle {


}
```

Let's create some NatSpec for our contract, if you have the Nomic Foundation VSCode extension, you can trigger the code by doing `/** + tab`.

Let's now think for a moment, what does our Raffle need to work? We need 2 things:

- A function to enter the raffle by paying a ticket price
- A function to pick a winner out of the registered users

```solidity
contract Raffle{
    function enterRaffle() public {}

    function pickWinner() public {}

}
```

Let's make our fee immutable and add a getter function to get the fee to enter.
Our contract should look like this:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

/**
 * @title A simple Raffle contract
 * @author Garo Sanchez
 * @notice This contract creates a simple raffle
 * @dev This contract implements Chainlink VRF v2.5 and Chainlink Automation
 */
contract Raffle {
    uint256 private immutable i_entranceFee;

    constructor(uint256 entranceFee) {
        i_entranceFee = entranceFee;
    }

    function enterRaffle() public payable {}

    function pickWinner() public {}

    /** Getter Function */

    function getEntranceFee() external view returns (uint256) {
        return i_entranceFee;
    }
}
```

## Solidity Style Guide

The Solidity Docs provide an [Order of Layout](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout) that looks like this:

```
// Layout of the contract file:
// version
// imports
// errors
// interfaces, libraries, contract

// Inside Contract:
// Type declarations
// State variables
// Events
// Modifiers
// Functions

// Layout of Functions:
// constructor
// receive function (if exists)
// fallback function (if exists)
// external
// public
// internal
// private

// view & pure functions
```

## Creating custom errors

Previously we defined the `i_entranceFee` variable. This is the amount the user has to send to enter the raffle. How do we check this?

```solidity
function enterRaffle() external payable {
    require(msg.value >= i_entranceFee, "Not enough ETH sent");

}
```

Custom errors were introduced in Solidity v0.8.4 to provide a more gas-efficient way to explain to users why an operation failed.

- [More information on Custom Errors](https://soliditylang.org/blog/2021/04/21/custom-errors/)

Our custom error would look like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

/**
 * @title A sample Raffle contract
 * @author Garo Sanchez
 * @notice This contract is for creating a sample raffle
 * @dev Implements Chainlink VRFv2.5
 */

contract Raffle {
    /*Errors */
    error Raffle__NotEnoughEthToEnterRaffle();
    uint256 private immutable i_entranceFee;

    constructor(uint256 entranceFee) {
        i_entranceFee = entranceFee;
    }

    function enterRaffle() public payable {
        // Old way using a require statement:
        // require(msg.value >= i_entranceFee, "Not enough ETH sent!");
        // New way using custom errors:
        if (msg.value < i_entranceFee) {
            revert Raffle__NotEnoughEthToEnterRaffle();
        }
    }

    function pickWinner() public {}

    /** Getter Functions */
    function getEntranceFee() external view returns (uint256) {
        return i_entranceFee;
    }
}
```

2 things:

1. Using an if statement combined with a custom error is the most efficient way to do this. We could use a custom error inside a require statement (supported as of 0.8.26) but it is not as gas efficient and has several caveats.
2. In the docs, the custom error is outside the contract. Patricks says we will put the custom error definition inside the contract to avoid testing headaches, why?

Notice it is a best practice to prefix your custom error with the name of the contract: `Raffle__NotEnoughEthToEnterRaffle()`

## Smart Contract Events

Now we need a storage structure to keep track of all registered users in the lottery.

We will use a dynamic array that is payable:

`address payable[] private s_players;`

We've made it `address payable` because one of the participants registered in this array will be paid the ETH prize, hence the need for the `payable` attribute.

Now in the `enterRaffle()` function, we need to push the address of the entered user into the array:

```solidity
function enterRaffle() public payable {
    // Old way using a require statement:
    // require(msg.value >= i_entranceFee, "Not enough ETH sent!");
    // New way using custom errors:
    if (msg.value < i_entranceFee) {
        revert Raffle__NotEnoughEthToEnterRaffle();
    }
    s_players.push(payable(msg.sender));
}
```

It would be good to create a little notification that a user has entered the raffle. Enter events.

### Events

Events are a way for smart contracts to communicate with the outside world, primarily with the front-end applications that interact with these contracts. Events are logs that the Ethereum Virtual Machine (EVM) stores in a special data structure known as the blockchain log, and are basically a very efficient way to track and retrieve information quickly.

- [Solidity Docs- Events](https://docs.soliditylang.org/en/v0.8.25/contracts.html#events)

An event has parameters and sometimes they have the keyword `indexed`. Indexed parameters, also called `topics`, are much more easy to search than non-indexed ones.

Finally, for an event to be logged we need to emit it.

This is the code to define an event: `event EnteredRaffle(address indexed player);`

And the code to emit it:

```solidity
function enterRaffle() external payable {
    if(msg.value < i_entranceFee) revert Raffle__NotEnoughEthSent();
    s_players.push(payable(msg.sender));
    emit EnteredRaffle(msg.sender);

}
```

This is how our code should be looking so far:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

/**
 * @title A sample Raffle contract
 * @author Garo Sanchez
 * @notice This contract is for creating a sample raffle
 * @dev Implements Chainlink VRFv2.5
 */

contract Raffle {
    /* Errors */
    error Raffle__NotEnoughEthToEnterRaffle();

    uint256 private immutable i_entranceFee;
    address payable[] private s_players;

    /* Events */
    event EnteredRaffle(address indexed player);

    constructor(uint256 entranceFee) {
        i_entranceFee = entranceFee;
    }

    function enterRaffle() public payable {
        // Old way using a require statement:
        // require(msg.value >= i_entranceFee, "Not enough ETH sent!");
        // New way using custom errors:
        if (msg.value < i_entranceFee) {
            revert Raffle__NotEnoughEthToEnterRaffle();
        }
        s_players.push(payable(msg.sender));
        emit EnteredRaffle(msg.sender);
    }

    function pickWinner() public {}

    /** Getter Functions */
    function getEntranceFee() external view returns (uint256) {
        return i_entranceFee;
    }
}
```

## Random Numbers - Block Timestamp

First, let's change our functions that were `public` to `external` to make them more gas efficient.

Now we want to define an interval for our lottery, we want to set a specific time that we want our lottery to last (like PancakeSwap's lottery that runs every 5 minutes).

So let's create this immutable variable:

`uint256 private immutable i_interval;`

And also add it to the constructor. You know how to do this.

Now we would use `block.timestamp` to revert the function `pickWinner()` if not enough time has passed. This is how our code should look like so far:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

/**
 * @title A sample Raffle contract
 * @author Garo Sanchez
 * @notice This contract is for creating a sample raffle
 * @dev Implements Chainlink VRFv2.5
 */

contract Raffle {
    /* Errors */
    error Raffle__NotEnoughEthToEnterRaffle();

    uint256 private immutable i_entranceFee;
    uint256 private immutable i_interval;
    address payable[] private s_players;
    uint256 private s_lastTimestamp;

    /* Events */
    event EnteredRaffle(address indexed player);

    constructor(uint256 entranceFee, uint256 interval) {
        i_entranceFee = entranceFee;
        i_interval = interval;
        s_lastTimestamp = block.timestamp;
    }

    function enterRaffle() external payable {
        // Old way using a require statement:
        // require(msg.value >= i_entranceFee, "Not enough ETH sent!");
        // New way using custom errors:
        if (msg.value < i_entranceFee) {
            revert Raffle__NotEnoughEthToEnterRaffle();
        }
        s_players.push(payable(msg.sender));
        emit EnteredRaffle(msg.sender);
    }

    function pickWinner() external {
        // See if enough time has passed
        if (block.timestamp - s_lastTimestamp < i_interval) {
            revert();
        }
    }

    /** Getter Functions */
    function getEntranceFee() external view returns (uint256) {
        return i_entranceFee;
    }
}
```

## Random Numbers - VRF

To start reading about VRF, check the official [Chainlink docs](https://docs.chain.link/vrf) about VRF.

So, how does Chainlink VRF work?

Chainlink VRF provides randomness in 3 steps:

1. Requesting Randomness: A smart contract makes a request for randomness by calling the `requestRandomness` function provided by the Chainlink VRF. This involves sending a request to the Chainlink oracle along with the necessary fees.

2. Generating Randomness: The Chainlink oracle node generates a random number off-chain using a secure cryptographic method. The oracle also generates a proof that this number was generated in a verifiable manner.

3. Returning the Result: The oracle returns the random number along with the cryptographic proof to the smart contract. The smart contract can then use the random number, and any external observer can verify the proof to confirm the authenticity and integrity of the randomness.

Let's implement VRF following these steps:

1. Go to the Chainlink Faucet and get some test LINK.

2. Go to the [subscription manager](https://vrf.chain.link/) and click 'Create Subscription'.

3. In the list of your subscriptions, click on the subscription > 'Fund subscription'.

4. Next click on 'Add Consumer'. Our smart contract and Chainlink VRF need to be aware of each other. So this page will give you a 'Subscription ID' that your contract will consume, and requests you the address of the contract that will use it.

5. Go to [this page](https://docs.chain.link/vrf/v2/subscription/examples/get-a-random-number#create-and-deploy-a-vrf-v2-compatible-contract) and check what it says.

First, the example code imports 3 dependencies:

- `VRFConsumerBaseV2.sol`
- `VRFCoordinatorV2Interface.sol`
- `ConfirmedOwner.sol`

This contract also includes hard-coded request parameters such as `vrfCoordinator` address, gas lane `keyHash`, `callbackGasLimit`, `requestConfirmations` and number of random words `numWords`.

- In [this page](https://docs.chain.link/vrf/v2-5/supported-networks) you will find all the configuration parameters that you may need for your app.

The rest of the video is a detailed walkthrough of the code in the example contract, we will skip that for now.

// TODO: Come back and take a look at the detailed walkthrough and add notes.

## Implementing VRF in our code

Getting a random number is a 2 step process:

1. Request the random number generator (RNG)
2. Get the actual random number

It's important to note that [this](<(https://remix.ethereum.org/#url=https://docs.chain.link/samples/VRF/v2-5/VRFD20.sol&autoCompile=true)>) is the most up to date implementation of the example code, so let's open that in Remix and bring in this piece of code:

```solidity
requestId = s_vrfCoordinator.requestRandomWords(
    VRFV2PlusClient.RandomWordsRequest({
        keyHash: s_keyHash,
        subId: s_subscriptionId,
        requestConfirmations: requestConfirmations,
        callbackGasLimit: callbackGasLimit,
        numWords: numWords,
        extraArgs: VRFV2PlusClient._argsToBytes(
            // Set nativePayment to true to pay for VRF requests with Sepolia ETH instead of LINK
            VRFV2PlusClient.ExtraArgsV1({nativePayment: false})
        )
    })
);
```

We will go step by step to make this code work in our contract.

## Implementing VRF in our code

Let's first install Chainlink:

`forge install smartcontractkit/chainlink-brownie-contracts --no-commit`

Then update the `foundry.toml` file to include remappings:

```toml
remappings = [
  '@chainlink/contracts/=lib/chainlink-brownie-contracts/contracts/',
]
```

And now we can do our imports like this:

```solidity
import {VRFConsumerBaseV2Plus} from "@chainlink/contracts/src/v0.8/vrf/dev/VRFConsumerBaseV2Plus.sol";
import {VRFV2PlusClient} from "@chainlink/contracts/src/v0.8/vrf/dev/libraries/VRFV2PlusClient.sol";
```

Now let's inherit `VRFConsumerBaseV2Plus`.

Ok so basically we created all the immutable and constant variables needed, passed them in the constructor and plugged them in our function. Oh we also had to import the randomWords function at the end so we could complete the compilation.

Our code should look like this now:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {VRFConsumerBaseV2Plus} from "@chainlink/contracts/src/v0.8/vrf/dev/VRFConsumerBaseV2Plus.sol";
import {VRFV2PlusClient} from "@chainlink/contracts/src/v0.8/vrf/dev/libraries/VRFV2PlusClient.sol";
import {VRFV2PlusClient} from "@chainlink/contracts/src/v0.8/vrf/dev/libraries/VRFV2PlusClient.sol";

/**
 * @title A sample Raffle contract
 * @author Garo Sanchez
 * @notice This contract is for creating a sample raffle
 * @dev Implements Chainlink VRFv2.5
 */

contract Raffle is VRFConsumerBaseV2Plus {
    /* Errors */
    error Raffle__NotEnoughEthToEnterRaffle();

    uint16 private constant REQUEST_CONFIRMATIONS = 3;
    uint32 private constant NUM_WORDS = 1;

    uint256 private immutable i_entranceFee;
    uint256 private immutable i_interval;
    bytes32 private immutable i_keyHash;
    uint32 private immutable i_callbackGasLimit;
    uint256 private immutable i_subscriptionId;
    address payable[] private s_players;
    uint256 private s_lastTimestamp;

    /* Events */
    event EnteredRaffle(address indexed player);

    constructor(
        uint256 entranceFee,
        uint256 interval,
        address vrfCoordinator,
        bytes32 gasLane,
        uint256 subscriptionId,
        uint32 callbackGasLimit
    ) VRFConsumerBaseV2Plus(vrfCoordinator) {
        i_entranceFee = entranceFee;
        i_interval = interval;
        i_keyHash = gasLane;
        i_subscriptionId = subscriptionId;
        s_lastTimestamp = block.timestamp;
        i_callbackGasLimit = callbackGasLimit;
    }

    function enterRaffle() external payable {
        // Old way using a require statement:
        // require(msg.value >= i_entranceFee, "Not enough ETH sent!");
        // New way using custom errors:
        if (msg.value < i_entranceFee) {
            revert Raffle__NotEnoughEthToEnterRaffle();
        }
        s_players.push(payable(msg.sender));
        emit EnteredRaffle(msg.sender);
    }

    function pickWinner() external {
        // See if enough time has passed
        if (block.timestamp - s_lastTimestamp < i_interval) {
            revert();
        }

        uint256 requestId = s_vrfCoordinator.requestRandomWords(
            VRFV2PlusClient.RandomWordsRequest({
                keyHash: i_keyHash,
                subId: i_subscriptionId,
                requestConfirmations: REQUEST_CONFIRMATIONS,
                callbackGasLimit: i_callbackGasLimit,
                numWords: NUM_WORDS,
                extraArgs: VRFV2PlusClient._argsToBytes(
                    // Set nativePayment to true to pay for VRF requests with Sepolia ETH instead of LINK
                    VRFV2PlusClient.ExtraArgsV1({nativePayment: false})
                )
            })
        );
    }

    function fulfillRandomWords(
        uint256 requestId,
        uint256[] calldata randomWords
    ) internal override {}

    /** Getter Functions */
    function getEntranceFee() external view returns (uint256) {
        return i_entranceFee;
    }
}
```

## CEI - Checks, Effects, Interactions

The Checks-Effects-Interactions pattern is a crucial best practice in Solidity development aimed at enhancing the security of smart contracts, especially against reentrancy attacks. This pattern structures the code within a function into three distinct phases:

- Checks: Validate inputs and conditions to ensure the function can execute safely. This includes checking permissions, input validity, and contract state prerequisites.
- Effects: Modify the state of our contract based on the validated inputs. This phase ensures that all internal state changes occur before any external interactions.
- Interactions: Perform external calls to other contracts or accounts. This is the last step to prevent reentrancy attacks, where an external call could potentially call back into the original function before it completes, leading to unexpected behavior. (More about reentrancy attacks on a later date)

https://updraft.cyfrin.io/courses/foundry/smart-contract-lottery/solidity-random-number-chainlink-vrf?lesson_format=transcript
