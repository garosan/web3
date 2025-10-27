# Section 03. Fund Me

## Introduction

Our project will be like a Blockchain Kickstarter. We will write 2 contracts `FundMe.sol` and `PriceConverter.sol`, with the following features:

- Users can send any native blockchain currency
- Owner of the contract can withdraw all the funds collected
- A minimum USD amount to donate will be set

## Setup

Let's create a `FundMe.sol` file in Remix and lay out the basic structure:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract FundMe {
    function fund() public {}
    function withdraw() public {}
}
```

## Sending ETH through a function

When a transaction is sent to the blockchain, a value field is always included in the transaction data. For a function to be able to receive Ethereum, it must be declared `payable`:

```solidity
function fund() public payable {
    // allow users to send $
    // have a minimum of $ sent
    // How do we send ETH to this contract?
    msg.value;

    //function withdraw() public {}
}
```

In Solidity, the **value** of a transaction is accessible through the `msg.value` property.

### Reverting transactions with `require`

We can use the`require` keyword as a checker, to enforce our function to receive a minimum `value` of one (1) whole ether:

`require(msg.value > 1e18); // 1e18 = 1 ETH = 1 * 10 ** 18`

The require statement in Solidity can include a custom error message, which is displayed if the condition isn't met, clearly explaining the cause of the transaction failure:

`require(msg.value > 1 ether, "Didn't send enough ETH"); // if the condition is false, revert with the error message`

Notice how we can replace `1 * 10 ** 18` with `1 ether`. You can check all the ether units available in the links.

## Introduction to Oracles

This lesson will walk you through adding a **currency conversion feature** to the `fundMe` contract and setting **price thresholds** with Chainlink Oracle, a decentralized network for external data.

### USD values

We can begin by specifying the new value with a state variable `uint256 public minimumUSD = 5` at the top of the contract.

Next step would be changing the condition inside the `fund` function, to check if the `value` sent is equal to or greater than our `minimumUSD`. But here's the tricky part: the `minimumUSD` value is in USD while the transaction message value is specified in ETH.

### Decentralized oracles

The USD price of assets like Ethereum cannot be derived from blockchain technology alone but it is determined by the financial markets. To obtain a correct _price information_, a connection between off-chain and on-chain data is necessary. This is facilitated by a _decentralized Oracle network_.

This limitation exists because blockchains need to be deterministic in nature.

For smart contracts to effectively replace traditional agreements, they must have the capability to interact with **real-world data**. Relying on a centralized Oracle for data transmission is inadequate as it reintroduces potential failure points.

### How Chainlink works

Chainlink is a _modular and decentralized Oracle Network_ that enables the integration of data and external computation into a smart contract.

These are some of the Chainlink features that we will learn to use:

- Data Feeds
- Verifiable Random Number
- Automation (previously known as 'Keepers')
- Functions

### Chainlink Data Feeds

_Chainlink Data Feeds_ are responsible for powering over $50B in the DeFi world. Chainlink nodes aggregate data from various exchanges and data providers, with each node independently verifying the asset price.

They aggregate this data and deliver it to a reference contract, the **price feed contract**, in a single transaction. Each contract will store the pricing details of a specific cryptocurrency

### Chainlink VRF (Verifiable Random Function)

VRF provides a solution for generating **provably random numbers**, ensuring true fairness in applications such as NFT randomization, lotteries, and gaming. These numbers are determined off-chain, and they are immune to manipulation.

### Chainlink Automation

These **nodes** listen for specific events and, upon being triggered, automatically execute the intended actions within the calling contract.

### Chainlink Functions

Functions allow **API calls** to be made within a decentralized environment. This feature is ideal for creating innovative applications and is recommended for advanced users with a thorough understanding of Chainlink ecosystem.

## Quick Recap

