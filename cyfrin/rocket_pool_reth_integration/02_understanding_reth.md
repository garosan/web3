# Section 02. Understanding rETH

## ETH Staking

We will beginning by understanding the challenges of ETH staking and what is the problem that Rocket Pool solves.

The purpose of ETH staking is to:

- Run a validator
- Secure the network
- Earn ETH rewards

The requirements are:

- 32 ETH
- Validator Key
- Withdrawal Key
- Run Validator

So to become a staker you need:

- 32 ETH
- Deep tech skills to run your validator and secure your ETH

## Rocket Pool

Rocket Pool is a decentralized ETH staking protocol.

In Rocket Pool you can add your little amount of ETH to the RocketPool contracts to _pool_ the ETH to stake, but also if you have technical knowledge you can register to participate in the network as a node operator. The difference would be that you would be operating RocketPool's software instead of an Ethereum node.

## rETH

rETH is a liquid staking token issued by Rocket Pool. There are two ways to obtain rETH.

- The first is to mint rETH by directly depositing ETH into the Rocket Pool protocol. For this to happen, minting must be enabled by the Rocket Pool protocol.

- The second way is to obtain rETH from a decentralized exchange. You can swap any tokens to buy rETH.

For the 1st way, the exchange rate is not one-to-one. In the example Alice deposits 1 ETH and gets back 0.83 rETH. This is because the rETH token is expected to accrue value over time if everything goes well.

Minting is activated by the Rocket Pool protocol and is not always active. Depending on market conditions and trading fees, it can be more advantageous to mint the rETH or to buy it in an exchange.

## Value of rETH

Today Apr 13th, 2025 rETH is $1830 and ETH is $1615.

rETH increases in value relative to ETH because it accumulates staking rewards.
Of course, if the node operators of the Rocket Pool are penalized for some kind of misbehavior, then the value of rETH can decrease relative to ETH.

The graph for rETH would look like an ascending stair because the rETH team adjusts the exchange rate periodically.

## Rebase Token

- Rocket Pool rETH is a non-rebase token.
- Lido stETH is a rebase token.

A rebase token means that the token supplies and the balances change algorithmically.

The algorithm will assign extra tokens to the users, the users will burn tokens and this will bring back down the supply.

### Non-rebase token

A non-rebase token means that the token supply and balances of a token only change when there's a mint or a burn.

### Taxes

