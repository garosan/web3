# Advanced Web3 Security Course | Part 1

- [Link](https://youtu.be/DRZogmD647U)

Notes so far...:

## Smart Contract Design

1. The less code, the less bugs
2. Be extremely picky about creating storage variables
3. How much of the logic of my contract can be done off-chain?
4. Avoid for loops as much as possible (where you can run out of gas)
5. Limit expected inputs (disallow inputs that shouldn't be allowed)
6. Handle all cases (i.e. what if the keeper/oracle goes down?)
7. Parallel data structures (avoid having 2 pieces of state tracking the same thing)

## External Call Safety

A study showed external calls and oracle manipulation are the 2 most common attacks.

Consider this when doing external calls in your contracts:

- Reentrancy
- DoS
- Return values
- Gas

### Reentrancy

- Use CEI
- Apply `nonReentrant`
- `nonReentrant` can fail in multiple contract systems.
- If we are talking about 2 contracts in 2 separated systems, 'read-only reentrancy' is still a possibility.

### DoS Attacks

Didn't understand this part:
https://youtu.be/DRZogmD647U?t=4108

### Return values

- Check for all possible values
- Check for unexpected bytes

### Gas

If you don't trust the contract you're calling, don't forward all the gas

### Documentation

Example of a very good README:

https://github.com/Blueberryfi/blueberry-core

## Complete Reentrancy Guide

We will cover 4 types of reentrancy:

1. Classic reentrancy
2. Cross-function reentrancy
3. Cross-contract reentrancy
4. Read-only reentrancy

https://youtu.be/DRZogmD647U?t=6711

## Links

- [Youtube Part 1](https://youtu.be/DRZogmD647U)
- [Youtube Part 2](https://youtu.be/DRZogmD647U)
- [Course Notion](https://guardianaudits.notion.site/Gateway-Free-Web3-Security-Course-574f4d819c144d7895cda6d61ba26503)