For now, here's an example of how you could use Chainlink to retrieve on-chain the latest price of gold:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";
contract OracleTest {
    AggregatorV3Interface internal priceFeed;
    // The Chainlink price feed contract address
    constructor() {
        priceFeed = AggregatorV3Interface(0x1b44F3514812d835EB1BDB0acB33d3fA3351Ee43);
    }
    // Get the BTC/USD price in Sepolia
    function getLatestBTCPrice() public view returns (int) {
        (,int price,,,) = priceFeed.latestRoundData();
        return price;
    }
}
```

This returns 9626011591048 for me on 17-FEB-2025. We know Chainlink pricefeeds have 8 decimals so this would equal $96,260.1159, nice.

Let's do another example, let's get the price of JPY in USD:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";
contract OracleTest {
    AggregatorV3Interface internal priceFeed;
    // The Chainlink price feed contract address
    constructor() {
        priceFeed = AggregatorV3Interface(0x8A6af2B75F23831ADc973ce6288e5329F63D86c6);
    }
    // Get the JPY/USD price in Sepolia
    function getLatestJPYPrice() public view returns (int) {
        (,int price,,,) = priceFeed.latestRoundData();
        return price;
    }

    // You can double check the amount of decimals
    function getDecimals() public view returns (uint8) {
        return priceFeed.decimals();
    }
}
```

This gives me back 659878 on 17-FEB-2025. If we add 8 decimals we get 0.0066 JPY per 1 USD. Pretty much the same that google give us. Sweet.

## Solidity interfaces

Let's continue writing code for our `FundMe.sol` contract and create these 2 functions:

```solidity
function getPrice() public {}
function getConversionRate() public {}
```

