# Section 3. Fund Me Frontend

## Introduction

In this section, we will delve into how MetaMask or your wallet interacts with websites and ensure that the transactions sent by the wallet are executed as intended.

## Setup

All the HTML and JS code is [here](https://github.com/Cyfrin/html-fund-me-cu)

## Intro to window.ethereum

First think of wallets. It's through a wallet like MetaMask that we are able to interact with the blockchain.

Inside the example frontend for FundMe, go to the browser console and type `window`.

There are some properties of this object which are not there by default, one of which is `window.ethereum`. It's through this property that a front end is able to interact with our wallet and it's accounts.

Recommended reading the [Metamask docs](https://docs.metamask.io/wallet/) to learn more about the window.ethereum object.
