# Section 01. Simple Storage

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract SimpleStorage {

    uint256 favoriteNumber; // storage variable: it's stored in a section of the blockchain called "Storage"

    function store(uint256 _favoriteNumber) public {
        // the variable favorite number is updated with the value that is contained into the parameter `_favoriteNumber`
        favoriteNumber = _favoriteNumber;
    }
}
```

### Visibility Keywords

- `public`: Can be called by anyone.
- `private`: Can be called by this contract only.
- `external`: Only for functions. Can only be called by outside entities.
- `internal`: Can be called by this contract and inherited contracts.

### Calldata and Memory

In Solidity, `calldata` and `memory` are temporary storage locations for variables during function execution. `calldata` is read-only, used for function inputs that can't be modified. In contrast, `memory` allows for read-write access, letting variables be changed within the function. Most variable types default to `memory` automatically. However, for **strings**, you must specify either `memory` or `calldata` due to the way arrays are handled in memory.

### Calldata

Calldata variables are read-only and cheaper than memory. They are mostly used for input parameters.

### Storage

Variables stored in `storage` are persistent on the blockchain, retaining their values between function calls and transactions.

### Strings and primitive types

If you try to specify the `memory` keyword for an `uint256` variable, you'll encounter this error:

`> Data location can only be specified for array, struct, or mapping type`

In Solidity, a `string` is recognized as an **array of bytes**. On the other hand, primitive types, like `uint256` have built-in mechanisms that dictate how and where they are stored, accessed and manipulated.

You can't use the `storage` keyword for variables inside a function. Only `memory` and `calldata` are allowed here, as the variable only exists temporarily.

### Mapping

Iterating through a long list of data is usually expensive and time-consuming. To directly access the desired value without the need to iterate through the whole array, we can use **mappings**. They are sets of (unique) **keys** linked to a **value**.

A mapping is defined using the mapping keyword, followed by the key type, the value type, the visibility, and the mapping name.

### Deploying to a testnet using Remix and Metamask

This is how the contract should look now:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract SimpleStorage {
    uint256 myFavoriteNumber;

    struct Person {
        uint256 favoriteNumber;
        string name;
    }
    // uint256[] public anArray;
    Person[] public listOfPeople;

    mapping(string => uint256) public nameToFavoriteNumber;

    function store(uint256 _favoriteNumber) public {
        myFavoriteNumber = _favoriteNumber;
    }

    function retrieve() public view returns (uint256) {
        return myFavoriteNumber;
    }

    function addPerson(string memory _name, uint256 _favoriteNumber) public {
        listOfPeople.push(Person(_favoriteNumber, _name));
        nameToFavoriteNumber[_name] = _favoriteNumber;
    }
}
```

Now make sure that it compiles and deploy to Sepolia testnet, to do this you will need to get some tokens from a faucet.

If I deployed to Sepolia ETH this would cost according to Metamask:

- 0.0129 SepoliaETH $43.73

I instead deployed to Base Sepolia, [txn hash](https://sepolia.basescan.org/tx/0x71b7f357255c50f4ea819135e32eec088b07c506e23fb51e11c82c79fe08569a), the cost was around $0.001 USD.

## ❓ Questions and 💪 Exercises

Question ❓: How can you tell apart a transaction that just sends ether from a transaction that deploys a contract?
R.- For a deployment transaction, the bytecode of the contract is placed inside the data field, otherwise its empty.

## 🛠️ Links and Resources

- [Github Repo](https://github.com/Cyfrin/foundry-full-course-cu)
- [Solidity Docs](https://docs.soliditylang.org/en/v0.8.28/)
- [Solidity Docs - Visibility](https://docs.soliditylang.org/en/latest/contracts.html#visibility-and-getters)
- [Solidity Docs - Storage, Transient Storage, Memory and the Stack](https://docs.soliditylang.org/en/latest/contracts.html#visibility-and-getters)