Due to the way the token balances change (or don't change), there might be different tax implications between rETH and stETH.

## What can you do with rETH?

Some of the things you can do with rETH:

- Add liquidity to DEXes like Uniswap V3, Curve, Balancer v2, etc.
- Lend rETH in Aave to create a leveraged position by supplying rETH as collateral to Aave, then borrowing a stablecoin to buy more rETH
- Restake rETH on EigenLayer

The 1st option is not advised:

It is important to know, however, that we would not recommend putting rETH as liquidity into Uniswap v3. The reason is the value of rETH generally increases over time relative to the value of ETH. This is because rETH accrues the staking rewards. So if we were to add liquidity to Uniswap v3, then we are saying that we are willing to sell our rETH at a certain price range. But over time, rETH will grow in value. So to keep up with this increasing value of rETH, we will have to keep on repositioning the liquidity position in Uniswap v3. Otherwise, we are selling our rETH at a discount.

We suggest we deposit our rETH into other decentralized exchanges such as Curve and Balancer.

//TODO: Investigate exactly why it's disadvantageous to deposit liquidity in Uniswap V3 but not on Curve or Balancer.

Curve is better than Uniswap for adding rETH because both on Curve and Balancer, the exchange rate of rETH is always updated.

For the 3rd option: When we stake ETH into the Rocket Pool, we get back rETH, which is giving us some staking rewards in ETH. Now take this rETH, and then stake it into EigenLayer

## Getting rETH

You have 3 options basically:

- Directly use the Rocket Pool protocol
- Use a DEX aggregator
- Use any other decentralized exchanges

If you want the best deal for your rETH, you'll have to compare the exchange rates between different protocols. Here we compared the exchange rates between the Rocket Pool protocol, DEX aggregator, and some decentralized exchanges.

## Rocket Pool Contracts - Storage

Let's start to undeerstand how the Rocket Pool contracts are organized. Let's start with the storage contract.

This is a contract that multiple Rocket Pool contracts have access to. Authorized contracts will be able to write any arbitrary data into the Rocket Pool storage contract, and any contract will be able to read the data from this Rocket Pool storage contract.

As an analogy, this is like a database where only authorized users will be able to write into the database, and any user will be able to query the database.

## Rocket Pool Contracts - Mint and Burn

Here are the steps needed to understand how a user deposits ETH and mints RETH:

1. A user deposits `ETH` and mints `rETH`.
2. Several Rocket Pool contracts are called, but we will highlight three contracts: `RocketDepositPool`, `RocketStorage`, and `RocketTokenRETH`.
3. Alice has 1 ETH, and she deposits this ETH into the `RocketDepositPool` contract.
4. Among many things, the `RocketDepositPool` contract will get an address from the `RocketStorage`.
5. This will get an address for a contract that is responsible for the protocol settings.
6. Here, it is going to check whether deposit is enabled or not.
7. It checks that deposit is enabled, and if it is, it will call the function `mint` on the `RocketTokenRETH`, which is the `rETH` token contract.
8. Now, the `RocketTokenRETH` contract will call the `RocketStorage` contract so that it can get contracts to query data that is needed to calculate the exchange rate from `ETH` to `rETH`.
9. Once the exchange rate is calculated, it will mint `rETH` to Alice.

And the steps to burn:

1. To redeem ETH from RETH, the user will have to burn their RETH.
2. There are multiple contracts that are involved with burning RETH, but again, we will highlight three contracts: RocketDepositPool contract, RocketTokenRETH, and RocketStorage.
3. Let’s say that Alice has RETH and she wants to redeem her ETH.
4. The first thing she’ll do is call the function burn on the RocketTokenRETH contract.
5. Once the token is burned, the RETH contract will have to get addresses by calling into the RocketStorage contract.
6. It will get the address for the contracts that are needed to calculate the exchange rate from RETH back to ETH.
7. Now, if there are enough ETH inside the RETH contract, then ETH will simply be sent back to Alice, and we’re done here.
8. However, if there is not enough ETH inside the RocketTokenRETH contract, then it will have to call the function withdrawExcessBalance on the RocketDepositPool contract.
9. The RETH contract is asking the RocketDepositPool contract to send some ETH back to the RocketTokenRETH contract.
10. The RocketDepositPool contract will have to get another contract to calculate the excess amount of ETH that can be withdrawn.
11. This is done by calling the function getAddress on the RocketStorage contract.
12. The RocketStorage contract will store the contract address that is responsible for calculating the excess balance that can be withdrawn.
13. Once that is done, ETH is sent from the RocketDepositPool contract over to the RETH token contract.
14. Then finally, ETH is sent back to Alice.

## Exchange rate math

To calculate the rETH/ETH exchange rate, we will use a formula to determine the relative value of rETH compared to ETH within the Rocket Pool protocol.

These are the variables:

- E = total ETH
- R = total rETH
- r = rETH amount
- e = ETH amount

With these variables, the equation to determine the ETH amount (e) for a given rETH amount (r) is:

`e = (r / R) * E`

Example:

Given the total amount of rETH is 5000 and the total amount of ETH locked into the Rocket Pool protocol is 6000, how much ETH can we get for 1 rETH?

`e = (1 / 5000) * 6000`

Solving this we get:

`e = 1.2 ETH`

To do it in reverse, we start with the same equation and solve for `r`:

`r = (e * R) / E` or `r = (1 * 5000) / 6000`

And the result is:

`r = 0.8333 rETH`

## Function `Deposit` in `RocketDepositPool` Contract

The function `deposit` is called by the user if they want to mint rETH.

So when a user calls `deposit`, the function checks if deposit are enabled or not:

- It first gets a contract address by calling the function `getAddress` on the `RocketStorage` contract
- It gets the contract address of `RocketDAOProtocolSettingsDeposit`. Once it gets this contract, it calls a function called `getDepositEnabled`.
- Next, it calls the function `getDepositFee` again on the contract `RocketDAOProtocolSettingsDeposit`
- Then multiplies this by `msg.value` and then divides by `calcBase`.

In the future this might change, but if you are writing an integration with the Rocket Pool protocol, currently there is a deposit fee. Depending on the percentage of what the deposit fee is, the actual amount of ETH to calculate the amount of rETH to mint may be less than the amount of ETH that the user sent.

This is the actual `deposit` function:

```solidity
    /// @notice Deposits ETH into Rocket Pool and mints the corresponding amount of rETH to the caller
    function deposit() override external payable onlyThisLatestContract {
        // Check deposit settings
        RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(getContractAddress("rocketDAOProtocolSettingsDeposit"));
        require(rocketDAOProtocolSettingsDeposit.getDepositEnabled(), "Deposits into Rocket Pool are currently disabled");
        require(msg.value >= rocketDAOProtocolSettingsDeposit.getMinimumDeposit(), "The deposited amount is less than the minimum deposit size");
        /*
            Check if deposit exceeds limit based on current deposit size and minipool queue capacity.

            The deposit pool can, at most, accept a deposit that, after assignments, matches ETH to every minipool in
            the queue and leaves the deposit pool with maximumDepositPoolSize ETH.

            capacityNeeded = depositPoolBalance + msg.value
            maxCapacity = maximumDepositPoolSize + queueEffectiveCapacity
            assert(capacityNeeded <= maxCapacity)
        */
        uint256 capacityNeeded = getBalance().add(msg.value);
        uint256 maxDepositPoolSize = rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize();
        if (capacityNeeded > maxDepositPoolSize) {
            // Doing a conditional require() instead of a single one optimises for the common
            // case where capacityNeeded fits in the deposit pool without looking at the queue
            if (rocketDAOProtocolSettingsDeposit.getAssignDepositsEnabled()) {
                RocketMinipoolQueueInterface rocketMinipoolQueue = RocketMinipoolQueueInterface(getContractAddress("rocketMinipoolQueue"));
                require(capacityNeeded <= maxDepositPoolSize.add(rocketMinipoolQueue.getEffectiveCapacity()),
                    "The deposit pool size after depositing (and matching with minipools) exceeds the maximum size");
            } else {
                revert("The deposit pool size after depositing exceeds the maximum size");
            }
        }
        // Calculate deposit fee
        uint256 depositFee = msg.value.mul(rocketDAOProtocolSettingsDeposit.getDepositFee()).div(calcBase);
        uint256 depositNet = msg.value.sub(depositFee);
        // Mint rETH to user account
        rocketTokenRETH.mint(depositNet, msg.sender);
        // Emit deposit received event
        emit DepositReceived(msg.sender, msg.value, block.timestamp);
        // Process deposit
        processDeposit(rocketDAOProtocolSettingsDeposit);
    }
```

And the whole [RocketDepositPool.sol](https://github.com/rocket-pool/rocketpool/blob/master/contracts/contract/deposit/RocketDepositPool.sol) contract.

## RocketTokenrEth.sol Contract

Let's check the contract `RocketTokenrEth.sol`. This is the rEth token contract.

We want to see the functions `mint`, `burn` and their exchange rates.

he function `mint` can only be called by an authorized contract, the `RocketDepositPool` contract.
It then calculates the amount of rEth to mint by calling the function `getREthValue`.

The `burn` function can be called by anyone to burn their rEth, and they can redeem the ETH. It calculates the amount to give back by calling `getEthValue`.

Finally, let's check the transfer hook. At the bottom, you see that there is a hook called `_beforeTokenTransfer`.

If this is not a mint but a regular transfer, then there is a block delay before another transfer can happen.

The information is stored inside the key `user.deposit.block`, inside the `RocketStorage` contract.

The deposit delay exists for a historical reason. Currently it's not used, since its functionality is replaced by the mint fee. The code still remains inside this contract, because the rEth contract is an immutable contract.
