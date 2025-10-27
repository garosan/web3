# Section 02. Swap

## Swap Math

### 🧠 The Goal:

We want to figure out **how many tokens you get out** of a swap when you **put in some tokens**.

For example:

- You add some token **X** into Uniswap.
- You want to know how many tokens **Y** you'll receive in return.

---

### 🧪 The Core Equation: `X * Y = L²`

This is the **invariant** of the AMM (Automated Market Maker). It means that the product of token X and token Y **must always stay the same** (as long as no one adds or removes liquidity).

Let’s define the variables:

- `X0` = amount of token X in the pool **before** the swap
- `Y0` = amount of token Y in the pool **before** the swap
- `dx` = amount of token X that Alice **adds**
- `dy` = amount of token Y that Alice **receives**
- After the swap:
  - New X amount = `X0 + dx`
  - New Y amount = `Y0 - dy`

We **lose** `dy` tokens from the pool (because they’re sent to Alice), and **gain** `dx` tokens (because Alice adds them).

---

### ✍️ Step-by-step Derivation:

Before the swap:

```
X0 * Y0 = L²
```

After the swap:

```
(X0 + dx) * (Y0 - dy) = L²
```

Since both expressions are equal to `L²`, we can set them equal to each other:

```
X0 * Y0 = (X0 + dx)(Y0 - dy)
```

Now, solve this for `dy`.

---

### 🔍 Solving for dy:

Start with:

```
X0 * Y0 = (X0 + dx)(Y0 - dy)
```

Expand the right side:

```
X0 * Y0 = (X0 + dx) * Y0 - (X0 + dx) * dy
```

Move terms around:

```
(X0 + dx) * dy = (X0 + dx) * Y0 - X0 * Y0
```

Now divide both sides by `(X0 + dx)`:

```
dy = [(X0 + dx) * Y0 - X0 * Y0] / (X0 + dx)
```

Distribute the top:

```
dy = [X0 * Y0 + dx * Y0 - X0 * Y0] / (X0 + dx)
```

Now cancel out `X0 * Y0`:

```
dy = dx * Y0 / (X0 + dx)
```

---

### ✅ Final Result:

> **dy = (Y0 \* dx) / (X0 + dx)**

This tells us:  
Given:

- X0 = current token X in the pool
- Y0 = current token Y in the pool
- dx = amount of token X you're swapping in

You can calculate:

- dy = amount of token Y you'll get **out**

---

### ⚠️ One Last Note:

This formula assumes **no fees**. In reality, Uniswap charges a small fee (like 0.3%), so you'll get **slightly less** than this amount. They’ll cover that in the next part of the course.

## Swap Fee

Let's discuss how swap fees are calculated and how we can factor them into our calculations when determining how much of a token we'll receive when we swap.

A swap fee is charged on the token that is inputted into the liquidity pool.

We can represent the swap fee as a fraction of the input token DX.

We'll call this fraction _F_. The swap fee rate _F_ is a number between zero and one. One would mean that the entire input is taken as a fee, while zero would mean there are no fees.

To calculate swap fee:

```javascript
swap fee = f * dx
```

Now, let's determine how to calculate the output amount of DY, _dy_, when we factor in swap fees.

### Example with numbers (how many tokens we'd get)

Let's go through an example of how to calculate DY, accounting for swap fees. Let's say we have a liquidity pool with 6,000,000 DAI and 3000 ETH. Uniswap V2 has a swap fee rate of 0.3%, or 0.003. If we are inputting _dx_ of 1000 DAI, we can plug our values into our formula:

```javascript
dy = (1000 * 0.997 * 3000) / (6000000 + 1000 * 0.997);
```

We can now use a calculator to determine the value of _dy_, which would be approximately 0.498341 ETH.

## Swap Contract Call

- We'll look at both single-hop and multi-hop swaps.
- A single-hop swap uses just one pair contract to swap between two tokens.
- A multi-hop swap requires two or more pair contracts, in a series, to swap two tokens.

Let's start with a single-hop swap. In this example, we'll swap WETH for DAI.

1. The user calls the `swapExactTokensForTokens` function on the Router contract.
2. This function takes the amount of WETH the user wants to swap and the address of the DAI token.
3. The Router then transfers the WETH to the DAI/WETH pair contract.
4. Once the WETH is transferred to the pair contract, the Router calls the `swap` function on the pair contract.
5. This function swaps the WETH for DAI, and the Router then transfers the DAI back to the user.

We can use the Router to swap other tokens, too. For instance, if the user wanted to swap ETH for DAI, they would call the `swapExactETHForTokens` function.

Let's look at a multi-hop swap. We'll swap WETH for MKR:

First, we see that there is no direct WETH/MKR pair contract. Instead, we have a DAI/WETH pair and a DAI/MKR pair.

1. In this case, the user calls the `swapExactTokensForTokens` function on the Router
2. The Router transfers the WETH to the DAI/WETH pair and calls the `swap` function. The WETH is swapped for DAI.
3. The Router then transfers the DAI to the DAI/MKR pair contract and calls the `swap` function. The DAI is swapped for MKR.
4. Finally, the Router transfers the MKR back to the user.

