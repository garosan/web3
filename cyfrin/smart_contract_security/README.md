# Smart Contract Audits, Security, and DeFi FULL Course | Learn smart contract auditing

- [Youtube Video](https://www.youtube.com/watch?v=pUWmJ86X_do)

## Common EIPs/ERCs

- [ERC-20](#)
- [ERC-677](#)
- [ERC-721](#)
- [ERC-777](#)
- [ERC-1155](#)
- [ERC-1967: Standard Proxy Storage Slots](#)

## Links

- [Introduction To ERC Token Standards — Part 1](https://medium.com/immunefi/how-erc-standards-work-part-1-c9795803f459)

Major Drawbacks of ERC20 Standards:

1. Impossibility of handling incoming token transactions.

ERC20 has only functionality for moving and accounting for funds. There is no way to handle incoming token transactions and no way to reject any non-supported tokens.

2. The transfer process is inefficient.

transfer() is suitable and safe for transferring to EOAs only. If you want to transfer tokens to a contract, you need to ensure that the receiver contract can handle incoming transfers. An accidental token transfer to a contract that is not ERC-20 compatible will result in loss of tokens.

To send the funds to a smart contract, Alice needs to use the approve and transferFrom combination.

a. First, the user needs to set allowance for the receiverContract with an amount: approve(receiverContract,100)

b. Then, the user has to make a call from receiverContract to move user funds from the ERC20 token contract to the receiver contract using transferFrom(Alice,receiverContract,100)

c. This pattern is also gas-inefficient, since it requires two separate transactions to move the funds from the user to the receiver contract.s
