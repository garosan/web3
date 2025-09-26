# Section 1. Foundry Simple Storage

## Dev Environment - Windows

For Windows, it all starts with running `wsl --install` on Powershell.

## Why Foundry?

Foundry Pros:

- It **leverages Rust** for compilation, offering significantly faster build times compared to frameworks like Hardhat or Brownie.
- It's entirely **Solidity-based**, eliminating the need to learn other programming languages

## Foundry Setup

Once WSL and VS Code are installed and everything is working let's install Foundry with:

`curl -L https://foundry.paradigm.xyz | bash`

Now let's run `foundryup`.

This will install four components: forge, cast, anvil, and chisel. To confirm the successful installation, run `forge --version`, `cast --version`, `anvil --version`, `chisel --version`.

## New Foundry Project

Run `mkdir foundry-simple-storage` and `cd foundry-simple-storage`.

To start a new project, run `forge init` inside the empty directory.

Keep in mind that by default `forge init` expects an empty folder. If your folder is not empty you must run `forge init --force .`

## Understanding Foundry tree structure

- `lib` is the folder where all your dependencies are installed. The default template comes with one dependency installed: Forge Standard Library. This is the preferred testing library used for Foundry projects.
- `script` is a folder that houses all your scripts
- `src` is the folder where you put all your smart contracts
- `test` is the folder that houses all your tests
- `foundry.toml` - gives configuration parameters for Foundry

- Run `forge build` to build the project
- Run `forge test` to run the tests

You'll notice that two new directories have popped up: `out` and `cache`.

The `out` directory contains your contract artifact, such as the ABI, while the `cache` is used by forge to only recompile what is necessary.

## Continue with the exercise