## Code Repo

The code for Uniswap V2 has 2 repos:

- v2-core
- v2-periphery

We could see it in these terms:

- The core is the engine
- The periphery is the user interface

`v2-core` contains the `UniswapV2Pair` contract.

1. Core = the engine
   Contains the UniswapV2Pair contract.

This is where the actual AMM logic happens:

It holds the tokens.

It does the swap math.

It updates reserves.

It issues LP (liquidity provider) tokens.

2. Periphery = the user interface
   Contains the Router contract.

It helps users interact safely with the Core:

Swap tokens

Add/remove liquidity

Perform multi-step swaps (multi-hop)

🧠 Why split them up?

1. Separation of concerns
   Each contract has a clear job:

Core: manage token pairs and handle the math/logic of the swaps.

Router (Periphery): guide users through complex interactions safely.

2. Safety
   If users interact directly with the Core (like calling swap() on UniswapV2Pair), they could:

Use wrong parameters

Lose tokens

Break the swap

The Router makes sure the steps are done correctly and in the right order.

3. Multi-hop swaps
   Say you want to trade Token A → Token B → Token C in one transaction.

The Core contracts don’t know how to do this.

The Router chains together multiple swaps across different pools for you.

4. Extensibility
   If Uniswap wants to add features or build new ways to interact (like different kinds of routes or fee options), they can upgrade or replace the Router contract without touching the Core engine logic — which should remain rock-solid and unchanged.

## Code Walk - swapExactTokensForTokens

In this lesson, we'll explore the code for `swapExactTokensForTokens` from Uniswap V2. This function allows us to swap tokens for other tokens. For instance, if we want to swap 1,000 DAI for the maximum possible amount of WETH, we'd use `swapExactTokensForTokens` for this purpose.

Let's review how this function works. The function uses a Uniswap V2 library function called `getAmountOut` to calculate an array of `uint` values. This array, called `amounts`, stores the amount of tokens exchanged during the swap process. The first element in the `amounts` array represents the input token amount. The last element holds the output token amount, while any intermediate elements correspond to amounts exchanged during multi-hop swaps.

The `swapExactTokensForTokens` function first transfers the input tokens to the pair contract. This pair contract is calculated using a function called `create2` based on the input and output token addresses provided in the `path` array. The `path` array contains the addresses of tokens involved in the swap. For instance, if we're swapping DAI to WETH and then to MKR, the `path` array would look like this:

```javascript
[DAI, WETH, MKR];
```

Inside the `swap` function, the function loops through the `path` array, iterating from index zero to `path.length - 1`. Each iteration calculates the pair contract address, and then executes the `swap` function on the pair contract.

We'll delve into the details of `getAmountOut` in a later lesson.

Find that code in this contract:
https://github.com/Uniswap/v2-periphery/blob/master/contracts/UniswapV2Router01.sol

## Code Walk - getAmountOut()

## 🧠 Goal of `getAmountOut`

We want to calculate:

> **How many tokens will I get out** if I put some in?

This is what Uniswap's `getAmountOut` function does:  
🧮 **It uses math + the reserves + the swap fee** to figure out how much of the other token you’ll receive.

---

## 🧩 Inputs to the Function

```solidity
function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut)
```

Let’s define each:

| Variable     | Meaning                                                         |
| ------------ | --------------------------------------------------------------- |
| `amountIn`   | The amount of tokens you're swapping in (like dx before)        |
| `reserveIn`  | How many tokens are in the pool **for the token you're giving** |
| `reserveOut` | How many tokens are in the pool **for the token you want**      |

---

## 🔁 Uniswap Charges a 0.3% Fee

So instead of using all of `amountIn`, Uniswap only lets **99.7%** of it go into the actual swap math.

```solidity
uint amountInWithFee = amountIn * 997;
```

(997 out of 1000 = 99.7%)

---

## 🧮 The Math

Let’s look at the core formula:

```solidity
amountOut = (amountInWithFee * reserveOut) / (reserveIn * 1000 + amountInWithFee)
```

Now let’s decode this piece-by-piece.

### 🔹 Numerator:

```solidity
amountInWithFee * reserveOut
```

This means:

> How much buying power do you have (after the fee) multiplied by how many tokens you want are available

### 🔹 Denominator:

```solidity
reserveIn * 1000 + amountInWithFee
```

This scales up the math to account for:

- Current reserves
- Your added tokens
- And the fee

---

### ✅ Final Formula Summary:

> **How much you get = (your input minus fee) × how much is available / the new pool balance after your swap**

---

## 📌 TL;DR

- This function calculates how many tokens you'll receive in a swap.
- It accounts for the 0.3% fee by multiplying your input by 997/1000.
- The formula ensures the AMM invariant is respected and you're not getting more than the reserves can offer.
- It's based on the `X * Y = K` curve + fee adjustments.

## Get Amounts Out Fork

## 🛠️ Links and Resources

- [v2-core-repo](https://github.com/Uniswap/v2-core)
- [v2-periphery repo](https://github.com/Uniswap/v2-periphery)
