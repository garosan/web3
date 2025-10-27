# Section 01. Favorites

## Intro

In the repo link, you can find a repo for each of the lessons.
I.e. for this **Favorites** exercise, you can find the associated repo [here](https://github.com/Cyfrin/remix-favorites-cu)

Introduction to Remix and then we will create our first `favorites.vy` file.

## Basics

To declare the pragma version:
(we will be using 0.4.0 which has a lot of additions and cool features)

```vyper
# pragma version 0.4.0
# This is a comment
```

To add a license:

```vyper
# @license MIT
```

## Compiling

Make the exercise of compiling this:

```vyper
# pragma version 0.4.0
```

Interestingly, we already get a ton of information in the compilation details:

A minimal ABI:

```
{
  "evmVersion": "",
  "version": "0.4.0"
}
```

A bytecode:

`61000361000f6000396100036000f35f5ffd84038000a1657679706572830004000011`

And a runtime bytecode:

`5f5ffd`

Notice that the comments are ignored by the compiler, are whatever you want in the comments, add a line, a haiku, a whole novel in the comments, the output bytecode will be exactly the same.

## Our First Contract

We are going to build a minimal Vyper contract that will store:

- A list of favorite things
- A list of favorite numbers
- A list of favorite people with their favorite numbers

## Vyper Types

You can take a look at the types available in Vyper [here](https://docs.vyperlang.org/en/stable/types.html).

But don't get overwhelmed, for now, just know that these types exist (you probably already are familiar with them):

- `uint<N>`
- `int<N>`
- `bool`
- `address`
- `bytes<N>`: Remember this is for _fixed-size byte arrays_, useful for hashes and fixed-length data.

And we will later learn about these more complex ones:

- `bytes`
- `string`
- `decimal`
- `struct`
- `array`
- `mapping`

No need to learn them by heart.

This is how some of the 1st examples would look (although this won't compile):

```vyper
my_number: uint256 = 42
is_active: bool = True
owner: address = 0x1234567890abcdef1234567890abcdef12345678
```

## Variable Visibility

Do this exercise, deploy this contract inside the Remix VM:

```vyper
# pragma version 0.4.0
# @license MIT

my_number: uint256
```

You will not see anything after deploy. Now deploy this one:

```vyper
# pragma version 0.4.0
# @license MIT

my_number: public(uint256)
```

And you will see a blue button for your variable. Since you made it public, you can access it publicly and Remix gives you a nice button to do that.

If you don't add the public visibility, by default your storage variable will only be accessible from within the contract, but remember, any smart engineer can access internal variables in a blockchain.

## Functions

Let's start doing cool stuff with functions.

Consider this code:

```vyper
@external
def store(new_number: uint256):

    self.my_favorite_number = new_number
```

And note:

- Functions are defined with the keyword `def` as in Python
- Remember to always indent the code inside the function
- We pass a parameter `new_number` and have to add declare its type
- You can think of `self` as 'within this contract, look for the `my_favorite_number` variable
- `@external` is a decorator, like in Python. Here it's saying the next function is of external visibility.

## Function Visibility

- If there is no visibility decorator, the function is internal by default.

## Return a value and @view decorator

To return the value:

```
@external
def retrieve() -> uint256:
    return self.my_favorite_number
```

This will create an 'orange' button in Remix, but when we click it, we don't get back the result. Why?

Well because in Vyper, if you use the above syntax, the compiler will assume you're trying to send a transaction.
It will actually waste gas that wasn't needed since you didn't write anything to the blockchain.

What we need is to add the `@view` decorator, to tell the compiler we don't want to send a transaction. This will make 'retrieve' a blue button.

A small caveat, remmeber that a view function can be called by a human any amount of times for free.
But if you call that same view function from within another contract, you will spend a bit of gas for that.

## Constructor

You already know what a constructor is, here it behaves exactly the same, this is the syntax:

```vyper
favorite_number: public(uint256)
owner: public(address)

@deploy
def __init__():
    self.favorite_number = 42
    self.owner = msg.sender
```

You add the `@deploy` decorator, then the function has to be named `__init__()`, that's it.

One of the most common uses of a constructor is to assign the variable `owner` to whichever address deployed the contract.

Another example:

```vyper
@deploy
def __init__(name: String[100], duration: uint256):
    self.owner = msg.sender
    self.name = name

    self.expiresAt = block.timestamp + duration
```

## Advanced functions

First, a comment, to divide two integers you have to do `//`:

````
def divide(x: uint256, y: uint256) -> uint256:
    return x // y
``

Sometimes you want to declare a function, but don't want to implement the code yet, you just want to make sure that the contract compiles. In this case, you can use the keyword `pass`:

```vyper
@external
def todo():
    pass
```

Finally we can return several values from a function:

```vyper
@external
@pure
def return_many() -> (uint256, bool):
    return (123, True)
```
## ❓ Questions and Resources

- [Vyper Docs](https://docs.vyperlang.org/en/stable/)
- [Repo Link](https://github.com/Cyfrin/moccasin-full-course-cu)
- [Remix](https://remix.ethereum.org/)

``

```

```

```

```

```

```

```

```

```

```

```

```
````
