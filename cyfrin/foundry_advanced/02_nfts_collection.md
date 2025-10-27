# # Section 02. Develop an NFTs Collection

## Introduction

2 implementations:

- 1st, basic, collection of puppies stored in IPFS.
- 2nd, art will be stored on-chain, dynamic, changing on a criteria we set.

## What is an NFT

At their core, NFTs are tokens which exist on a smart contract platform, they can be viewed, bought and sold on marketplaces such as **[OpenSea](https://opensea.io/)** and **[Rarible](https://rarible.com/)**.

### ERC 721 Standard

NFTs, and the ERC721 Token Standard, differ from ERC20s in a few fundamental ways.

ERC20s handle ownership via a simple mapping of a uint256 token balance to an address.

ERC721s, by contrast, each have a unique tokenId, these tokenIds are mapped to a user's address. In addition to a tokenId, ERC721s include a tokenUri.

For art use cases of NFTs, people used to include all the metadata (like the image of the NFT) on the blockchain which was extremely expensive. So a tokenUri was added to the standard so that you could just put the URL string of the associated image and some other metadata on the blockchain, this obviously becomes a very centralized point of failure.

Often a protocol will use a service like **[IPFS](https://ipfs.tech/)** to store their images and metadata in a more decentralized way.

## Project Setup

```bash
mkdir nft-project
cd nft-project
forge init
```

- Remove `src/Counter.sol`, `test/Counter.t.sol` and `script/Counter.s.sol`.
- Make sure that your `.gitignore` contains `.env` and `broadcast/`.
- Create `src/BasicNft.sol`.

Install OpenZeppelin contracts:

`forge install OpenZeppelin/openzeppelin-contracts --no-commit`

Add the remapping in `foundry.toml`:

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
remappings = ["@openzeppelin/contracts=lib/openzeppelin-contracts/contracts"]
```

Import and inherit the ERC721.

### TokenURI

Hard to believe, but when NFTs began, `tokenUri` was an optional parameter, despite being integral to how NFTs are used and consumed today. TokenURI basically is an endpoint that returns the metadata for a given NFT.

Example schema:

```json
{
  "title": "Asset Metadata",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "Identifies the asset to which this NFT represents"
    },
    "description": {
      "type": "string",
      "description": "Describes the asset to which this NFT represents"
    },
    "image": {
      "type": "string",
      "description": "A URI pointing to a resource with mime type image/* representing the asset to which this NFT represents. Consider making any images at a width between 320 and 1080 pixels and aspect ratio between 1.91:1 and 4:5 inclusive."
    }
  }
}
```

As an exercise, we can check the PudgyPenguins contract on Etherscan [here](https://etherscan.io/address/0xbd3531da5cf5857e7cfaa92426877b022e612cf8#readContract).

Go to the `tokenURI` function and enter any valid `tokenId`. This will return a string like:

`ipfs://bafybeibc5sgo2plmjkq2tzmhrn54bk3crhnc23zd2msg4ea7a4pxrkgfna/1000`

To see this in Chrome replace `ipfs://` with `https://ipfs.io/ipfs/`:

`https://ipfs.io/ipfs/bafybeibc5sgo2plmjkq2tzmhrn54bk3crhnc23zd2msg4ea7a4pxrkgfna/1000`.

Enter the above URL in Chrome and you'll see the metadata for this NFT. Once again, convert the image URL that you see:

`https://ipfs.io/ipfs/QmNf1UsmdGaMbpatQ6toXSkzDpizaGmC9zfunCyoz1enD5/penguin/1000.png`

And voilá, you'll see the image for that particular NFT, it should be the same as the one listed on OpenSea.

Both the `tokenUri` and `imageUri` for this PudgyPenguin are stored in IPFS. We will also use IPFS to store our DOGI NFTs.

### Our Code

So anytime someone mints a Doggie NFT, we need to assign a TokenURI to the minted TokenID.

The OZ implementation of ERC721 that we imported has its own virtual tokenURI function which we will override.

Let's add our own `tokenURI` function, our `BasicNft.sol` is looking like this:

```solidity
// SPDX-License-Identifier: MIT

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

pragma solidity 0.8.26;

contract BasicNft is ERC721 {
    uint256 private s_tokenCounter;

    constructor() ERC721("Doggie", "DOGI") {
        s_tokenCounter = 0;
    }

    function tokenURI(
        uint256 tokenId
    ) public view override returns (string memory) {}
}
```

Now let's create a folder called `img` in our root. I will add 4 DOGI images I created with ChatGPT.

## Intro to IPFS

IPFS is a decentralized storage protocol, it doesn't do any computation. It basically works like this:

We provide our data to the IPFS network via a node, a hash is produced that points to the location and details of the data and also serves to verify the data's integrity.

You can check more info in these links:

