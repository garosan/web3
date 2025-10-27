# Section 01. ERC20 Crypto Currency

## Welcome and Intro to ERC-20

We hit the ground running in this section. We are going to develop an ERC20 Crypto Currency.

One of the most recognized `Ethereum Requests for Comments` is the **[ERC20 Token Standard](https://eips.ethereum.org/EIPS/eip-20)**. This is a proposal for creating and managing these tokens on the Ethereum blockchain.

A few common use cases of ERC-20 tokens:

- Governance Tokens
- Securing an underlying network
- Synthetic Assets
- Stable coins

How do I build an ERC-20?

All anyone has to do to develop and `ERC20` is to deploy a smart contract which follows the **[token standard](https://eips.ethereum.org/EIPS/eip-20)**. This ultimate boils down to assuring our contract includes a number of necessary functions: `transfer`, `approve`, `name`, `symbol`, `balanceOf` etc.

## Create an ERC 20

```bash
mkdir erc-20-manual
cd erc-20-manual
forge init
```

Go ahead and delete our 3 `Counter` examples so we can start with a clean slate.

We will cover 2 different ways to create our own token, first the hard way and then a much easier way!

Let's now create a `ManualToken.sol` and add this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract ManualToken {}
```

Now all we need to do is add the required functions/methods to follow the ERC-20 standard:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract ManualToken {

    function name() public pure returns(string memory) {
        return "Manual Token";
    }

    function totalSupply() public pure returns (uint256){
        return 100 ether; // 100000000000000000000
    }

    function decimals() public pure returns (uint8) {
        return 18;
    }

    function balanceOf(address _owner) public pure returns (uint256){
        return // ???
    }
}
```

Hmm, what is this function meant to return exactly? We're probably going to need a mapping to track the balances of each address...

```solidity
mapping(address => uint256) private s_balances;
```

So now our `balanceOf` function can return this mapped value based on the address parameter being passed.

```solidity
function balanceOf(address _owner) public view returns (uint256) {
    return s_balances[_owner];
}
```

Our next required function is transfer:

```solidity
function transfer(address _to, uint256 _amount) public {
    uint256 previousBalance = balanceOf(msg.sender) + balanceOf(_to);
    s_balances[msg.sender] -= _amount;
    s_balances[_to] += _amount;

    require(balanceOf(msg.sender) + balanceOf(_to) == previousBalance);
}
```

We could absolutely continue going through each of the required functions outlined in the ERC20 Token Standard and eventually create a compatible contract, but there's an easier way. In the next lesson we will explore OpenZeppelin.

## Exploring OpenZeppelin

OpenZeppelin is renowned for its Smart Contract framework, offering a vast repository of audited contracts readily integrable into your codebase.

We invite you to explore OpenZeppelin's contracts [documentation](https://docs.openzeppelin.com/contracts/5.x/).

Let's leverage OpenZeppelin to create a new ERC20 Token. Create a new file within `src` named `OurToken.sol`. Once that's done, let's install the OpenZeppelin library into our contract. Run:

`forge install OpenZeppelin/openzeppelin-contracts --no-commit`

Once installed add this to your `foundry.toml`:

`remappings = ["@openzeppelin=lib/openzeppelin-contracts"]`

Now we can now import and inherit this contract into `OurToken.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract OurToken is ERC20 {
    //constructor goes here
}
```

Now we're getting an error. We should recall that when inheriting from a contract with a constructor, our contract must fulfill the requirements of that constructor. We'll need to define details like a name and symbol for OurToken:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract OurToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("OurToken", "OT") {
        _mint(msg.sender, initialSupply);
    }
}
```

For the purposes of simple examples like this, I like to mint the initialSupply to the deployer/msg.sender, which I've demonstrated above.

Awesome, let's now see how to deploy this token!

## Deploy your ERC-20 token

In your workspace's `script` folder, create a file named `DeployOurToken.s.sol`.


## ❓ Questions and 💪 Exercises

## Links

- [GitHub Repo](https://github.com/Cyfrin/foundry-erc20-cu)
- [ERC-20: Token Standard](https://eips.ethereum.org/EIPS/eip-20)