Delete all the `Counter` Solidity files inside `script`, `src` and `test`. Create `src/SimpleStorage.sol` and add the contents from [this](https://github.com/Cyfrin/foundry-simple-storage-cu/blob/main/src/SimpleStorage.sol) file.

## `compile` and `build`

Difference between `forge compile` and `forge build` commands.

---

### Key Differences:

| Feature                      | `forge compile`              | `forge build`                        |
| ---------------------------- | ---------------------------- | ------------------------------------ |
| **Focus**                    | Basic compilation            | Comprehensive build process          |
| **Includes Pre-Build Steps** | No                           | Yes (if specified in `foundry.toml`) |
| **Performance**              | Faster, minimal              | Potentially slower, complete         |
| **Use Case**                 | Development and quick checks | Deployment and final builds          |

### Recommendation:

- Use `forge compile` for quick iterative development.
- Use `forge build` when preparing for deployment, running a full suite of tests, or ensuring all configurations and optimizations are included.

## Introduction to Anvil

Anvil is a local testnet node shipped with Foundry. You can use it for testing your contracts from frontends or for interacting over RPC.

To run Anvil you simply have to type `anvil` in the terminal.

This testnet node always listens on `127.0.0.1:8545` this will be our `RPC_URL` parameter when we deploy smart contracts here. More on this later!

- Read more on [Anvil](https://book.getfoundry.sh/reference/anvil/)

## Deploy a smart contract locally

You can read [here](https://book.getfoundry.sh/reference/forge/forge-create) all the options to deploy using `forge create`.

Try running `forge create SimpleStorage`. It should fail because we haven't specified a couple of required parameters:

1. `Where do we deploy?`
2. `Who's paying the gas fees/signing the transaction?`

## Deploy locally using Anvil

- Run `anvil` in one terminal

If you are working with Anvil you can skip the rpc url:

> **THIS IS JUST FOR DEMO AND EDUCATION PURPOSES**

**NEVER PASTE THE PRIVATE KEY, i.e. NEVER DO THIS:**

`forge create SimpleStorage --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80`

> **The above is anvil's first test private key, it's a dummy private key**

## Using `--interactive`

`forge create SimpleStorage --interactive`

This will prompt us for our private key. I'll take the first of the ones given by Anvil, paste it (nothing actually shows but you click enter) and you get the following back:

`Warning: Dry run enabled, not broadcasting transaction`

Plus a bunch of information about the transaction, including the bytecode and the ABI. But remember the warning:

`Warning: To broadcast this transaction, add --broadcast to the previous command`

## What is dry-run?

When you ran the above command you got `Warning: Dry run enabled, not broadcasting transaction` and you didn't get a contract address, that's because the deployment was simulated.

A dry-run is a simulation mode to allow you to preview the outcome of the deployment. This is good for several reasons:

- Safety: It ensures that you can verify the deployment parameters, like gas estimates, constructor arguments, and deployment costs.
- Testing: Verify that your deployment setup works correctly without spending gas on a live network.
- Configuration Validation: Helps catch misconfigurations before real deployment.

## Actually deploying your contract

To deploy your contract add `--broadcast` as advised:

`forge create SimpleStorage --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 --broadcast`

> **The above is anvil's first test private key, it's a dummy private key**

And you get:

```
Deployer: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Transaction hash: 0xe933149cbe9c0958a21367bbf2f30a26dcdb497bcb22da9642c2059a71907cf8
```

Just as a curiosity if you ran `cast block-number` when you started your project you got `0` if you run it now you get `1`.

## Deleting your history

You can delete your bash history by typing `history -c`.

## 💪 Extra exercises and information

Run `cast code 0x5FbDB2315678afecb367f032d93F642f64180aa3 --rpc-url http://127.0.0.1:8545`

Let's now interact with this contract:

1. Store a value in SimpleStorage:

`cast send 0x5FbDB2315678afecb367f032d93F642f64180aa3 "store(uint256)" 42 --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80`

Notice that sometimes I add `--rpc-url http://127.0.0.1:8545 ` and sometimes I omit it, both should yield the same result.

Now retrieve the stored value:

`cast call <CONTRACT_ADDRESS> "retrieve() (uint256)" --rpc-url http://127.0.0.1:8545` or:

`cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 "retrieve() (uint256)"`

Notice we didn't need the private key for that last one, you can tell why, right?

### More `anvil` stuff

If you shut down your anvil and then try this `anvil --block-time 5` you'll notice it now mines a new block every 5 seconds.

Send a transaction, notice it will take 5 seconds to be confirmed:

`cast send 0x5FbDB2315678afecb367f032d93F642f64180aa3 "store(uint256)" 50 --rpc-url http://127.0.0.1:8545 --private-key <PRIVATE_KEY>`

### ADVANCED: Forking Mainnet and Testing Real Data

Run Anvil in fork mode:

`anvil --fork-url https://mainnet.infura.io/v3/<YOUR_INFURA_API_KEY>`

Impersonate Vitalik's account:

`cast rpc anvil_impersonateAccount 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url http://127.0.0.1:8545`

Send ETH from the impersonated account:

`cast send 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 --value 432ether --from 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --unlocked --rpc-url http://127.0.0.1:8545`

Verify your new balance:

`cast balance <YOUR_WALLET> --rpc-url http://127.0.0.1:8545`

Stop impersonating:

`cast rpc anvil_stopImpersonatingAccount 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url http://127.0.0.1:8545`

## Deploy using a script

In Foundry we keep our scripts in the `script` folder.

Please create a new file called `DeploySimpleStorage.s.sol`. Using `.s.sol` as a suffix is a naming convention for Foundry scripts.

Read the [best practices](https://book.getfoundry.sh/tutorials/best-practices#scripts) for Foundry scripts.

`DeploySimpleStorage.s.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {Script} from "forge-std/Script.sol";
import {SimpleStorage} from "../src/SimpleStorage.sol";

contract DeploySimpleStorage is Script {
    function run() external returns (SimpleStorage) {
        vm.startBroadcast();

        SimpleStorage simpleStorage = new SimpleStorage();

        vm.stopBroadcast();
        return simpleStorage;
    }
}
```

Make sure Anvil isn't running and run `forge script script/DeploySimpleStorage.s.sol`.

You should get this output:

```bash
[⠆] Compiling...
[⠔] Compiling 2 files with 0.8.19
[⠒] Solc 0.8.19 finished in 1.08s
Compiler run successful!
Script ran successfully.
Gas used: 338569

== Return ==
0: contract SimpleStorage 0x90193C961A926261B756D1E5bb255e67ff9498A1

If you wish to simulate on-chain transactions pass a RPC URL.
```

If you get an error from a missing `Counter.sol` file or similar in your `artifacts` folder, run `forge clean` and then run your script again.

If we didn't pass an RPC URL, where did this deploy to? If no RPC URL is specified, Foundry automatically launches an Anvil instance, runs your script and terminates the Anvil instance.

Now run an Anvil instance and in a new terminal type:

`forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545`

You will get the following output:

```bash
No files changed, compilation skipped
EIP-3855 is not supported in one or more of the RPCs used.
Unsupported Chain IDs: 31337.
Contracts deployed with a Solidity version equal or higher than 0.8.20 might not work properly.
For more information, please see https://eips.ethereum.org/EIPS/eip-3855
Script ran successfully.

== Return ==
0: contract SimpleStorage 0x34A1D3fff3958843C43aD80F30b94c510645C316

## Setting up 1 EVM.

==========================

Chain 31337

Estimated gas price: 2 gwei

Estimated total gas used for script: 464097

Estimated amount required: 0.000928194 ETH

==========================

SIMULATION COMPLETE. To broadcast these transactions, add --broadcast and wallet configuration(s) to the previous command. See forge script --help for more.
```

Emphasis on the _SIMULATION COMPLETE._. Is it deployed now?

No, the output indicates this was a simulation. But, we got a new folder out of this, the broadcast folder.

Now **to actually deploy it locally** run:

`forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80`

In the Anvil terminal, you'll see:

```bash
Transaction: 0xedae1838f9887073ad402ab58227fbff9fc3ac3496da14470f4a305535cd3ac4
Contract created: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Gas used: 341922

Block Number: 1
Block Hash: 0x2472ac00b89c5894fcbd92ccce0dd96530df256061f2c33118b6b23bda660cd8
Block Time: "Sat, 21 Dec 2024 07:40:11 +0000"
```

So just to reiterate:

- When you run `forge script script/DeploySimpleStorage.s.sol` with Anvil not running, Foundry automatically spins up a temporary in-memory blockchain (Anvil) just for the script execution. The contract deployment is simulated and no actual blockchain state is modified.

- When you run `forge script script/DeploySimpleStorage.s.sol` with Anvil running, Foundry still runs the script in a simulated mode and no contract is deployed but we get a `broadcast/` folder.

- When you run `forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key <PRIVATE_KEY>` Foundry connects to the running Anvil instance and actually sends the deployment transaction.

## What is a transaction

What you need to remember here is that a `broadcast` folder was created. Look inside in the `run-latest.json` file and you'll see this:

```json
  "transactions": [
    {
      "hash": "0xedae1838f9887073ad402ab58227fbff9fc3ac3496da14470f4a305535cd3ac4",
      "transactionType": "CREATE",
      "contractName": "SimpleStorage",
      "contractAddress": "0x5fbdb2315678afecb367f032d93f642f64180aa3",
      "function": null,
      "arguments": null,
      "transaction": {
        "from": "0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266",
        "gas": "0x6c852",
        "value": "0x0",
        "input": "0x60806...0033",
        "nonce": "0x0",
        "chainId": "0x7a69"
      },
      "additionalContracts": [],
      "isFixedGasLimit": false
    }
  ],
```

**QUICK TIP**: To read this `"gas": "0x6c852"` in a human-readable way you can do this in the terminal:

`cast --to-base 0x6c852 dec`

## To use an `.env` file

Create your `.env` file. **Remember to check that it's in your .gitignore!!**

Next run `source .env`. This adds the above-mentioned environment variables into our shell.

Now you can run your commands in the terminal like:

`some command --cmd $MY_ENV_VAR1 $MY_ENV_VAR2`

## 🚨 PRIVATE KEY SAFETY 🚨

But now you would have your private keys in plain text in your `.env` file.

Foundry has a very nice option called `keystore`. Using `forge script --keystore <PATH>` allows you to specify a path to an encrypted store file, encrypted by a password. Thus your private key would never be available in plain text.

Let's agree to the following:

1. For testing purposes use a `$PRIVATE_KEY` in an `.env` file as long as you don't expose that `.env` file anywhere.
2. Where real money is involved use the `--interactive` option or a [keystore file protected by a password](https://github.com/Cyfrin/foundry-full-course-f23?tab=readme-ov-file#can-you-encrypt-a-private-key---a-keystore-in-foundry-yet).

## 🚨 PRIVATE KEY SAFETY UPDATE 🚨

You should never use a `.env` again, but instead **encrypt your Keys Using ERC2335**

For this example we will use `0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80` (key 0 from Anvil).

Type this in your terminal (not inside VS Code):

`cast wallet import nameOfAccountGoesHere --interactive`

You will be asked for your private key and a password to secure it. You will do this only once.

## Interacting with a smart contract using the CLI

Now we have `anvil` running and the smart contract is deployed. Copy the contract address.

We will use `cast send` to interact with our deployed contract:

`cast send 0xcontractAddress "store(uint256)" 1337 --rpc-url $RPC_URL --private-key $PRIVATE_KEY`

Where `"store(uint256)"` is the signature of the function we're calling and `1337` is the argument passed to `store()`.

If you'd like to call the retrieve() function to read the number you'd do:

`cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 "retrieve()"`

You will receive back the value in hex format. To convert do:

`cast --to-base 0x0000000000000000000000000000000000000000000000000000000000000539 dec`

## Deploying our contract to a testnet or mainnet

For this exercise, this will be fairly easy, we will go the `.env` file route and replace the values with our actual private key with testnet funds and our RPC URL from Infura or Alchemy.

`forge script script/DeploySimpleStorage.s.sol --rpc-url $SEPOLIA_RPC_URL --broadcast --private-key $PRIVATE_KEY`

## Verify your smart contract on Etherscan

Follow the instructions from here to verify your contract on Etherscan:

[Link](https://updraft.cyfrin.io/courses/foundry/foundry-simple-storage/verify-smart-contract-etherscan)

Note: This is the manual way to verify your contract on Etherscan but it's prone to errors and not very recommended. We will learn to verify a contract programmatically in the next lessons.

## Verifying programatically

You could also just deploy and verify your contract using Foundry's tools running the command like this:

```solidity
forge script script/DeploySimpleStorage.s.sol:DeploySimpleStorage \
  --rpc-url $SEPOLIA_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --verify \
  --chain sepolia \
  --etherscan-api-key $ETHERSCAN_API_KEY
```

## Cleaning up the project

Forge has a built-in command to format our code that we should always run:

`forge fmt`

Finally, include a `README.md` file. This file serves as a guide for anyone who wants to learn about your project, how to use it, or how to contribute to it.

## 🔎 Testing

At the very, very minimum, get used to ask chatGPT to write you some tests for your contract:

Create a `test/SimpleStorageTest.t.sol` file.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import "forge-std/Test.sol";
import "../src/SimpleStorage.sol";

contract SimpleStorageTest is Test {
    SimpleStorage public simpleStorage;

    function setUp() public {
        // Deploy the contract before each test
        simpleStorage = new SimpleStorage();
    }

    function test_InitialFavoriteNumber_ShouldBeZero() public view {
        uint256 storedValue = simpleStorage.retrieve();
        assertEq(storedValue, 0, "Initial favorite number should be 0");
    }

    function test_StoreNumber_ShouldUpdateValue() public {
        simpleStorage.store(42);
        uint256 storedValue = simpleStorage.retrieve();
        assertEq(storedValue, 42, "Stored number should be 42");
    }

    function test_AddPerson_ShouldStoreInList() public {
        simpleStorage.addPerson("Alice", 7);
        (uint256 storedNumber, string memory storedName) = simpleStorage
            .listOfPeople(0);
        assertEq(storedNumber, 7, "Stored favorite number should be 7");
        assertEq(storedName, "Alice", "Stored name should be Alice");
    }

    function test_AddPerson_ShouldMapNameToNumber() public {
        simpleStorage.addPerson("Bob", 99);
        uint256 storedNumber = simpleStorage.nameToFavoriteNumber("Bob");
        assertEq(storedNumber, 99, "Mapping should return correct number");
    }
}
```

Run `forge test` and everything should pass.

## ❓ Questions and 💪 Exercises

Question ❓: What are the different mining modes in `Anvil`?

R.- By default, a new block is generated everytime a new transaction is submitted. You can change this to interval mining with `--block-time <block-time-in-seconds>` or to _never_ with `anvil --no-mining` and generate the blocks on demand.

## 🛠️ Links and Resources

- [Foundry Docs](https://book.getfoundry.sh/)
- [Course Repo](https://github.com/Cyfrin/Updraft/tree/main/courses/foundry)
- [VS Code Key Bindings](https://code.visualstudio.com/docs/getstarted/keybindings)
- [FreeCodeCamp - VS Code Crash Course](https://www.youtube.com/watch?v=WPqXP_kLzpo)
- [The Linux command line for beginners](https://ubuntu.com/tutorials/command-line-for-beginners#1-overview)

Extensions

- [Solidity by Nomic Foundation](https://marketplace.visualstudio.com/items?itemName=NomicFoundation.hardhat-solidity)
- [Even Better TOML](https://marketplace.visualstudio.com/items?itemName=tamasfe.even-better-toml)
- [Inline Bookmarks](https://marketplace.visualstudio.com/items?itemName=tintinweb.vscode-inline-bookmarks)

Compromised private key hacks

- [The Ronin hack](https://www.halborn.com/blog/post/explained-the-ronin-hack-march-2022)
- [Chinese investor loses $42M](https://cointelegraph.com/news/chinese-vc-loses-42m-in-crypto-due-to-compromised-mnemonic-seed-phrase)
- [The $477 million FTX hack](https://www.elliptic.co/blog/the-477-million-ftx-hack-following-the-blockchain-trail)

More Links

- [Scripting best practices](https://getfoundry.sh/guides/scripting-with-solidity)
- [Solidity scripting](https://book.getfoundry.sh/tutorials/solidity-scripting)
- [Cast Reference](https://book.getfoundry.sh/reference/cast/cast)