- [IPFS](https://ipfs.tech/)
- [IPFS Docs](https://docs.ipfs.tech/)
- [IPFS Project explanation](https://iq.wiki/wiki/ipfs)

Let's download the Desktop IPFS application to start using it.

After downloading the Desktop app I uploaded my DOGI images, and I immediatly get a CID for all of them. I can now view them in my browser:

`https://ipfs.io/ipfs/QmTx1tZNCySh1Q5Z6UNwhrjYZZJETbQLcvHFkvSjENiZzS`

## TokenURI function

Ok, now check this tokenURI provided by Cyfrin:

```json
{
  "name": "PUG",
  "description": "An adorable PUG pup!",
  "image": "https://ipfs.io/ipfs/QmSsYRx3LpDAb1GZQm7zZ1AuHZjfbPkD6J7s9r41xu1mf8?filename=pug.png",
  "attributes": [
    {
      "trait_type": "cuteness",
      "value": 100
    }
  ]
}
```

We could use this as the return value for our `tokenUri` function but that would mean every DOGI minted would be identical to each other. So instead, we will allow the user to pass a tokenUri (a json object) to the mint function and map this URI to their minted tokenId:

```solidity
contract BasicNft is ERC721 {
    uint256 private s_tokenCounter;
    mapping(uint256 => string) private s_tokenIdToUri;

    constructor() ERC721("Doggie", "DOGI") {
        s_tokenCounter = 0;
    }

    function mintNFT(string memory tokenUri) public returns (uint256) {
        s_tokenIdToUri[s_tokenCounter] = tokenUri;
    }

    function tokenURI(
        uint256 tokenId
    ) public view override returns (string memory) {
        return s_tokenIdToUri[tokenId];
    }
}
```

Awesome. Now let's actually mint the NFT and increment our token counter. We can mint the token by calling `_safeMint` from OZ.

Our final code should look like this:

```solidity
// SPDX-License-Identifier: MIT

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

pragma solidity 0.8.26;

contract BasicNft is ERC721 {
    uint256 private s_tokenCounter;
    mapping(uint256 => string) private s_tokenIdToUri;

    constructor() ERC721("Doggie", "DOGI") {
        s_tokenCounter = 0;
    }

    function mintNFT(string memory tokenUri) public returns (uint256) {
        s_tokenIdToUri[s_tokenCounter] = tokenUri;
        _safeMint(msg.sender, s_tokenCounter);
        s_tokenCounter++;
    }

    function tokenURI(
        uint256 tokenId
    ) public view override returns (string memory) {
        return s_tokenIdToUri[tokenId];
    }
}
```

## Deploying and interacting with the contract

- [Txn hash](https://sepolia.basescan.org/tx/0x6f748bce31b9851e7d3ede6befb20fc500eae0e277c8299b76f4e52cdefea174) of the deployed BasicNft contract.
- [Contract created](https://sepolia.basescan.org/address/0xc282a7dd595f35ea7e29a114d98f3d05f515724f)

To test this, I created a `dogi001.json` file and uploaded it to IPFS. File content:

```json
{
  "name": "DOGI001",
  "description": "A cute dogi",
  "image": "https://ipfs.io/ipfs/QmTx1tZNCySh1Q5Z6UNwhrjYZZJETbQLcvHFkvSjENiZzS",
  "attributes": [
    {
      "trait_type": "cuteness",
      "value": 100
    }
  ]
}
```

Then I pass this string with the CID I got:

`ipfs://QmdUFESRZN2cN1pghHFW9bNwLYT6816bBaodVjwiaTTW5V`

Now to confirm everything went well, pass '0' (since its the 1st minted NFT) to the `tokenURI` function and we get back the exact same string. Pretty underwhelming so far!

## Deploy Script

Let's now create a deploy script to deploy this via Foundry.

Create a new file `script/DeployBasicNft.s.sol`:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

import {Script} from "forge-std/Script.sol";
import {BasicNft} from "../src/BasicNft.sol";

contract DeployBasicNft is Script {
    function run() external returns (BasicNft) {
        vm.startBroadcast();
        BasicNft basicNft = new BasicNft();
        vm.stopBroadcast();
        return basicNft;
    }
}
```

Run `forge compile` to ensure everything is ok.

## Testing our NFT smart contract

Now it's time to write tests! 🥳

Create the file `test/BasicNftTest.t.sol`.

We add this code:

```solidity
//SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

import {Test} from "forge-std/Test.sol";
import {BasicNft} from "../src/BasicNft.sol";
import {DeployBasicNft} from "../script/DeployBasicNft.s.sol";

contract BasicNFTTest is Test {
    DeployBasicNft public deployer;
    BasicNft public basicNft;

    function setUp() public {
        deployer = new DeployBasicNft();
        basicNft = deployer.run();
    }

    function testNameIsCorrect() public view {
        string memory expected = "Doggie";
        string memory actual = basicNft.name();
        assert(expected == actual);
    }
}
```

But now we get this error:

`Built-in binary operator == cannot be applied to types string memory and string memory.solidity(2271)`

This is because a string in Solidity is an array of bytes and you can't compare arrays for equality like we're trying to do. We will **use chisel to encode each of our string objects into a hash and compare the hashes**.

Do this for now:

```bash
chisel
...
string memory cat = "cat";
string memory dog = "dog";
```

When you type `cat` you get this crazy output:

![](https://updraft.cyfrin.io/foundry-nfts/7-basic-nft-tests/basic-nft-tests2.png)

We will:

1. Use `abi.encodePacked` to convert this to bytes
2. Use keccak256 to hash the value into a `bytes32` object.

```bash
bytes memory encodedCat = abi.encodePacked(cat);
bytes32 catHash = keccak256(encodedCat);
```

If we apply this encoding and hashing methodology to our BasicNft test, we should come out with something that looks like this:

```solidity
function testNameIsCorrect() public view {
  string memory expectedName = "Doggie";
  string memory actualName = basicNft.name();

  assert(keccak256(abi.encodePacked(expectedName)) == keccak256(abi.encodePacked(actualName)));
}
```

Now run `forge test --mt testNameIsCorrect` and tests should pass.

Let's now test mint and balance.

We will write a test that ensures a user can mint the NFT and then change the user's balance.
We will create a user to prank in our test.

## Basic NFT Interactions

Let's write an interaction script for this.

Create the file `script/Interactions.s.sol`

We know we'll always want to be interacting with the latest deployment, so let's install the `foundry-devops` library to help with this.

`forge install Cyfrin/foundry-devops --no-commit`

Now, we can import `DevOpsTools` and use this to acquire our most recent deployment. We'll use this address as a parameter for the `mint` function we'll call.

## Deploy NFTs to a testnet

Great, let's now deploy this to a testnet.

Let's use the Makefile provided for us:

```
-include .env

.PHONY: all test clean deploy fund help install snapshot format anvil

DEFAULT_ANVIL_KEY := 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

help:
	@echo "Usage:"
	@echo "  make deploy [ARGS=...]\n    example: make deploy ARGS=\"--network sepolia\""
	@echo ""
	@echo "  make fund [ARGS=...]\n    example: make deploy ARGS=\"--network sepolia\""

all: clean remove install update build

# Clean the repo
clean  :; forge clean

# Remove modules
remove :; rm -rf .gitmodules && rm -rf .git/modules/* && rm -rf lib && touch .gitmodules && git add . && git commit -m "modules"

install :; forge install Cyfrin/foundry-devops@0.0.11 --no-commit && forge install foundry-rs/forge-std@v1.5.3 --no-commit && forge install openzeppelin/openzeppelin-contracts@v4.8.3 --no-commit

# Update Dependencies
update:; forge update

build:; forge build

test :; forge test

snapshot :; forge snapshot

format :; forge fmt

anvil :; anvil -m 'test test test test test test test test test test test junk' --steps-tracing --block-time 1

NETWORK_ARGS := --rpc-url http://localhost:8545 --private-key $(DEFAULT_ANVIL_KEY) --broadcast

ifeq ($(findstring --network sepolia,$(ARGS)),--network sepolia)
	NETWORK_ARGS := --rpc-url $(SEPOLIA_RPC_URL) --private-key $(PRIVATE_KEY) --broadcast --verify --etherscan-api-key $(ETHERSCAN_API_KEY) -vvvv
endif

deploy:
	@forge script script/DeployBasicNft.s.sol:DeployBasicNft $(NETWORK_ARGS)

Mint:
    @forge script script/Interactions.s.sol:MintNFT ${NETWORK_ARGS}

deployMood:
	@forge script script/DeployMoodNft.s.sol:DeployMoodNft $(NETWORK_ARGS)

mintMoodNft:
	@forge script script/Interactions.s.sol:MintMoodNft $(NETWORK_ARGS)

flipMoodNft:
	@forge script script/Interactions.s.sol:FlipMoodNft $(NETWORK_ARGS)
```

## ❓ Questions and 💪 Exercises

Question ❓: What does the `--no-commit` flag do?
Question ❓: Why can't we just compare a string to a string in our Solidity tests and what's the solution?

## 🛠️ Links and Resources

- [Repo](https://github.com/Cyfrin/foundry-nft-cu)

- [ERC-721: NFT Standard](https://eips.ethereum.org/EIPS/eip-721)
- [ERC-1155: Multi Token Standard](https://eips.ethereum.org/EIPS/eip-1155)
- [IPFS](https://ipfs.tech/)
- [IPFS Docs](https://docs.ipfs.tech/)
- [IPFS Project explanation](https://iq.wiki/wiki/ipfs)

- [OpenSea](https://opensea.io/)
- [Rarible](https://rarible.com/)