You can see all about Chainlink Data Feeds in [this](https://docs.chain.link/data-feeds) link.

Now to interact with the Chainlink contract that will give us the price data, we will need its address and ABI. You can get the contract address from this page:

- [Price Feed Contract Addresses Page](https://docs.chain.link/data-feeds/price-feeds/addresses?network=ethereum&page=1)

What about the ABI? Well if we think about it, the ABI is basically just a detail of the functions of a contract and their properties, like if the function is payable, external, etc. There are other ways to call the functions of an external smart contract like knowing the function selector, but for now, we will use an _interface_.

Basically, an interface is just code for the functions and their properties, signatures, etc. (like an ABI!) so we will import the `AggregatorV3Interface` from Chainlink and use it.

You can learn a bit more about interfaces [here](https://solidity-by-example.org/interface/).

Also, here are the main differences between an ABI and an Interface:

- An ABI is a JSON file that allows external apps (via Ethers.js, viem, etc.) to call contract functions. It is generated by the Solidity compiler at compile time.
- An Interface is a Solidity file written by a developer to help other developers interact with his contract without having to know all the code and implementations of said contract.

## Import from GitHub

Let's import the `AggregatorV3Interface` from GitHub instead of what Patrick did of copying the whole interface into the file.

```solidity
 pragma solidity 0.8.26;
 import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";
 contract FundMe {}
```

## Getting real world data from Chainlink

So, to recap the `AggregatorV3Interface` allows us to interact with the Data Feed contract.

Back to our code, first we need to declare a new variable, `priceFeed`, of type `AggregatorV3Interface` and pass it the correct contract address:

`AggregatorV3Interface priceFeed = AggregatorV3Interface(0x1b44F3514812d835EB1BDB0acB33d3fA3351Ee43);`

Then, we will use the `latestRoundData()` function, first let's take a look at its signature:

`function latestRoundData() external view returns (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound);`

For now, we will focus on the `answer` value in our code:

```solidity
function getLatestPrice() public view returns (int) {
    (,int price,,,) = priceFeed.latestRoundData();
    return price;
}
```

Our `getLatestPrice()` function now retrieves the latest ETH price in USD from the `latestRoundData()` function of the Data Feed contract.

### Decimals

- `msg.value` is a `uint256` value with 18 decimal places.
- `answer` is an `int256` value with 8 decimal places (USD-based pairs use 8 decimal places, while ETH-based pairs use 18 decimal places).

This means the `price` returned from our `latestRoundData` function isn't directly compatible with `msg.value`. To match the decimal places, we multiply `price` by 1e10:

```solidity
return price * 1e10;
```

### Typecasting

Typecasting, or type conversion, involves converting a value from one data type to another. In Solidity, not all data types can be converted due to differences in their underlying representations and the potential for data loss. However, certain conversions, such as from `int` to `uint`, are allowed.

```solidity
return uint(price) * 1e10;
```

We can finalize our `view` function as follows:

```solidity
function getLatestPrice() public view returns (uint256) {
    (,int answer,,,) = priceFeed.latestRoundData();
    return uint(answer) * 1e10;
}
```

This complete `getLatestPrice` function retrieves the latest price, adjusts the decimal places, and converts the value to an unsigned integer, making it compatible for its use inside other functions.

## Solidity math

So up to now, we have `getPrice()` and `getConversionRate()` functions.

The `getPrice` function returns the current value of Ethereum in USD as a `uint256`.
The `getConversionRate` function converts a specified amount of ETH to its USD equivalent.

In Solidity, only integer values are used, as the language does not support floating-point numbers.

```solidity
function getConversionRate(uint256 ethAmount) internal view returns (uint256) {
    uint256 ethPrice = getPrice();
    uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1e18;
    return ethAmountInUsd;
}
```

> The line `uint256 ethAmountInUsd = (ethPrice * ethAmount)` results in a value with a precision of 1e18 \* 1e18 = 1e36. To bring the precision of `ethAmountInUsd` back to 1e18, we need to divide the result by 1e18.

**Always multiply before dividing to maintain precision and avoid truncation errors.**

### Example of `getConversionRate`

- `ethAmount` is set at 1 ETH, with 1e18 precision.
- `ethPrice` is set at 2000 USD, with 1e18 precision, resulting in 2000e18.
- `ethPrice * ethAmount` results in 2000e36.
- To scale down `ethAmountInUsd` to 1e18 precision, divide `ethPrice * ethAmount` by 1e18.

### Checking Minimum USD Value

We can verify if users send at least 5 USD to our contract:

```solidity
require(getConversionRate(msg.value) >= MINIMUM_USD, "You need to spend more ETH!");
```

Since `getConversionRate` returns a value with 18 decimal places, we need to multiply `5` by `1e18`, resulting in `5 * 1e18` (equivalent to `5 * 10**18`).

### Deployment to the Testnet

Let's deploy the `FundMe` contract to a testnet. After deployment, the `getPrice` function can be called to obtain the current value of Ethereum. It's also possible to send money to this contract, and an error will be triggered if the ETH amount is less than 5 USD.

## `msg.sender` explained

In this lesson, we will learn how to track addresses that are funding the contract and the amounts they will send to it.

First, we will create an array of addresses:

`address[] public funders;`

Then, when someone sends money we will add their address to the array:

`funders.push(msg.sender);`

We can also map each funder's address to the amount they have sent using mappings:

`mapping(address => uint256) public addressToAmountFunded;`

Finally, the `addressToAmountFunded` mapping associates each funder's address with the total amount they have contributed. When a new amount is sent, we can add it to the user's total contribution:

`addressToAmountFunded[msg.sender] += msg.value;`

## Libraries

When a functionality can be _commonly used_, we can create a **library** to efficiently manage repeated parts of codes

- Solidity libraries are similar to contracts but do not allow the declaration of any **state variables** and **cannot receive ETH**.
- All functions in a library must be declared as `internal`.

We can start by creating a new file called `PriceConverter.sol`, and replace the `contract` keyword with `library`.

Let's copy `getPrice`, `getConversionRate`, and `getVersion` functions from the `FundMe.sol` contract into our new library, remembering to import the `AggregatorV3Interface` into `PriceConverter.sol`. Finally, we can mark all the functions as `internal`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";

library PriceConverter {
    function getPrice() internal view returns (uint256) {
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x694AA1769357215DE4FAC081bf1f309aDC325306);
        (, int256 answer, , , ) = priceFeed.latestRoundData();
        return uint256(answer * 10000000000);
    }

    function getConversionRate(uint256 ethAmount) internal view returns (uint256) {
        uint256 ethPrice = getPrice();
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
        return ethAmountInUsd;
    }
}
```

### Accessing the Library

You can import the library in your contract and attach it to the desired type with the keyword `using`:

```Solidity
import {PriceConverter} from "./PriceConverter.sol";
using PriceConverter for uint256;
```

The `PriceConverter` functions can then be called as if they are native to the `uint256` type. For example, calling the `getConversionRate()` function will now be changed into:

```solidity
require(msg.value.getConversionRate() >= minimumUsd, "didn't send enough ETH");
```

Here, `msg.value`, which is a `uint256` type, is extended to include the `getConversionRate()` function. The `msg.value` gets passed as the first argument to the function. If additional arguments are needed, they are passed in parentheses:

```solidity
uint256 result = msg.value.getConversionRate(123);
```

In this case, `123` is passed as the second `uint256` argument to the function.

## Resetting an array

At this point, these notes are incomplete and are just for the purpose of roughly understanding the FundMe code.

### Introduction

In this section, we'll focus on one of the final steps to complete the `withdraw` function: effectively resetting the `funders` array.

The simplest way to reset the `funders` array is similar to the method used with the mapping: iterate through all its elements and reset each one to 0. Alternatively, we can create a brand new `funders` array.

```solidity
funders = new address[]();
```

> 🗒️ **NOTE**:
> You might recall using the `new` keyword when deploying a contract. In this context, however, it resets the `funders` array to a zero-sized, blank address array.

## Sending ETH from a contract

This lesson explores three different methods of sending ETH from one account to another: `transfer`, `send`, and `call`. We will understand their differences, how each one works, and when to use one instead of another.

### Transfer

The `transfer` function is the simplest way to send Ether to a recipient address:

```solidity
payable(msg.sender).transfer(amount); // the current contract sends the Ether amount to the msg.sender
```

It's necessary to convert the recipient address to a **payable** address to allow it to receive Ether. This can be done by wrapping `msg.sender` with the `payable` keyword.

However, `transfer` has a significant limitation. It can only use up to 2300 gas and it reverts any transaction that exceeds this gas limit, as illustrated by [Solidity by Example](https://solidity-by-example.org/sending-ether/).

### Send

The `send` function is similar to `transfer`, but it differs in its behaviour:

```solidity
bool success = payable(msg.sender).send(address(this).balance);
require(success, "Send failed");
```

Like `transfer`, `send` also has a gas limit of 2300. If the gas limit is reached, it will not revert the transaction but return a boolean value (`true` or `false`) to indicate the success or failure of the transaction. It is the developer's responsibility to handle failure correctly, and it's good practice to trigger a **revert** condition if the `send` returns `false`.

### Call

The `call` function is flexible and powerful. It can be used to call any function **without requiring its ABI**. It does not have a gas limit, and like `send`, it returns a boolean value instead of reverting like `transfer`.

```solidity
(bool success, ) = payable(msg.sender).call{value: address(this).balance}("");
require(success, "Call failed");
```

To send funds using the `call` function, we convert the address of the receiver to `payable` and add the value inside curly brackets before the parameters passed.

The `call` function returns two variables: a boolean for success or failure, and a byte object which stores returned data if any.

> 👀❗**IMPORTANT**:
> `call` is the recommended way of sending and receiving Ethereum or other blockchain native tokens.

### Conclusion

In conclusion, _transfer_, _send_, and _call_ are three unique methods for transferring Ether in Solidity. They vary in their syntax, behaviour, and gas limits, each offering distinct advantages and drawbacks.

## Using constructor to avoid hacks

Currently, **anyone** can call the `withdraw` function and drain all the funds from the contract. To fix this, we need to **restrict** the withdrawal function to the contract owner.

The constructor function is automatically called during contract deployment, within the same transaction that deploys the contract.

We can use the constructor to set the contract's owner immediately after deployment:

```solidity
address public owner;
constructor() {
    owner = msg.sender;
}
```

Here, we initialize the state variable `owner` with the contract deployer's address (`msg.sender`).

### Modifying the Withdraw Function

The next step is to update the `withdraw` function to ensure it can only be called by the owner:

```solidity
function withdraw() public {
    require(msg.sender == owner, "must be owner");
    // rest of the function here
}
```

Before executing any withdrawal actions, we check that `msg.sender` is the owner. If the caller is not the owner, the operation **reverts** with the error message "must be the owner" This access restriction ensures that only the intended account can execute the function.

### Conclusion

By incorporating a constructor to assign ownership and updating the withdraw function to restrict access, we have significantly improved the security of the fundMe contract. These changes ensure that only the contract owner can withdraw funds, preventing unauthorized access.

## Solidity function modifiers

In this lesson, we will explore **modifiers** and how they can simplify code writing and management in Solidity. Modifiers enable developers to create reusable code snippets that can be applied to multiple functions, enhancing code readability, maintainability, and security.

### Repeated Conditions

If we build a contract with multiple _administrative functions_, that should only be executed by the contract owner, we might repeatedly check the caller identity:

```solidity
require(msg.sender == owner, "Sender is not owner");
```

However, repeating this line in every function clutters the contract, making it harder to read, maintain, and debug.

### Modifiers

Modifiers in Solidity allow embedding **custom lines of code** within any function to modify its behaviour.

Here's how to create a modifier:

```solidity
modifier onlyOwner {
    require(msg.sender == owner, "Sender is not owner");
    _;
}
```

> 🗒️ **NOTE**:
> The modifier is named `onlyOwner` to reflect the condition it checks.

### The `_` (underscore)

The underscore `_` placed in the body is a placeholder for the modified function's code. When the function with the modifier is called, the code before `_` runs first, and if it succeeds, the function's code executes next.

For example, the `onlyOwner` modifier can be applied to the `withdraw` function like this:

```solidity
function withdraw(uint amount) public onlyOwner {
    // Function logic
}
```

When `withdraw` is called, the contract first executes the `onlyOwner` modifier. If the `require` statement passes, the rest of the `withdraw` function executes.

If the underscore `_` were placed before the `require` statement, the function's logic would execute first, followed by the `require` check, which is not the intended use case.

### Conclusion

Using modifiers like `onlyOwner` simplifies contract development by centralizing common conditions, reducing code repetition, and enhancing contract readability and maintainability.

## Deploying to testnet and end to end testing

First, deploy to Sepolia testnet, and then:

### Contract Interaction

After successfully deploying the `FundMe` contract, you'll see several buttons to interact with it:

- **Red button**: Payable functions (e.g., `fund`)
- **Orange button**: Non-payable functions (e.g., `withdraw`)
- **Blue buttons**: `view` and `pure` functions

The `fund` function allows us to send ETH to the contract (minimum 5 USD). The `owner` of the contract is our MetaMask account, as the **constructor** sets the deployer as the owner.

> 🗒️ **NOTE**:
> If the `fund` function is called without any value or with less than 5 USD, you will encounter a gas estimation error, indicating insufficient ETH, and gas will be wasted.

### Successful Transaction

If you set the amount to `0.1 ETH` and confirm it in MetaMask, you can then track the successful transaction on Etherscan. In the Etherscan transaction log, you will see that the `fundMe` balance has increased by `0.1 ETH`. The `funders` array will register your address, and the mapping `addressToAmountFunded` will record the amount of ETH sent.

### Withdraw Function and Errors

After funding the contract, we can initiate the `withdraw` function. This function can only be called by the owner; if a non-owner account attempts to withdraw, a gas estimation error will be thrown, and the function will revert.

Upon successful withdrawal, the `fundMe` balance, the `addressToAmountFunded` mapping, and the `funders` array will all reset to zero.

### Conclusion

In this lesson, we've explored the end-to-end process of deploying and interacting with a Solidity contract using Remix and MetaMask. We covered the deployment transaction, contract interaction, and how to handle successful transactions and potential errors.

## Immutability and constants

The variables `owner` and `minimumUSD` are set one time and they never change their value: `owner` is assigned during contract creation, and `minimumUSD` is initialized at the beginning of the contract.

> Naming conventions for `constant` are all caps with underscores in place of spaces (e.g., `MINIMUM_USD`).

While `constant` variables are for values known at compile time, `immutable` can be used for variables set at deployment time that will not change. The naming convention for `immutable` variables is to add the prefix `i_` to the variable name (e.g., `i_owner`).

## Creating custom errors

### Require

One way to improve gas efficiency is by optimizing our `require` statements. Currently, the `require` statement forces us to store the string 'sender is not an owner'. Each character in this string is stored individually, making the logic to manage it complex and expensive.

### Custom Errors

Introduced in **Solidity 0.8.4**, custom errors can be used in `revert` statements. These errors should be declared at the top of the code and used in `if` statements. The cheaper error code is then called in place of the previous error message string, reducing gas costs.

We can start by creating a custom error:

```solidity
error NotOwner();
```

Then, we can replace the `require` function with an `if` statement, using the `revert` function with the newly created error:

```solidity
if (msg.sender != i_owner) {
    revert NotOwner();
}
```

By implementing custom errors, we reduce gas costs and simplify error handling in our smart contracts.

### Conclusion

In this lesson, we have learned how to further optimize gas efficiency in Solidity contracts by using custom errors instead of traditional require statements with strings.

## Receive and fallback functions

In Solidity, if Ether is sent to a contract without a `receive` or `fallback` function, the transaction will be **rejected**, and the Ether will not be transferred. In this lesson, we'll explore how to handle this scenario effectively.

### receive and fallback functions

`receive` and `fallback` are _special functions_ triggered when users send Ether directly to the contract or call non-existent functions. These functions do not return anything and must be declared `external`.

To illustrate, let's create a simple contract:

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract FallbackExample {
    uint256 public result;

    receive() external payable {
        result = 1;
    }

    fallback() external payable {
        result = 2;
    }
}
```

In this contract, `result` is initialized to zero. When Ether is sent to the contract, the `receive` function is triggered, setting `result` to one. If a transaction includes **data** but the specified function _does not exist_, the `fallback` function will be triggered, setting `result` to two. For a comprehensive explanation, refer to [SolidityByExample](https://solidity-by-example.org/fallback/).

```text
// Ether is sent to contract
//      is msg.data empty?
//          /   \
//         yes  no
//         /     \
//    receive()?  fallback()
//     /   \
//   yes   no
//  /        \
//receive()  fallback()
```

### Sending Ether to fundMe

When a user sends Ether **directly** to the `fundMe` contract without calling the `fund` function, the `receive` function can be used to _redirect_ the transaction to the `fund` function:

```solidity
receive() external payable {
    fund();
}

fallback() external payable {
    fund();
}
```

To test this functionality, send some Sepolia Ether to the `fundMe` contract using MetaMask. This does not directly call the `fund` function, but the `receive` function will trigger it. After confirming the transaction, you can check the `funders` array to see that it has been updated, reflecting the successful invocation of the `fund` function by the `receive` function.

This approach ensures that all transactions are processed as intended. Although directly calling the `fund` function costs less gas, this method ensures the user's contribution is properly acknowledged and credited.

### Conclusion

By implementing `receive` and `fallback` functions, contracts can handle direct Ether transfers and non-existent function calls effectively, ensuring that transactions are processed as intended and users' contributions are properly tracked.

## ❓ Questions and 💪 Exercises

- Exercise 💪: Implement a function `contributionCount` to monitor how many times a user calls the `fund` function to send money to the contract.

- Exercise 💪: Create a simple library called `MathLibrary` that contains a function `sum` to add two `uint256` numbers. Then create a function `calculateSum` inside the `fundMe` contract that uses the `MathLibrary` function.

- Exercise 💪: Implement a function `callAmountTo` using `call` to send Ether from the contract to an address provided as an argument. Ensure the function handles failures appropriately.

- Exercise 💪: Implement a modifier named `onlyAfter(uint256 _time)` that ensures a function can only be executed after a specified time.

- Exercise 💪: Interact with the `FundMe` contract on Remix and explore all possible outcomes that its functions can lead to.

- Exercise 💪: Create a custom error that is triggered when msg.sender is address(0) and then convert it into an equivalent if statement with a `revert` function.

- Question ❓: What are interfaces in Solidity and why are they helpful?
- Question ❓: What are libraries in Solidity?
- Question ❓: What are the consequences if a library function is not marked as `internal`? (Check [this](https://x.com/garosan1/status/1896967666453393642) thread)
- Question ❓: What are the primary differences between _transfer_, _send_, and _call_ when transferring Ether?
- Question ❓: Why is it necessary to convert an address to a `payable` type before sending Ether to it?
- Question ❓: Why is it beneficial to use `modifiers` for access control?
- Question ❓: What are the benefits of declaring custom errors instead of using the `require` keyword?
- Question ❓: How does the `fallback` function differ from the `receive` function?
- Question ❓: What does it happen when Ether is sent with _data_ but in the contract only a `receive` function exist?
- Question ❓: In the process of verifying a contract in Etherscan, you sometimes need to flatten the contract, what does flattening a contract means?

## Final Code (For Sepolia)

`FundMe.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";
import {PriceConverter} from "./PriceConverter.sol";

error NotOwner();

contract FundMe {
    using PriceConverter for uint256;

    mapping(address => uint256) public addressToAmountFunded;
    address[] public funders;

    // Could we make this constant?  /* hint: no! We should make it immutable! */
    address public i_owner;
    uint256 public constant MINIMUM_USD = 5 * 10 ** 18;

    constructor() {
        i_owner = msg.sender;
    }

    modifier onlyOwner() {
        if (msg.sender != i_owner) revert NotOwner();
        _;
    }

    function fund() public payable {
        require(msg.value.getConversionRate() >= MINIMUM_USD, "You need to spend more ETH!");
        // require(PriceConverter.getConversionRate(msg.value) >= MINIMUM_USD, "You need to spend more ETH!");
        addressToAmountFunded[msg.sender] += msg.value;
        funders.push(msg.sender);
    }

    function getVersion() public view returns (uint256) {
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x694AA1769357215DE4FAC081bf1f309aDC325306);
        return priceFeed.version();
    }

    function withdraw() public onlyOwner {
        for (uint256 funderIndex = 0; funderIndex < funders.length; funderIndex++) {
            address funder = funders[funderIndex];
            addressToAmountFunded[funder] = 0;
        }
        funders = new address[](0);
        // // transfer
        // payable(msg.sender).transfer(address(this).balance);

        // // send
        // bool sendSuccess = payable(msg.sender).send(address(this).balance);
        // require(sendSuccess, "Send failed");

        // call
        (bool callSuccess,) = payable(msg.sender).call{value: address(this).balance}("");
        require(callSuccess, "Call failed");
    }

    fallback() external payable {
        fund();
    }

    receive() external payable {
        fund();
    }
}
```

`PriceConverter.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";

library PriceConverter {
    // We could make this public, but then we'd have to deploy it
    function getPrice() internal view returns (uint256) {
        // Sepolia ETH / USD Address
        // https://docs.chain.link/data-feeds/price-feeds/addresses
        AggregatorV3Interface priceFeed = AggregatorV3Interface(
            0x694AA1769357215DE4FAC081bf1f309aDC325306
        );
        (, int256 answer, , , ) = priceFeed.latestRoundData();
        // ETH/USD rate in 18 digit
        return uint256(answer * 10000000000);
    }

    // 1000000000
    function getConversionRate(
        uint256 ethAmount
    ) internal view returns (uint256) {
        uint256 ethPrice = getPrice();
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
        // the actual ETH/USD conversion rate, after adjusting the extra 0s.
        return ethAmountInUsd;
    }
}
```

- [PriceConverter.sol on Etherscan](https://sepolia.etherscan.io/address/0x6f70f2752fbfde63ff4aed77b99976abda0c4025)
- [FundMe.sol on Etherscan](https://sepolia.etherscan.io/address/0x18cafd8b130a5dd02ce8b873ae9d590c4a1312a7)

## 🛠️ Links and Resources

- [GitHub Repo](https://github.com/Cyfrin/remix-fund-me-cu)
- [Ethereum Unit Converter](https://eth-converter.com/)
- [Ether Units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#ether-units)
- [Chainlink](https://chain.link/)
- [Chainlink Docs](https://docs.chain.link/)
- [Solidity by Example - Library](https://solidity-by-example.org/library/)
