# Section 2. Foundry Fund Me

## Introduction

We will learn:

- Putting our code on Github
- Explore storage using the FunWithStorage contract
- Professional deployment techniques with Foundry scripts
- Gas optimization techniques
- Debugging
- Creating professional tests
- Setting up a professional development environment
- How to use a price feed
- How to use Chisel

## Project Setup

```bash
mkdir foundry-fund-me
cd foundry-fund-me
forge init
```

## Finishing the setup

Delete the 3 `Counter` example files and copy the [fund me contract](https://github.com/Cyfrin/remix-fund-me-cu/blob/main/FundMe.sol), and the [PriceConverter.sol](https://github.com/Cyfrin/remix-fund-me-cu/blob/main/PriceConverter.sol)

Remember that as of Feb 1st, 2025, the contracts now live in:

`import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";`

Update the path accordingly.

After this if we try `forge compile` we will get an error. We need to install the chainlink dependencies with `forge install`:

`forge install smartcontractkit/chainlink-brownie-contracts@0.6.1 --no-commit`

We end the install command with `--no commit` in order to not create a git commit. More on this option later.

Go to the `lib` folder and you will see the newly installed `chainlink-brownie-contracts` library. Inside, you'll see a folder called `contracts` then a folder called `src`. Here we can find the `AggregatorV3Interface` that we are importing in `FundMe.sol`.

But now we need to indicate the correct path to our imports of where to look for these files, so in `foundry.toml` we need to add this:

`remappings = ['@chainlink/contracts/=lib/chainlink-brownie-contracts/contracts/']`.

If you now try `forge compile` everything works correctly.

## Writing tests

Inside the `test` folder create a file called `FundMeTest.t.sol`. `.t.` is a naming convention of Foundry, please use it.

Let's now write our test file. We need to:

`import {Test} from "forge-std/Test.sol";`

and then, we will deploy our `FundMe` contract inside our `setUp` function. This function is always the first to execute whenever we run our tests. Here we will perform all the prerequisite actions that are required before doing the actual testing, things like:

- Deployments
- User addresses
- Balances
- Approvals

Let's also create a function called `testDemo`. Our file should look like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {Test} from "forge-std/Test.sol";

contract FundMeTest is Test {

    function setUp() external { }

    function testDemo() public { }

 }
```

Now run `forge test`. 1 empty test should pass.

Let's update the contract to the following and run `forge test` again:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {Test} from "forge-std/Test.sol";

contract FundMeTest is Test {
    uint256 favNumber = 0;
    bool greatCourse = false;

    function setUp() external {
        favNumber = 1337;
        greatCourse = true;
    }

    function testDemo() public view {
        assertEq(favNumber, 1337);
        assertEq(greatCourse, true);
    }
}
```

As you can see this is the order of operations:

1. We declare some state variables.
2. Next up `setUp()` is called.
3. After that, forge runs all the test functions.

The `forge-std/Test.sol` offers us a `console.log` capability, let's import it:

`import {Test, console} from "forge-std/Test.sol";`

And add some some console.logs:

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {Test, console} from "forge-std/Test.sol";

contract FundMeTest is Test {
    uint256 favNumber = 0;
    bool greatCourse = false;

    function setUp() external {
        favNumber = 1337;
        greatCourse = true;
        console.log("This will get printed first!");
    }

    function testDemo() public view {
        assertEq(favNumber, 1337);
        assertEq(greatCourse, true);
        console.log("This will get printed second!");
        console.log("Updraft is changing lives!");
        console.log(
            "You can print multiple things, for example this is a uint256, followed by a bool:",
            favNumber,
            greatCourse
        );
    }
}
```

`forge test` has an option called verbosity. By controlling this option we decide how verbose should the output of the `forge test` be. The default `forge test` has a verbosity of 1. Here are the verbosity levels, choose according to your needs:

```
Verbosity levels:

- 2: Print logs for all tests
- 3: Print execution traces for failing tests
- 4: Print execution traces for all tests, and setup traces for failing tests
- 5: Print execution and setup traces for all tests
```

If we want to see the console.logs, we would run `forge test -vv` (the number of v's indicates the level).

Let's delete the logs for now and deploy the `FundMe` contract.

For that, we will first need to import it into our test file, then declare it as a state variable and deploy it in the `setUp` function.

Our test file should look like this now:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {Test, console} from "forge-std/Test.sol";
import {FundMe} from "../src/FundMe.sol";

contract FundMeTest is Test {
    FundMe fundMe;

    function setUp() external {
        fundMe = new FundMe();
    }

    function testMinimumDollarIsFive() public view {
        assertEq(fundMe.MINIMUM_USD(), 5e18);
    }
}
```

## More tests

Let's test that the owner is recorded properly, add this:

```solidity
function testOwnerIsMsgSender() public view {
    assertEq(fundMe.i_owner(), msg.sender);
}
```

Run `forge test` and you will see an error:

```bash
[FAIL: assertion failed: 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496 != 0x1804c8AB1F12E6bbf3894d4083f33e07309d1f38] testOwnerIsMsgSender()
```

Why?

Technically we are not the ones that deployed the `FundMe` contract. The `FundMe` contract was deployed by the `setUp` function, which is part of the `FundMeTest` contract. So, even though we are the ones who called `setUp` via `forge test`, the actual testing contract is the deployer of the `FundMe` contract.

To fix this, we would need to tweak our test function like this:

```solidity
    function testOwnerIsMsgSender() public {
        assertEq(fundMe.i_owner(), address(this));

    }
```

## Advanced deploy scripts

When we went straight to testing, we left behind a very important element: deploy scripts. Why is this important you ask? Because we need a certain degree of flexibility that we can't obtain in any other way, let's look through the two files `FundMe.sol` and `PriceConverter.sol`, we can see that both have an address (`0x694AA1769357215DE4FAC081bf1f309aDC325306`) hardcoded for the AggregatorV3Interface. This address is valid, it matches the AggregatorV3 on Sepolia but what if we want to test on Anvil? What if we deploy on mainnet or Arbitrum? What then?

The deploy script is the key to overcome this problem!

Create a new file called `DeployFundMe.s.sol` in `script` folder. Please use the `.s.sol` naming convention.

It should look like this:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

import {Script} from "forge-std/Script.sol";
import {FundMe} from "../src/FundMe.sol";

contract DeployFundMe is Script {
    function run() external{
        vm.startBroadcast();
        new FundMe();
        vm.stopBroadcast();
    }

}
```

In my case it was weirdly referencing `Counter.sol`, so I had to run `forge clean` to fix it.

## Running tests on chain forks

This course will cover 4 different types of tests:

- **Unit tests**: Focus on testing individual smart contract functions or functionalities.
- **Integration tests**: Verify how a smart contract interacts with other contracts or external systems.
- **Forking tests**: Forking refers to creating a copy of a blockchain state at a specific point in time. This copy, called a fork, is then used to run tests in a simulated environment.
- **Staging tests**: Execute tests against a deployed smart contract on a staging environment before mainnet deployment.

Coming back to our contracts, the central functionality of our protocol is the `fund` function.

For that to work, we need to be sure that AggregatorV3 runs the current version. We know from previous courses that the version returned needs to be `4`. Let's put it to the test!

Add the following code to your test file:

```solidity
function testPriceFeedVersionIsAccurate() public view {
    uint256 version = fundMe.getVersion();
    assertEq(version, 4);

}
```

It ... fails. But why? Looking through the code we see this AggregatorV3 address `0x694AA1769357215DE4FAC081bf1f309aDC325306` over and over again. The address is correct, its the Sepolia deployment of the AggregatorV3 contract. But our tests use Anvil for testing purposes, so that doesn't exist.

Let's use this to run tests just for the function we care about:

`forge test --mt testPriceFeedVersionIsAccurate`

As a curiosity if you had two functions named the same in different files, the above will run both functions.

But back to our problem, how can we fix it?

Forking is the solution we need. If we run the test on an anvil instance that copies the current Sepolia state, where AggregatorV3 exists at that address, then our test function will not revert anymore. For that, we need a Sepolia RPC URL.

So we would again create a `.env` file with our env variable:

`SEPOLIA_RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOURAPIKEYWILLGOHERE`

Remember to add it to source with `source .env`.

Then run `forge test --mt testPriceFeedVersionIsAccurate --fork-url $SEPOLIA_RPC_URL`

You can run `forge coverage --fork-url $SEPOLIA_RPC_URL` to check your total code coverage.

Obviously `forge coverage` runs your test suite, so in this case we are adding the `--fork-url` flag so that all tests pass, but normally you want to run just `forge coverage` to get the test suite coverage.

## Refactoring your tests

Given the hardcoded Sepolia address we cannot deploy to other chains. To fix this we can make our project more modular, which would help improve the maintainability, testing and deployment. **This is done by moving the hardcoded changing variables to the constructor**, thus regardless of the chain we deploy our contracts to, we provide the chain-specific elements once, in the constructor, and then we are good to go.

In `FundMe.sol`, create a new storage variable:

`AggregatorV3Interface private s_priceFeed;`

Add this in the constructor:

```solidity
constructor(address priceFeed){
    i_owner = msg.sender;
    s_priceFeed = AggregatorV3Interface(priceFeed);
}
```

Then this:

```solidity
function getVersion() public view returns (uint256){
    AggregatorV3Interface priceFeed = AggregatorV3Interface(s_priceFeed);
    return priceFeed.version();
}
```

Ok, so basically we are changing our `FundMe.sol` and `PriceConverter.sol` files to not hardcode the address for the priceFeed, but rather provide it at deploy time.

Our files should look like this:

`FundMe.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";
import {PriceConverter} from "./PriceConverter.sol";

error FundMe__NotOwner();

contract FundMe {
    using PriceConverter for uint256;

    mapping(address => uint256) public addressToAmountFunded;
    address[] public funders;

    address public i_owner;
    uint256 public constant MINIMUM_USD = 5 * 10 ** 18;
    AggregatorV3Interface private s_priceFeed;

    constructor(address priceFeed) {
        i_owner = msg.sender;
        s_priceFeed = AggregatorV3Interface(priceFeed);
    }

    function fund() public payable {
        require(
            msg.value.getConversionRate(s_priceFeed) >= MINIMUM_USD,
            "You need to spend more ETH!"
        );
        // require(PriceConverter.getConversionRate(msg.value) >= MINIMUM_USD, "You need to spend more ETH!");
        addressToAmountFunded[msg.sender] += msg.value;
        funders.push(msg.sender);
    }

    function getVersion() public view returns (uint256) {
        return s_priceFeed.version();
    }

    modifier onlyOwner() {
        // require(msg.sender == owner);
        if (msg.sender != i_owner) revert FundMe__NotOwner();
        _;
    }

    function withdraw() public onlyOwner {
        for (
            uint256 funderIndex = 0;
            funderIndex < funders.length;
            funderIndex++
        ) {
            address funder = funders[funderIndex];
            addressToAmountFunded[funder] = 0;
        }
        funders = new address[](0);

        (bool callSuccess, ) = payable(msg.sender).call{
            value: address(this).balance
        }("");
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

import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

library PriceConverter {
    function getPrice(
        AggregatorV3Interface priceFeed
    ) internal view returns (uint256) {
        (, int256 answer, , , ) = priceFeed.latestRoundData();
        return uint256(answer * 10000000000);
    }

    function getConversionRate(
        uint256 ethAmount,
        AggregatorV3Interface priceFeed
    ) internal view returns (uint256) {
        uint256 ethPrice = getPrice(priceFeed);
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
        return ethAmountInUsd;
    }
}
```

`//TODO. CLarify exactly why we made that change in PriceConverter.sol`

Take a moment and think if we missed updating anything in our project.

You probably know the answer, right? We are missing to provide the priceFeed address in our deploy script and in our test. Let's hardcode it for now:

`DeployFundMe.s.sol` should be updated with this now:

```solidity
function run() external returns (FundMe) {
    vm.startBroadcast();
    FundMe fundMe = new FundMe(0x694AA1769357215DE4FAC081bf1f309aDC325306);
    vm.stopBroadcast();
    return fundMe;
}
```

Finally, back in `FundMe.sol`, we also need to pass the `s_priceFeed` as input for `getConversionRate` in the `fund` function:

`msg.value.getConversionRate(s_priceFeed) >= MINIMUM_USD,`


Now we could do the same thing for our tests, but it would then become tedious and not a great practice to keep updating things in both files, so let's modify our deploy script so that we can use it in our test file. Your `DeployFundMe.s.sol` should look like this:

```solidity
// SPDX-License_identifier: MIT

pragma solidity 0.8.26;

import {Script} from "forge-std/Script.sol";
import {FundMe} from "../src/FundMe.sol";

contract DeployFundMe is Script {
    function run() external returns (FundMe) {
        vm.startBroadcast();
        FundMe fundMe = new FundMe(0x694AA1769357215DE4FAC081bf1f309aDC325306);
        vm.stopBroadcast();
        return fundMe;
    }
}
```

Now for our tests, let's import the deployment script into the `FundMe.t.sol`:

`import {DeployFundMe} from "../script/DeployFundMe.s.sol";`

- Create a new state variable `DeployFundMe deployFundMe;`;
- Update the `setUp` function:

```solidity
contract FundMeTest is Test {
    FundMe fundMe;
    DeployFundMe deployFundMe;

    function setUp() external {
        deployFundMe = new DeployFundMe();
        fundMe = deployFundMe.run();
    }

    function testMinimumDollarIsFive() public view {
        assertEq(fundMe.MINIMUM_USD(), 5e18);
    }
```

Let's call `forge test --fork-url $SEPOLIA_RPC_URL` to make sure everything compiles.

Looks like the `testOwnerIsMsgSender` fails again. Take a moment and think about why.

This surely must have to do with the fact that the deployer is coming now from our deploy script and not natively from within our setUp function.

**Note**: `vm.startBroadcast` is special, it uses the address that calls the test contract or the address / private key provided as the sender.

To account for the way `vm.startBroadcast` works, perform the following modification in `FundMe.t.sol`:

```solidity
function testOwnerIsMsgSender() public {
    assertEq(fundMe.i_owner(), msg.sender);
}
```

Run `forge test --fork-url $SEPOLIA_RPC_URL` again. Everything should work now.

## Deploying a mock priceFeed

The problem: So we've come a long way but we still need to call `--fork-url` with our tests, otherwise if we just run `forge test`, our `testPriceFeedVersionIsAccurate()` test function is going to fail because such contract doesn't exist in our local Anvil blockchain. And we don't want to be calling our `SEPOLIA_RPC_URL` every time we run tests, or we're going to run out of credits very easily. Enter mocks.

We are going to write a mock priceFeed to interact with in our local Anvil blockchain for the duration of our local tests.

Create a file in `script/HelperConfig.s.sol`.

We want to achieve 2 things:

- Deploy mocks when we are on a local anvil chain
- Keep track of contract addresses across different chains

Let's begin by setting the 2 functions that we're gonna need:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

import {Script} from "forge-std/Script.sol";

contract HelperConfig {
    function getSepoliaEthConfig() public pure {
        // priceFeed address
    }

    function getAnvilEthConfig() public pure {
        // priceFeed address
    }
}
```

Now instead of passing a bunch of variables with everything we need, let's create a struct for our configuration.

Our file should look like this now:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

import {Script} from "forge-std/Script.sol";

contract HelperConfig {
    NetworkConfig public activeNetworkConfig;

    struct NetworkConfig {
        address priceFeed; // ETH/USD priceFeed address
    }

    constructor() {
        if (block.chainid == 11155111) {
            activeNetworkConfig = getSepoliaEthConfig();
        } else {
            activeNetworkConfig = getAnvilEthConfig();
        }
    }

    function getSepoliaEthConfig() public pure returns (NetworkConfig memory) {
        // priceFeed address
        NetworkConfig memory sepoliaConfig = NetworkConfig({
            priceFeed: 0x694AA1769357215DE4FAC081bf1f309aDC325306
        });
        return sepoliaConfig;
    }

    function getAnvilEthConfig() public pure returns (NetworkConfig memory) {
        // priceFeed address
    }
}
```

Now let's modify our deploy script to include our new `HelperConfig`:

```solidity
// SPDX-License_identifier: MIT

pragma solidity 0.8.26;

import {Script} from "forge-std/Script.sol";
import {FundMe} from "../src/FundMe.sol";
import {HelperConfig} from "./HelperConfig.s.sol";

contract DeployFundMe is Script {
    function run() external returns (FundMe) {
        HelperConfig helperConfig = new HelperConfig();
        address ethUsdPriceFeed = helperConfig.activeNetworkConfig();

        vm.startBroadcast();
        FundMe fundMe = new FundMe(ethUsdPriceFeed);
        vm.stopBroadcast();
        return fundMe;
    }
}
```

If we now run `forge test --fork-url $SEPOLIA_RPC_URL` the test should pass correctly, meaning we haven't broken anything.

## 💪 Exercises Time

Now let's do a little exercise. What if I want to test this on Base Sepolia? or Optimism Sepolia?

I already have my RPC_URL configured for Optimism Sepolia so let me run:

`forge test --fork-url $OPTIMISM_SEPOLIA_RPC_URL`

Failed ❌! Hmm 🤔 sounds like we have to adjust our `HelperConfig.s.sol`, hopefully it's straight forward:

We need:

- The correct `chainid` from [ChainList](https://chainlist.org/)
- We need the correct price feed address from [ChainLink](https://docs.chain.link/data-feeds/price-feeds/addresses)

In `HelperConfig.s.sol` our `constructor()` is now:

```solidity
constructor() {
    if (block.chainid == 11155111) {
        activeNetworkConfig = getSepoliaEthConfig();
    } else if (block.chainid == 11155420) {
        activeNetworkConfig = getOptimismSepoliaConfig();
    } else {
        activeNetworkConfig = getAnvilEthConfig();
    }
}
```

Our new function:

```solidity
    function getOptimismSepoliaConfig()
        public
        pure
        returns (NetworkConfig memory)
    {
        NetworkConfig memory opSepoliaConfig = NetworkConfig({
            priceFeed: 0x61Ec26aA57019C486B10502285c5A3D4A4750AD7
        });
        return opSepoliaConfig;
    }
```

## Back to our mock

We still have the problem where if we try to test in our Anvil blockchain, our test will fail because there is no price feed contract in Anvil.

We need our `getAnvilEthConfig` function in `HelperConfig` to deploy a mock contract and return the mock address.

First of all, we need to make sure we import `Script.sol` in the `HelperConfig.s.sol` contract. Also, `HelperConfig` needs to inherit from `Script`. This will give us access to the `vm.startBroadcast`/`vm.stopBroadcast` functionality. We will use this to deploy our mocks.

```solidity
import {Script} from "forge-std/Script.sol";

contract HelperConfig is Script {

}
```

In the `test` folder create a new folder called `mocks`. Inside `mocks` create a new file `MockV3Aggregator.sol`.

Creating a mock price feed from scratch is not easy so copy this content into your file:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

/**
 * @title MockV3Aggregator
 * @notice Based on the FluxAggregator contract
 * @notice Use this contract when you need to test
 * other contract's ability to read data from an
 * aggregator contract, but how the aggregator got
 * its answer is unimportant
 */
contract MockV3Aggregator is AggregatorV3Interface {
    uint256 public constant version = 4;

    uint8 public decimals;
    int256 public latestAnswer;
    uint256 public latestTimestamp;
    uint256 public latestRound;

    mapping(uint256 => int256) public getAnswer;
    mapping(uint256 => uint256) public getTimestamp;
    mapping(uint256 => uint256) private getStartedAt;

    constructor(uint8 _decimals, int256 _initialAnswer) {
        decimals = _decimals;
        updateAnswer(_initialAnswer);
    }

    function updateAnswer(int256 _answer) public {
        latestAnswer = _answer;
        latestTimestamp = block.timestamp;
        latestRound++;
        getAnswer[latestRound] = _answer;
        getTimestamp[latestRound] = block.timestamp;
        getStartedAt[latestRound] = block.timestamp;
    }

    function updateRoundData(
        uint80 _roundId,
        int256 _answer,
        uint256 _timestamp,
        uint256 _startedAt
    ) public {
        latestRound = _roundId;
        latestAnswer = _answer;
        latestTimestamp = _timestamp;
        getAnswer[latestRound] = _answer;
        getTimestamp[latestRound] = _timestamp;
        getStartedAt[latestRound] = _startedAt;
    }

    function getRoundData(
        uint80 _roundId
    )
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return (
            _roundId,
            getAnswer[_roundId],
            getStartedAt[_roundId],
            getTimestamp[_roundId],
            _roundId
        );
    }

    function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return (
            uint80(latestRound),
            getAnswer[latestRound],
            getStartedAt[latestRound],
            getTimestamp[latestRound],
            uint80(latestRound)
        );
    }

    function description() external pure returns (string memory) {
        return "v0.6/test/mock/MockV3Aggregator.sol";
    }
}
```

Now let's import this in our `HelperConfig.s.sol` file and deploy it in our `getAnvilEthConfig` function, then return the address.

Make these changes to `HelperConfig.s.sol`:

```solidity
import {MockV3Aggregator} from "../test/mocks/MockV3Aggregator.sol";

[...]

// In state variables section
MockV3Aggregator mockPriceFeed;

[...]

function getAnvilEthConfig() public returns (NetworkConfig memory) {
    vm.startBroadcast();
    mockPriceFeed = new MockV3Aggregator(8, 2000e8);
    vm.stopBroadcast();

    NetworkConfig memory anvilConfig = NetworkConfig({
        priceFeed: address(mockPriceFeed)
    });

    return anvilConfig;

}
```

If everything went well, run `forge test` and you will see your tests passing. Awesome 👏👏

## 🎩 Magic Numbers

Magic numbers refer to literal values directly included in the code without any explanation or context. We should always avoid magic numbers.

Write clean, maintainable, and less error-prone code. You make your own life easier, you make your auditor(s) life easier. Use constants and configuration variables.

Open `HelperConfig.s.sol`, go to the `getAnvilEthConfig` function and delete the `8` corresponding to the decimals and `2000e8` corresponding to the `_initialAnswer` that are used inside the `MockV3Aggregator`'s constructor.

At the top of the `HelperConfig` contract create two new variables:

```solidity
uint8 public constant DECIMALS = 8;
int256 public constant INITIAL_PRICE = 2000e8;
```

Now replace the deleted magic numbers:

```solidity
vm.startBroadcast();
mockPriceFeed = new MockV3Aggregator(DECIMALS, INITIAL_PRICE);
vm.stopBroadcast();
```

## More fine-tuning and refactoring

One thing that we should do that would make our `getAnvilEthConfig()` more efficient is to check if we already deployed the `mockPriceFeed` before deploying it once more.

Remember how addresses that are declared but aren't initialized default to address zero? We can use this to create our check.

Add this at the top of our `getAnvilEthConfig()` function:

```solidity
if (activeNetworkConfig.priceFeed != address(0)) {
    return activeNetworkConfig;
}
```

Also, to be more precise in our naming, let's rename `getAnvilEthConfig()` to `getOrCreateAnvilEthConfig()` and update in the constructor too.

Great. Now if you run `forge test` everything should work fine.

And if you're like me and explored a couple other options, run `forge test --fork-url $SEPOLIA_RPC_URL` and `forge test --fork-url $OPTIMISM_SEPOLIA_RPC_URL` and they should both work correctly.

## Foundry Test Cheatcodes

Let's now try to increase that test coverage. Call `forge coverage`. Not all things require 100% coverage, or maybe achieving 100% coverage is too time expensive, but ... 12-13%? That is a joke, we can do way better than that.

Let's take a look at the `fund` function in `FundMe.sol` for a second. What should it do?

- `fund` should revert if our `msg.value` converted to USDC is lower than `MINIMUM_USD`
- The `addressToAmountFunded` mapping should be updated appropriately to show the funded value;
- The `funders` array should be updated with `msg.sender`

To test all this we will learn to use one of Foundry's main features: Cheatcodes.

From the docs: *Cheatcodes give you powerful assertions, the ability to alter the state of the EVM, mock data, and more.*

We will first use `expectRevert`. You can read all about it [here](https://book.getfoundry.sh/cheatcodes/expect-revert).

Open `FundMe.t.sol` and add the following function:

```solidity
function testFundFailsWIthoutEnoughETH() public {
    vm.expectRevert(); // <- The next line after this one should revert! If not test fails.
    fundMe.fund();     // <- We send 0 value

}
```

Run `forge test`. We are attempting to fund the contract with `0` value, it reverts and our test passes.

Now let's refactor a bit more in `FundMe.sol`:

- As we've discussed before storage variables should start with `s_`. Change all the `addressToAmountFunded` mentions to `s_addressToAmountFunded` and all the `funders` to `s_funders`
- Also change the visibility of these variables to `private`. Private variables are more gas-efficient than public ones.

Call a quick `forge test` to make sure nothing broke anywhere.

Now that we made those two variables private, we need to write some getters for them. Please add the following at the end of `FundMe.sol` (but before `fallback()` and `receive()`):

```solidity
/** Getter Functions */

function getAddressToAmountFunded(address fundingAddress) public view returns (uint256) {
    return s_addressToAmountFunded[fundingAddress];
}

function getFunder(uint256 index) public view returns (address) {
    return s_funders[index];

}
```

Great. Now we can test points 2 and 3 of the `fund()` function. Add the following in `FundMe.t.sol`:

```solidity
function testFundUpdatesFundDataStructure() public {
    fundMe.fund{value: 10 ether}();
    uint256 amountFunded = fundMe.getAddressToAmountFunded(msg.sender);
    assertEq(amountFunded, 10 ether);

}
```

Run `forge test --mt testFundUpdatesFundDataStructure` aaand it fails.

Put `address(this)` instead of `msg.sender`. Now it passed, but we still don't quite get why.

**User management** is a very important aspect you need to take care of when writing tests. Imagine you are writing a more complex contract, where you have different user roles, maybe the `owner` has some privileges, different from an `admin` who has different privileges from a `minter`, who, as you've guessed, has different privileges from the `end user`. How can we differentiate all of them in our testing? We need to make sure we can write tests about who can do what.

As always, Foundry can help us with that. Please **remember the cheatcodes below, you are going to use them thousands of times**.

1. [prank](https://book.getfoundry.sh/cheatcodes/prank)
   "Sets `msg.sender` to the specified address for the next call. *The next call* includes static calls as well, but not calls to the cheat code address."

2. [startPrank](https://book.getfoundry.sh/cheatcodes/start-prank) and [stopPrank](https://book.getfoundry.sh/cheatcodes/stop-prank)
   `startPrank` sets `msg.sender` for all subsequent calls until `stopPrank` is called. Everything between the `vm.startPrank` and `vm.stopPrank` is signed by the address you provide inside the ().

To create a new user address we can use the `makeAddr` cheatcode. Read more about it [here](https://book.getfoundry.sh/reference/forge-std/make-addr).

Add the following line at the start of your `FundMeTest` contract:

`address USER = makeAddr("user");`

Now whenever we need a user to call a function we can use `prank` and `USER` to run our tests.

To further increase the readability of our contract, let's avoid using a magic number for the funded amount. Create a constant variable called `SEND_VALUE` and give it the value of `0.1 ether` (don't be scared by the floating number which technically doesn't work with Solidity - `0.1 ether` means `10 ** 17 ether`):

```solidity
address USER = makeAddr("user");
uint256 public constant SEND_VALUE = 0.1 ether;
```

Back to our test, update your test function `testFundUpdatesFundDataStructure()` in `FundMe.t.sol`:

```solidity
function testFundUpdatesFundDataStructure() public {
    vm.prank(USER);
    fundMe.fund{value: SEND_VALUE}();
    uint256 amountFunded = fundMe.getAddressToAmountFunded(USER);
    assertEq(amountFunded, SEND_VALUE);

}
```

Finally, now let's run `forge test --mt testFundUpdatesFundDataStructure`. It fails ... again!

But why? Let's call `forge test --mt testFundUpdatesFundDataStructure -vvv` to get more information about where and why it fails.

Turns out our `USER` address doesn't have any funds. But there's always a cheatcode to help us. `deal` allows us to set the ETH balance of a user. Read more about it [here](https://book.getfoundry.sh/cheatcodes/deal).

Now let's add `uint256 constant STARTING_BALANCE = 10 ether;` and add this line `vm.deal(USER, STARTING_BALANCE);` at the end of your `setUp()` function.

The whole file should now look like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {Test, console} from "forge-std/Test.sol";
import {FundMe} from "../src/FundMe.sol";
import {DeployFundMe} from "../script/DeployFundMe.s.sol";

contract FundMeTest is Test {
    address USER = makeAddr("user");
    uint256 public constant SEND_VALUE = 0.1 ether;
    uint256 constant STARTING_BALANCE = 10 ether;
    FundMe fundMe;
    DeployFundMe deployFundMe;

    function setUp() external {
        deployFundMe = new DeployFundMe();
        fundMe = deployFundMe.run();
        vm.deal(USER, STARTING_BALANCE);
    }

    function testMinimumDollarIsFive() public view {
        assertEq(fundMe.MINIMUM_USD(), 5e18);
    }

    function testOwnerIsMsgSender() public view {
        assertEq(fundMe.i_owner(), msg.sender);
    }

    function testPriceFeedVersionIsAccurate() public view {
        uint256 version = fundMe.getVersion();
        assertEq(version, 4);
    }

    function testFundFailsWIthoutEnoughETH() public {
        vm.expectRevert(); // <- The next line after this one should revert! If not test fails.
        fundMe.fund(); // <- We send 0 value
    }

    function testFundUpdatesFundDataStructure() public {
        vm.prank(USER);
        fundMe.fund{value: SEND_VALUE}();
        uint256 amountFunded = fundMe.getAddressToAmountFunded(USER);
        assertEq(amountFunded, SEND_VALUE);
    }
}
```

Let's run `forge test --mt testFundUpdatesFundDataStructure` again.

And now it passes. Congratulations 🥳!

Keep in mind that these are the most important cheatcodes there are, and you are going to use them over and over again. Regardless if you are developing or auditing a project, that project will always have at least an `owner` and a `user`. These two would always have different access to different functionalities. Most of the time the user needs some kind of balance, be it ETH or some other tokens. So, making a new address, giving it some balance, and pranking it to act as a caller for a tx will 100% be part of your every test file.

## Adding more coverage

In the previous lesson, we tested if the `s_addressToAmountFunded` is updated correctly. Continuing from there we need to test that `funders` array is updated with `msg.sender`.

Add the following test to your `FundMe.t.sol`:

```solidity
function testAddsFunderToArrayOfFunders() public {
    vm.startPrank(USER);
    fundMe.fund{value: SEND_VALUE}();
    vm.stopPrank();

    address funder = fundMe.getFunder(0);
    assertEq(funder, USER);
}
```

Run the test using `forge test --mt testAddsFunderToArrayOfFunders`. It passed, perfect!

Each of our tests uses a fresh `setUp`, so if we run all of them and `testFundUpdatesFundDataStructure` calls `fund`, that won't be persistent for `testAddsFunderToArrayOfFunders`.

Moving on, we should test the `withdraw` function. Let's check that only the owner can `withdraw`.

Add the following test to your `FundMe.t.sol`:

```solidity
function testOnlyOwnerCanWithdraw() public {
    vm.prank(USER);
    fundMe.fund{value: SEND_VALUE}();

    vm.expectRevert();
    vm.prank(USER);
    fundMe.withdraw();
}
```

**REMEMBER:** Whenever you have a situation where two or more `vm` cheatcodes come one after the other keep in mind that these would ignore one another. Cheatcodes affect transactions, not other cheatcodes.

Run the test using `forge test --mt testOnlyOwnerCanWithdraw` and it should pass.

As you can see, in both `testAddsFunderToArrayOfFunders()` and `testOnlyOwnerCanWithdraw()` we used `USER` to fund the contract, but when we're writing hundreds of tests, there's a better way.

Sometimes multiple tests will share some building blocks. We can define these building blocks using modifiers to dramatically increase our efficiency in writing tests.

Add the following modifier to your FundMe.t.sol:

```solidity
modifier funded() {
    vm.prank(alice);
    fundMe.fund{value: SEND_VALUE}();
    assert(address(fundMe).balance > 0);
    _;

}
```

We first use the `vm.prank` cheatcode to signal the fact that the next transaction will be called by `USER`. We call `fund` and then we assert that the balance of the `fundMe` contract is higher than 0, if true, it means that `USER`'s transaction was successful. Every single time we need our contract funded we can use this modifier to do it.

Now we can refactor `testOnlyOwnerCanWithdraw()`:

```solidity
function testOnlyOwnerCanWithdraw() public funded {
    vm.expectRevert();
    fundMe.withdraw();
}
```

Ok, we've tested that a non-owner cannot withdraw. But **can the owner withdraw**?

To test this we will need a new getter function. Add the following to the `FundMe.sol` file next to the other getter functions:

```solidity
function getOwner() public view returns (address) {
    return i_owner;
}
```

Make sure to make `i_owner` `private`.

The **arrange-act-assert (AAA) methodology** is one of the simplest and most universally accepted ways to write tests. As the name suggests, it comprises three parts:

- **Arrange**: Set up the test by initializing variables, and objects and prepping preconditions.
- **Act**: Perform the action to be tested like a function invocation.
- **Assert**: Compare the received output with the expected output.

We will start our test like so:

```solidity
function testWithdrawWithASingleFunder() public funded {
    // Arrange
    // Act
    // Assert
}
```

Now:

```solidity
function testWithdrawWithASingleFunder() public funded {
    // Arrange
    uint256 startingOwnerBalance = fundMe.getOwner().balance;
    uint256 startingFundMeBalance = address(fundMe).balance;

    // Act
    vm.prank(fundMe.getOwner());
    fundMe.withdraw();

    // Assert
    uint256 endingOwnerBalance = fundMe.getOwner().balance;
    uint256 endingFundMeBalance = address(fundMe).balance;
    assertEq(endingFundMeBalance, 0);
    assertEq(startingFundMeBalance + startingOwnerBalance, endingOwnerBalance);
}
```

Now we run `forge test --mt testWithdrawWithASingleFunder` and it works!

Let's do one last test called `testWithdrawWithMultipleFunders()`, this is the final code:

```solidity
function testWithdrawWithMultipleFunders() public funded {
    // Arrange
    uint160 numberOfFunders = 10;
    uint160 startingFunderIndex = 1;

    for (uint160 i = startingFunderIndex; i < numberOfFunders; i++) {
        // This is like vm.prank and vm.deal combined
        hoax(address(i), SEND_VALUE);
        fundMe.fund{value: SEND_VALUE}();
    }

    uint256 startingOwnerBalance = fundMe.getOwner().balance;
    uint256 startingFundMeBalance = address(fundMe).balance;

    // Act
    vm.startPrank(fundMe.getOwner());
    fundMe.withdraw();
    vm.stopPrank();

    // Assert
    assert(address(fundMe).balance == 0);
    assert(
        startingFundMeBalance + startingOwnerBalance ==
            fundMe.getOwner().balance
    );
}
```

Now if we run `forge test --mt testWithdrawWithMultipleFunders` it works again. Sweet!

If we run `forge coverage` the coverage looks a lot better now.

## Introduction to Foundry Chisel

`Chisel` is one of the 4 components of Foundry alongside `forge`, `cast` and `anvil`. It's a tool that allows users to quickly test the behavior of Solidity code on a local (anvil) or forked network. It's basically a `REPL` shell.

Here's the [Chisel Reference page](https://book.getfoundry.sh/reference/chisel/)

## Chisel to calculate withdraw gas costs

An important aspect of smart contract development is making your code efficient to minimize the gas you/other users spend when calling functions. This can have a serious impact on user retention for your protocol. Imagine you have to exchange 0.1 ETH for 300 USDC, but you have to pay 30 USDC in gas fees. No one wants to pay that. It's your duty as a developer to minimize gas consumption.

Now how do we find out how much gas things cost?

Run the following command in your terminal:

`forge snapshot --mt testWithdrawWithASingleFunder`

You'll see that a new file appeared in your project root folder: `.gas-snapshot`. When you open it you'll find the following:

`FundMeTest:testWithdrawWithASingleFunder() (gas: 90743)`

This means that calling that test function consumes `90743` gas. How do we find out what this means in \$?

Etherscan provides a cool tool that we can use: <https://etherscan.io/gastracker>. Today 13-MAR-2025 at 3:30pm CST, it says that the gas price is around `0.7 gwei`. If we multiply the two it gives us a total price of `63520 gwei`. Ok, at least that's an amount we can work with. Now we will use the handy [Alchemy converter](https://www.alchemy.com/gwei-calculator) to find out that `63520 gwei = 0.00006352 ETH` and `1 ETH = 1844.67 USD` according to [Coinmarketcap](https://coinmarketcap.com/) meaning that our transaction would cost `0.11 USD` on Ethereum mainnet. Let's see if we can lower this.

Looking closer at the `testWithdrawWithASingleFunder()` we can observe that we found out the initial balances, then we called a transaction and then we asserted that `startingFundMeBalance + startingOwnerBalance` matches the expected balance, but inside that test we called `withdraw` which should have cost gas. Why didn't the gas we paid affect our balances? Simple, for testing purposes the Anvil gas price is defaulted to `0` (different from what we talked about above in the case of Ethereum mainnet where the gas price was around `0.7 gwei`), so it wouldn't interfere with our testing.

Let's change that and force the `withdraw` transaction to have a gas price.

At the top of your `FundMe.t.sol` contract define the following variable:

`uint256 constant GAS_PRICE = 1;`

and refactor the `testWithdrawWithASingleFunder` function as follows:

```solidity
function testWithdrawWithASingleFunder() public funded {
    // Arrange
    uint256 startingFundMeBalance = address(fundMe).balance;
    uint256 startingOwnerBalance = fundMe.getOwner().balance;

    vm.txGasPrice(GAS_PRICE);
    uint256 gasStart = gasleft();

    // Act
    vm.startPrank(fundMe.getOwner());
    fundMe.withdraw();
    vm.stopPrank();

    uint256 gasEnd = gasleft();
    uint256 gasUsed = (gasStart - gasEnd) * tx.gasprice;
    console.log("Withdraw consumed: %d gas", gasUsed);

    // Assert
    uint256 endingFundMeBalance = address(fundMe).balance;
    uint256 endingOwnerBalance = fundMe.getOwner().balance;
    assertEq(endingFundMeBalance, 0);
    assertEq(
        startingFundMeBalance + startingOwnerBalance,
        endingOwnerBalance
    );

}
```

Let's review what we changed:

1. We used a new cheatcode called `vm.txGasPrice`, this sets up the transaction gas price for the next transaction. Read more about it [here](https://book.getfoundry.sh/cheatcodes/tx-gas-price).
2. We used `gasleft()` to find out how much gas we had before and after we called the transaction. Then we subtracted them to find out how much the `withdraw` transaction consumed. `gasleft()` is a built-in Solidity function that returns the amount of gas remaining in the current Ethereum transaction.
3. Then we logged the consumed gas. Read more about [Console Logging here](https://book.getfoundry.sh/reference/forge-std/console-log)

Let's run the test:

`forge test --mt testWithdrawWithASingleFunder -vv`

## Intro to Storage Optimization

Let's learn how we can optimize gas better now

By now you know that **storage** is that area within the blockchain were our contract's state variables and data live. Each storage space has a capacity of 32 bytes. Let's review this topic more in depth:

### Layout of State Variables in Storage

The [Solidity documentation](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html) is extremely good for understanding this subject.

These are the important aspects in a nutshell:

- Each storage slot has 32 bytes;
- The slots numbering starts from 0;
- Data is stored contiguously starting with the first variable placed in the first slot;
- Dynamically-sized arrays and mappings are treated differently (we'll discuss them below);
- The size of each variable, in bytes, is given by its type;
- If possible, multiple variables < 32 bytes are packed together;
- If not possible, a new slot will be started;
- Immutable and Constant variables are baked right into the bytecode of the contract, thus they don't use storage slots.

Then comes a practical example that I recommend you check directly at: https://updraft.cyfrin.io/courses/foundry/foundry-fund-me/solidity-storage-optimization?lesson_format=transcript.

To avoid confusion, remember that a `uint256` holds 256 bits or 32 bytes. So one `uint256` that takes all up its space would fill exactly one 32byte storage slot.

Finally, **mappings and Dynamic Arrays** can't be stored in between the state variables as we did above. That's because we don't quite know how many elements they would have. Without knowing that we can't mitigate the risk of overwriting another storage variable. The elements of mappings and dynamic arrays are stored in a different place that's computed using the Keccak-256 hash.

Back in our contracts, add the following getter in `FundMe.sol`:

```solidity
function getPriceFeed() public view returns (AggregatorV3Interface) {
    return s_priceFeed;

}
```

Then add the following function in your `FundMe.t.sol`:

```solidity
function testPrintStorageData() public view {
    for (uint256 i = 0; i < 3; i++) {
        bytes32 value = vm.load(address(fundMe), bytes32(i));
        console.log("Value at location", i, ":");
        console.logBytes32(value);
    }
    console.log("PriceFeed address:", address(fundMe.getPriceFeed()));

}
```

In the test above we used a new cheatcode: `vm.load`. Its sole purpose is to load the value found in the provided storage slot of the provided address. Read more about it [here](https://book.getfoundry.sh/cheatcodes/load).

Now run the test: `forge test --mt testPrintStorageData -vv`.

Let's interpret the resulting data:

- In `slot 0` we have a bytes32(0) stored (or 32 zeroes). This happened because the first slot is assigned to the `s_addressToAmountFunded` mapping.
- In `slot 1` we have a bytes32(0) stored. This happened because the second slot is assigned to the `s_funders` dynamic array.
- In `slot 2` we have `0x00000000000000000000000090193c961a926261b756d1e5bb255e67ff9498a1` stored. This is composed of 12 pairs of zeroes (12 x 00) corresponding to the first 12 bytes and `90193c961a926261b756d1e5bb255e67ff9498a1`. If you look on the next line you will see that is the `priceFeed` address.

This is one of the methods of checking the storage of a smart contract.

Another popular method is using `forge inspect`. This is used to obtain information about a smart contract. Read more about it [here](https://book.getfoundry.sh/reference/forge/forge-inspect).

Call the following command in your terminal:

`forge inspect FundMe storageLayout`

Another method of checking a smart contract's storage is by using `cast storage`. For this one, we need a bit of setup.

Open a new terminal, and type `anvil` to start a new `anvil` instance.

Deploy the `fundMe` contract using the following script:

`forge script DeployFundMe --rpc-url http://127.0.0.1:8545 --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 --broadcast`

The rpc url used is the standard `anvil` rpc. The private key used is the first of the 10 private keys `anvil` provides.

Find the address of the newly deployed `fundMe`. In my case, this is `0x5FbDB2315678afecb367f032d93F642f64180aa3`.

Call the following command in your terminal:

`cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 2`

This prints what's stored in slot number 2 of the `fundMe` contract. This checks with our previous methods of finding what's in slot number 2 (even if the address is different between methods).

Storage is one of the harder Solidity subjects. Mastering it is one of the key prerequisites in writing gas-efficient and tidy smart contracts.

## Optimizing the withdraw function, intro to Opcodes!

Let's keep exploring storage. Open a new terminal and start a new `anvil` instance:

On your original terminal deploy the `fundMe` contract:

`forge script DeployFundMe --rpc-url http://127.0.0.1:8545 --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 --broadcast`

Grab the contract address and run (replace the address with your deployed contract's address):

`cast code 0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9`.

You just got a big chunk of bytecode. Copy the entire thing and paste it [here](https://etherscan.io/opcode-tool), and so you obtain something that looks like this (a list of Opcodes):

```bash
[0] 'Cf'(Unknown Opcode)
[1] PUSH31 0xd3AccA5a467e9e704C703E8D87F634fB0Fc9 60806040526004361061007e
[2] JUMPI
[3] PUSH0 0x
[4] CALLDATALOAD
[5] PUSH1 0xe0
[6] SHR
[7] DUP1
[8] PUSH4 0x893d20e8
```

**Opcodes** are the fundamental instructions that the EVM understands and executes. These opcodes are essentially the building blocks that power smart contract functionality. You can read about each opcode [here](https://www.evm.codes/).

In that table alongside the description, you will find the bytecode number of each opcode, the name of the opcode, the minimum gas it consumes and the input/output. Please be mindful of the gas each opcode costs. Scroll down the list until you get to the 51-55 opcode range.

As you can see an MLOAD/MSTORE has a minimum gas cost of 3 and a SLOAD/SSTORE has a minimum gas of 100 ... that's over 33x. And keep in mind these are minimums. The difference is usually bigger. This is why we need to be careful with saving variables in storage, every time we access or modify them we will be forced to pay way more gas.

Let's take a closer look at the `withdraw` function.

We start with a `for` loop, that is initializing a variable called `funderIndex` in memory and compares it with `s_funders.length` on every loop iteration. As you know `s_funders` is the private array that holds all the funder's addresses, currently located in the state variables zone. If we have 1000 funders, we will end up reading the length of the `s_funders` array 1000 times, paying the SLOAD costs 1000 times. This is extremely inefficient.

Let's rewrite the function. Add the following to your `FundMe.sol`:

```solidity
function cheaperWithdraw() public onlyOwner {
    uint256 fundersLength = s_funders.length;
    for(uint256 funderIndex = 0; funderIndex < fundersLength; funderIndex++) {
        address funder = s_funders[funderIndex];
        s_addressToAmountFunded[funder] = 0;
    }
    s_funders = new address[](0);

    (bool callSuccess,) = payable(msg.sender).call{value: address(this).balance}("");
    require(callSuccess, "Call failed");

}
```

First, let's cache the `s_funders` length. This means we create a new variable, inside the function (to be read as in memory) so if we read it 1000 times we don't end up paying a ridiculous amount of gas.

Then let's integrate this into the for loop.

The next step is getting the funder's address from storage. Sadly we can't avoid this one. After this we zero the recorded amount in the `s_addressToAmountFunded` mapping, also we can't avoid this. We then reset the `s_funders` array, and send the ETH. Both these operations cannot be avoided.

Let's find out how much we saved. Open `FundMe.t.sol`.

Let's copy the `testWithdrawWithMultipleFunders` function and replace the `withdraw` function with `cheaperWithdraw`.

```solidity
function testWithdrawFromMultipleFundersCheaper() public funded {
    uint160 numberOfFunders = 10;
    uint160 startingFunderIndex = 1;
    for (uint160 i = startingFunderIndex; i < numberOfFunders + startingFunderIndex; i++) {
        // we get hoax from stdcheats
        // prank + deal
        hoax(address(i), SEND_VALUE);
        fundMe.fund{value: SEND_VALUE}();
    }

    uint256 startingFundMeBalance = address(fundMe).balance;
    uint256 startingOwnerBalance = fundMe.getOwner().balance;

    vm.startPrank(fundMe.getOwner());
    fundMe.cheaperWithdraw();
    vm.stopPrank();

    assert(address(fundMe).balance == 0);
    assert(startingFundMeBalance + startingOwnerBalance == fundMe.getOwner().balance);
    assert((numberOfFunders + 1) * SEND_VALUE == fundMe.getOwner().balance - startingOwnerBalance);

}
```

Now let's call `forge snapshot`. If we open `.gas-snapshot` we will find the following at the end:

```bash
[PASS] testWithdrawWithMultipleFunders() (gas: 510997)
[PASS] testWithdrawWithMultipleFundersCheaper() (gas: 560138)
```

In my case this didn't work.

// TODO: Find out why the gas optimization didn't work for me.

## Creating Integration Tests

First of all, don't forget to write a good `README.md`: A README is your project's face to the world, and investing time in making it clear, comprehensive, and engaging can significantly impact your project's success and community engagement.

### Integration Tests

Please create a new file called `Interactions.s.sol` in the `script` folder.

In this file, we will create two scripts, one for funding and one for withdrawing.

Each contract will contain one script, and for it to work each needs to inherit from the Script contract. Each contract will have a `run` function which shall be called by `forge script` when we run it.

In order to properly interact with our `fundMe` contract we would want to interact only with the most recent deployment we made. This task is easily achieved using the `foundry-devops` library. Please install it using the following command:

`forge install Cyfrin/foundry-devops --no-commit`

Ok, now with that out of the way, let's work on our scripts.

Put the following code in `Interactions.s.sol`:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.26;

import {Script, console} from "forge-std/Script.sol";
import {FundMe} from "../src/FundMe.sol";
import {DevOpsTools} from "foundry-devops/src/DevOpsTools.sol";

contract FundFundMe is Script {
    uint256 SEND_VALUE = 0.1 ether;

    function fundFundMe(address mostRecentlyDeployed) public {
        vm.startBroadcast();
        FundMe(payable(mostRecentlyDeployed)).fund{value: SEND_VALUE}();
        vm.stopBroadcast();
        console.log("Funded FundMe with %s", SEND_VALUE);
    }

    function run() external {
        address mostRecentlyDeployed = DevOpsTools.get_most_recent_deployment("FundMe", block.chainid);
        fundFundMe(mostRecentlyDeployed);
    }
}

contract WithdrawFundMe is Script {
    function withdrawFundMe(address mostRecentlyDeployed) public {
        vm.startBroadcast();
        FundMe(payable(mostRecentlyDeployed)).withdraw();
        vm.stopBroadcast();
        console.log("Withdraw FundMe balance!");
    }

    function run() external {
        address mostRecentlyDeployed = DevOpsTools.get_most_recent_deployment("FundMe", block.chainid);
        withdrawFundMe(mostRecentlyDeployed);
    }

}
```

Integration tests are crucial for verifying how your smart contract interacts with other contracts, external APIs, or decentralized oracles that provide data feeds. These tests help ensure your contract can properly receive and process data, send transactions to other contracts, and function as intended within the wider ecosystem.

Before starting with the integration tests let's organize our tests into folders.

Create two new folders called integration and unit inside the test folder. Move `FundMe.t.sol` inside the `unit` folder. Update the imports in `FundMe.t.sol`.

Run a quick `forge test` to ensure that everything builds and all tests pass.

Inside the integration folder create a new file called `FundMeTestIntegration.t.sol`.

Paste the following code inside it:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

import {DeployFundMe} from "../../script/DeployFundMe.s.sol";
import {FundFundMe, WithdrawFundMe} from "../../script/Interactions.s.sol";
import {FundMe} from "../../src/FundMe.sol";
import {Test, console} from "forge-std/Test.sol";

contract InteractionsTest is Test {
    FundMe public fundMe;
    DeployFundMe deployFundMe;

    uint256 public constant SEND_VALUE = 0.1 ether;
    uint256 public constant STARTING_USER_BALANCE = 10 ether;

    address alice = makeAddr("alice");


    function setUp() external {
        deployFundMe = new DeployFundMe();
        fundMe = deployFundMe.run();
        vm.deal(alice, STARTING_USER_BALANCE);
    }

    function testUserCanFundAndOwnerWithdraw() public {
        uint256 preUserBalance = address(alice).balance;
        uint256 preOwnerBalance = address(fundMe.getOwner()).balance;

        // Using vm.prank to simulate funding from the USER address
        vm.prank(alice);
        fundMe.fund{value: SEND_VALUE}();

        WithdrawFundMe withdrawFundMe = new WithdrawFundMe();
        withdrawFundMe.withdrawFundMe(address(fundMe));

        uint256 afterUserBalance = address(alice).balance;
        uint256 afterOwnerBalance = address(fundMe.getOwner()).balance;

        assert(address(fundMe).balance == 0);
        assertEq(afterUserBalance + SEND_VALUE, preUserBalance);
        assertEq(preOwnerBalance + SEND_VALUE, afterOwnerBalance);
    }

}
```

You will see that the first half, including the `setUp` is similar to what we did in `FundMe.t.sol`. The test `testUserCanFundAndOwnerWithdraw` has a similar structure to `testWithdrawWithASingleFunder` from `FundMe.t.sol`. We record the starting balances, we use `alice` to fund the contract then the `WithdrawFundMe` script to call withdraw. The next step is recording the ending balances and running the same assertions we did in `FundMe.t.sol`.

Run the integration test using the following command:

`forge test --mt testUserCanFundAndOwnerWithdraw -vv`

Pfew! I know this was a lot. You are a true champion 🏆🏆🏆 for reaching this point!

Note 1: Depending on when you go through this lesson there is a small chance that foundry-devops library has a problem that prevents you from building. The reason this happening is vm.keyExists used at foundry-devops/src/DevOpsTools.sol:119 is deprecated. Please replace vm.keyExists with vm.keyExistsJson in the place indicated. Next, we need to make sure that the Vm.sol contract in your forge-std library contains the vm.keyExistsJson. If you can't find it in your Vm.sol then please run the following command in your terminal: forge update --force. If you still can't forge build the project the please come ask questions in the Updraft section of Cyfrin's discord.

Note2:

Inside the video lesson, Patrick touched on the subject of ffi. We didn't present it at length in the body of this lesson because foundry-devops doesn't need it anymore. But in short:

Forge FFI, which stands for Foreign Function Interface, is a cheatcode within the Forge testing framework for Solidity. It allows you to execute arbitrary shell commands directly from your Solidity test code.

FFI enables you to call external programs or scripts from within your Solidity tests.

You provide the command or script name along with any arguments as an array of strings.

The Forge testing framework then executes the command in the underlying system environment and captures the output.

Read more about it [here](https://book.getfoundry.sh/cheatcodes/ffi?highlight=ffi#ffi).

A word of caution: FFI bypasses the normal security checks and limitations of Solidity. By running external commands, you introduce potential security risks if not used carefully. Malicious code within the commands you execute could compromise your setup. Whenever you clone repos or download other projects please make sure they don't have ffi = true in their foundry.toml file. If they do, we advise you not to run anything before you thoroughly examine where ffi is used and what commands is it calling. Stay safe!

## Automate smart contract actions with a Makefile

If you think a bit about your experience with the whole `FundMe` project by now, how many times have you written a `forge script NameOfScript --rpc-url xyz --private-key 0xPrivateKey ...`. There's got to be an easier way to run scripts and other commands.

The answer for all your troubles is a `Makefile`!

A `Makefile` is a special file used in conjunction with the `make` command in Unix-based systems and some other environments. It provides instructions for automating the process of building software projects.

The main advantages of using a `Makefile` are:

- Automates tasks related to building and deploying your smart contracts.
- Integrates with Foundry commands like `forge build`, `forge test` and `forge script`.
- Can manage dependencies between different smart contract files.
- Streamlines the development workflow by reducing repetitive manual commands.
- Allows you to automatically grab the `.env` contents.

In the root folder of your project create a new file called `Makefile`.

After creating the file run `make` in your terminal.

If you have `make` installed then you should receive the following message:

(I had to install it with `sudo apt install make`).

`make: *** No targets.  Stop`

Let's start our `Makefile` with `-include .env` on the first line. This way we don't have to call `source .env` every time we want to access something from it.

Soo... how do we actually write a shortcut?

Let's write one for `forge build`.

In your `Makefile` write the following line:

`build:; forge build`

Run `make build` in your terminal.

And it works! We've written our first shortcut. Arguably not the best of shortcuts, we've saved 1 letter, but still, it's a start.

Let's write a more complex shortcut. Add the following shortcut to your `Makefile`:

```make
deploy-sepolia:

    forge script script/DeployFundMe.s.sol:DeployFundMe --rpc-url $(SEPOLIA_RPC_URL) --private-key $(SEPOLIA_PRIVATE_KEY) --broadcast --verify --etherscan-api-key $(ETHERSCAN_API_KEY) -vvvv
```

I had to run `forge clean` and ask chatGPT to correct my `Makefile` which had spaces instead of tabs, here's my correct `Makefile`:

```make
-include .env

build:
    forge build

deploy-sepolia:
    forge script script/DeployFundMe.s.sol:DeployFundMe --rpc-url $(SEPOLIA_RPC_URL) --private-key $(PRIVATE_KEY) --broadcast --verify --etherscan-api-key $(ETHERSCAN_API_KEY) -vvvv
```

Aaand everything worked, except the verification.

// TODO: Check why verification didn't work.

My output:

```bash
ONCHAIN EXECUTION COMPLETE & SUCCESSFUL.
##
Start verification for (1) contracts
Start verifying contract `0x7F080196962aD0c85f068b853AA3468Fd5D17Db7` deployed on sepolia
Compiler version: 0.8.26
Constructor args: 000000000000000000000000694aa1769357215de4fac081bf1f309adc325306

Submitting verification for [src/FundMe.sol:FundMe] 0x7F080196962aD0c85f068b853AA3468Fd5D17Db7.
Warning: Etherscan could not detect the deployment.; waiting 5 seconds before trying again (4 tries remaining)

Submitting verification for [src/FundMe.sol:FundMe] 0x7F080196962aD0c85f068b853AA3468Fd5D17Db7.
Warning: Etherscan could not detect the deployment.; waiting 5 seconds before trying again (3 tries remaining)

Submitting verification for [src/FundMe.sol:FundMe] 0x7F080196962aD0c85f068b853AA3468Fd5D17Db7.
Warning: Etherscan could not detect the deployment.; waiting 5 seconds before trying again (2 tries remaining)

Submitting verification for [src/FundMe.sol:FundMe] 0x7F080196962aD0c85f068b853AA3468Fd5D17Db7.
Warning: Etherscan could not detect the deployment.; waiting 5 seconds before trying again (1 tries remaining)

Submitting verification for [src/FundMe.sol:FundMe] 0x7F080196962aD0c85f068b853AA3468Fd5D17Db7.
Warning: Etherscan could not detect the deployment.; waiting 5 seconds before trying again (0 tries remaining)

Submitting verification for [src/FundMe.sol:FundMe] 0x7F080196962aD0c85f068b853AA3468Fd5D17Db7.
Error: Failed to verify contract: Etherscan could not detect the deployment.

Transactions saved to: /home/garosan/cyfrin-foundry/fund-me/broadcast/DeployFundMe.s.sol/11155111/run-latest.json

Sensitive values saved to: /home/garosan/cyfrin-foundry/fund-me/cache/DeployFundMe.s.sol/11155111/run-latest.json

Error: Not all (0 / 1) contracts were verified!
make: *** [Makefile:7: deploy-sepolia] Error 1
```

This is just an introductory lesson on how to write Makefiles. Properly organizing your scripts and then transforming them into shortcuts that save you from typing 3 lines of code in the terminal is an ART!

## Pushing to Github



## ❓ Questions and 💪 Exercises

- Question ❓: What is this AggregatorV3Interface.sol file? What exactly does it do?
- Question ❓: What are the differences between using a `require` and a `revert` in our `onlyOwner()` modifier?
- Question ❓: You added some console.logs to your tests, you run `forge test` and you don't see them. What is missing?
- Question ❓: Why did we have to use a `uint160` when passing a number to generate an address in our tests?

## 🛠️ Links and Resources

- [Course Repo](https://github.com/Cyfrin/foundry-fund-me-cu)
- [FunWithStorage Contract](https://github.com/Cyfrin/foundry-fund-me-cu/blob/main/src/exampleContracts/FunWithStorage.sol)
- [Forge install](https://book.getfoundry.sh/reference/cli/forge/install)
- [Forge test](https://book.getfoundry.sh/reference/cli/forge/test)
- [Forge console.log](https://book.getfoundry.sh/reference/forge-std/console-log)
- [Forge coverage](https://book.getfoundry.sh/reference/forge/forge-coverage)
- [Foundry Cheatcodes](https://book.getfoundry.sh/cheatcodes/)