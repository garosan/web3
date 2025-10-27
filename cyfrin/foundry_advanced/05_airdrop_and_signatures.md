# Section 04. Airdrop and Signatures

## Introduction

In this project, we'll delve into **Merkle Trees** and **Signatures** to create our very own _ERC20 airdrop contract_.

An airdrop occurs when a token development team distributes tokens or allows people to claim them. These tokens can be of various types, including ERC-20, ERC-1155, or ERC-721.

Tokens are typically given for free, with eligibility criteria such as contributing to the project's GitHub repository or participating in the community. This process helps to _bootstrap the project_ by distributing tokens to a **list of eligible addresses**.

Let's check the [code base](https://github.com/Cyfrin/foundry-merkle-airdrop-cu). We have a minimal Bagel token.

The [`MerkleAirdrop`](https://github.com/Cyfrin/foundry-merkle-airdrop-cu/blob/main/src/MerkleAirdrop.sol) contract is the cornerstone of this project, and it uses **Merkle Proofs** to verify address eligibility.

It includes a `claim` function that allows addresses to receive the airdrop without paying gas fees. Furthermore, it implements _signatures_ to ensure that only intended recipients can claim the tokens.

We will generate **scripts** to create Merkle Trees, Proofs, and Root Hash, as well as deploy and interact with the contract.

In this course, we will cover:

- Merkle Trees
- Merkle Proofs
- signatures
- ECDSA (Elliptical Curve Digital Signature) algorithm
- Transaction types

Furthermore:

- After initializing a ZKsync local node with Docker, we'll deploy the `Bagel` token and `MerkleAirdrop` contracts on it
- We'll then **sign a message** to allow someone else to call `claim` on your behalf so you can receive the token while not paying for gas fees
- The initial supply of tokens is created and sent to the airdrop contract
- Finally, we can claim tokens on behalf of the claiming address (so they do not have to pay gas) using a signature
- The address will now hold a balance of the airdropped tokens

## Project Setup

```bash
mkdir merkle-airdrop
cd merkle-airdrop
foundryup
forge init
```

- Delete the 3 `Counter` example files.
- Create `BagelToken.sol`. We will use OpenZeppelin's `ERC20` and `Ownable` so let's run:

`forge install openzeppelin/openzeppelin-contracts --no-commit`

Add the remapping in our `foundry.toml`:

`remappings = [ '@openzeppelin/contracts/=lib/openzeppelin-contracts/contracts/']`

Now let's create our contract with a `constructor` and a `mint` function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";

contract BagelToken is ERC20, Ownable {
    constructor() ERC20("Bagel Token", "BT") Ownable(msg.sender) { //the deployer is the owner of the contract
    }

    function mint(address account, uint256 amount) external onlyOwner {
        _mint(account, amount);
    }
}
```

Let's now create a new file named `MerkleAirdrop.sol`, where we will have a list of addresses

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract MerkleAirdrop {
    // list of addresses that can receive tokens
    // allow someone in the list to claim some tokens
}
```

The contracts will be connected by passing the `BagelToken`, or any ERC20 token to the `MerkleAirdrop` constructor. Then we can add our claimer address into an array of addresses:

`address [] claimers;`

Then we would need a function that checks that the claimer is in this whitelist and allow him to receive tokens.

We could iterate through the array like this:

```solidity
function claim(address account) external {
    for (uint256 i=0; i<claimers.length; i++){
        //check if the account is in the claimers array
    }
}
```

But this is cost-prohibitive, can make our tx run out of gas and fail.

Merkle Trees are a data structure that allows us to manage and verify large sets of data efficiently, while Merkle Proofs can help to prove that some piece of data is contained within a group.

## Merkle Proofs

Merkle Trees, Merkle Proofs, and Root Hashes are very important in blockchain technology, invented by Ralph Merkle in 1979.

Instead of manually iterating through the array, Merkle trees provide a smarter way to do this by using cryptographic hashes and a tree-like structure to verify membership efficiently.

When you read this, try to remember how a Merkle tree is built:

- Step 1: Hash each item in the array
- Step 2: Next, we take pairs of hashes and hash them together
- Step 3: Keep pairing and hashing until one final hash remains

This single hash at the top is called the Merkle Root or the hash root, and it represents the entire tree.

To prove that Charlie is in the list, you only need:

- The Merkle Root (the top hash)
- A small "proof" consisting of:
  - h(Dave) (Charlie's sibling hash in the tree)
  - the other branch of the tree

### Applications

Merkle trees are used in **rollups** to verify state changes and transaction order and in **airdropping**, enabling specific addresses to claim tokens by being included as **leaf nodes** in the tree.

OpenZeppelin provides a library for [MerkleProofs](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/dbb6104ce834628e473d2173bbc9d47f81a9eec3/contracts/utils/cryptography/MerkleProof.sol), and we will use it in our smart contract. Its [`verify`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/dbb6104ce834628e473d2173bbc9d47f81a9eec3/contracts/utils/cryptography/MerkleProof.sol#L32) function takes the proof, the Merkle root, and the leaf to be verified as inputs.

```solidity
function verify(bytes32[] memory proof, bytes32 root, bytes32 leaf) internal pure returns (bool) {
    return processProof(proof, leaf) == root;
}
```

The **root** is typically stored _on-chain_, while the **proof** is generated _off-chain_.

The [`processProof`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/dbb6104ce834628e473d2173bbc9d47f81a9eec3/contracts/utils/cryptography/MerkleProof.sol#L49) function iterates through the proof array, updating the computed hash by hashing it with the next proof element. This process ultimately returns a computed hash, which is compared to the expected root to verify the leaf's presence in the Merkle Tree.

```solidity
function processProof(bytes32[] memory proof, bytes32 leaf) internal pure returns (bytes32) {
    bytes32 computedHash = leaf;
    for (uint256 i = 0; i < proof.length; i++) {
        computedHash = _hashPair(computedHash, proof[i]);
    }
    return computedHash;
}
```

### Conclusion

Merkle proofs will help verifying that a specific piece of data is part of the Merkle tree. By providing hashes of sibling nodes at each level of the tree, a verifier can reconstruct the root hash and confirm data integrity.

## Base Airdrop Contract

We are going to implement Merkle proofs and Merkle trees in our `MerkleAirdrop.sol` contract by setting up the _constructor_ and creating a _claim_ function.

The constructor should take an ERC-20 token and a Merkle root as parameters, to store them for later use:

## 🛠️ Links and Resources

- [GitHub Repo](https://github.com/Cyfrin/foundry-merkle-airdrop-cu)
