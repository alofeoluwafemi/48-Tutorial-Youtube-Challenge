

## Build and Deploy an Onchain Calculator on Stacks Blockchain, using Clarity Programming Language.


## Introduction 

Just 2 weeks ago, I didn't know about the **stacks blockchain**, and beyond the usual speculating or holding of bitcoin like most people,  I didn't care to find out if its now possible to build dapps on the bitcoin blockchain.

Well, you can't blame me, becuase since the emergence of ethereum which purpose was to allow buidling of programable apps due to bitcoin failure of allowing that. I pretty much closed my mind to that possibility. 

Certainly, I will put that kind of idealogy in check as should you. In the blockchain space, new technologies are emerging every now and then and to keep up with pace you have to stay open and keep up to date.

That been said, lets get to it.

### What you Will  Learn
In this tutorial, you will learn the following:
- What is Stacks blockchain and why you should consider it
- Introdcution to Clarity, Stacks own version of Solidity language
- How to build and deploy a smart contract on Clarity testnet

### Difficulty
- Beginner :white_check_mark:

### Requirement
For this tutorial, you would require.

- Visual studio IDE 
- A linux or Mac terminal

A windows machine user too can also follow this article successfully, but I've not tried installing Clarity development environment on a windows machine. But if you followed any extra step that this article does not entail, be sure to leave a comment for others.

## Table of Content
- Overview
	- What is stacks
	- Stacking
	- How does stack settles transaction
	- Why you should consider using stacks
	- What stack blockchain is not
- Clarity Lanaguage
- Setup Clarity Software
- Setup Project
- Deploy Contract on Testnet
	-  Install Hiro wallet
	- Fund wallet from faucet
	- Deploy

## Overview

### What is Stacks

Stacks 2.0 is a layer-1 blockchain that connects to Bitcoin and brings smart contracts and decentralized apps to it. Smart contracts and apps developed on the Stacks platform are natively integrated with the security, stability, and economic power of Bitcoin.

### Stacking

Bitcoins used for miner bids are sent to a set of specific addresses corresponding to Stacks (STX) tokens holders that are actively participating in consensus ("Stackers"). Thus, rather than being destroyed, the bitcoins consumed in the mining process go to productive Stacks holders as a reward based on their holdings of Stacks and participation in the Stacking algorithm.

### How does stack settles transaction
Stacks layer 1 blockchain connects to Bitcoin by its consensus mechanism called PoX(Proof of Transfer), that spans the two chains. This enables Stacks to leverage Bitcoin’s security and enables Stacks apps to use Bitcoin’s state, despite being a separate blockchain.

PoX connects to Bitcoin with a 1:1 block ratio, meaning anything that happens on the Stacks blockchain can be verified on the Bitcoin Blockchain.

Transactions are the fundamental unit of execution in the Stacks blockchain. Each transaction is originated from a Stacks 2.0 account, and is retained in the Stacks blockchain history for eternity. This guide helps you understand Stacks 2.0 transactions.

![How does stack settles transaction](https://docs.stacks.co/assets/images/tx-lifecycle-6cd7e053b6eff31fdc23d3e51f6e96fb.png)


### Why you should consider using stacks

- **Unlocking Bitcoin capital:** Before now, bitcoin unlike ethereum cannot be used on Defi platform as much as ethereum its counterpart. Stacks unlocks the hundreds of billions in capital in Bitcoin, and gives Bitcoiners new opportunities to use and earn BTC providing new oppurtunities to dapp developers.  

- **Bitcoin’s security:** A transaction is nearly impossible to reverse or change once it has settled on Bitcoin. These settlement assurances are critical to the base layer of a network of decentralized apps. Settling the system on Bitcoin grants Stacks novel security properties not seen in other blockchains — it guarantees that all Stacks forks are public and helps nodes identify the canonical Stacks fork and find Stacks blocks they have not yet downloaded.

- **Bitcoin’s network effects:** The world is converging on Bitcoin as a standard, and demand for use cases around Bitcoin is increasing. There is hundreds of billions of capital locked into Bitcoin, most of it as a passive store of value. Instead of competing with Bitcoin, Stacks can provide novel use cases for holders of bitcoin, such as bitcoin-backed loans, bitcoin DeFi or using your bitcoin to trade NFTs.

### What Stack blockchain is not

- **Stacks is NOT a PoS chain:** The act of block production requires an extrinsic expenditure — it is not tied to owning the native token, as would be required in PoS. The only way to produce blocks in the Stacks chain is to transfer Bitcoin to a predetermined randomized list of Bitcoin addresses.  The Stacks blockchain can fork, and there exists a protocol to rank forks by quality independent of the set of miners and the tokens they hold, unlike PoS chain which cannot fork due to the inability to identify a canonical fork without trusting a 3rd party to decree a particular fork as valid.
- **Stacks is NOT a sidechain:** First, the history of all Stacks blocks produced is recorded to Bitcoin. This means that the act of producing a private Stacks fork is at least as hard as reorging the Bitcoin chain. Second, the Stacks blockchain has its own token; it does not represent pegged Bitcoin. This means that the safety of the canonical fork of the Stacks blockchain is underpinned by its entire token economy’s value, whereas the safety of a sidechain’s canonical fork is underpinned only by whatever system-specific measures incentivize its validators to produce blocks honestly.
- **Stacks is NOT a layer-2 system for Bitcoin:**  it's a sovereign system in its own right. The Stacks blockchain state is distinct from Bitcoin, and is wholly maintained by and for Stacks nodes. Stacks transactions are separate from Bitcoin transactions. Layer-2 systems like Lightning are designed to help scale Bitcoin payment transactions, whereas Stacks is designed to bring new use-cases to Bitcoin through smart contracts.
- **Stacks is NOT a merged-mined chain:**  The only block producers on the Stacks chain are Stacks miners. Bitcoin miners do not participate in Stacks block validation, and do not claim Stacks block rewards.

## Clarity Lanaguage

Clarity is a new language for smart contracts on the Stacks blockchain. The Clarity smart contract language optimizes for predictability and security.

Clarity is distinct from other languages designed for writing smart contracts in a few ways:

**Predictability**: The Clarity language uses precise and unambiguous syntax that allows developers to predict exactly how their contracts will be executed.
**Security**: The Clarity language allows users to supply their own conditions for transactions that ensure that a contract may never unexpectedly transfer a token owned by a user.
**No compiler**: Contracts written in Clarity are broadcasted on the Stacks blockchain exactly as they are written by developers. This ensures that the code developers wrote, analyzed, and tested, is exactly what gets executed.

One of the first thing you need to understand while learning a new langue is the Types. In Clarity there are 11 types system listed on the documentation which includes the follwoing.

| Type | Description |
|--|--|
|int  | signed 128-bit integer |
|uint  | unsigned 128-bit integer |
|bool  | boolean value (`true` or `false`) |
|principal  | object representing a principal (whether a contract principal or standard principal) |
|(buff max-len)  | byte buffer of maximum length `max-len` |
|(string-ascii max-len)  | ASCII string of maximum length `max-len` |
|(string-utf8 max-len)  | UTF-8 string of maximum length `max-len` (u"A smiley face emoji \u{1F600} as a utf8 string") |
|(list max-len entry-type)  | list of maximum length `max-len`, with entries of type `entry-type` |
|{label-0: value-type-0, label-1: value-type-1, ...}  | tuple, group of data values with named fields |
|(optional some-type)  | an option type for objects that can either be `(some value)` or `none` |
|(response ok-type err-type)  | object used by public functions to commit their changes or abort. May be returned or used by other functions as well, however, only public functions have the commit/abort behavior. |

If you are coming from the Solidity programming world to Clarity. Things can be a little bit weired at first. With no opening and closing curly braces, instead many closing and opening brackets, it can tend to get confusing, but stick with me, you will soon deploy your first contract on Stacks.

Comment in Clarity
```js
;; This is a comment
```
Adding two numbers in clarity
```js
(+ 1 2 3) ;; Returns 6
(+ 9 -3) ;; Returns 6
```

The same above sytax applies several mathematical expression.

Get a variable value
```js
(define-data-var cursor int 6)  
(var-get cursor) ;; Returns 6
``` 

Set a variable value
```js
(define-data-var cursor int 6)
(var-get cursor) ;; Returns 6
(var-set cursor (+ (var-get cursor) 1)) ;; Returns true
(var-get cursor) ;; Returns 7
```

If statement example
```js
(if true 1 2) ;; Returns 1 

;; If this expression (1 > 2) is true return 1 else 2 
(if (> 1 2) 1 2) ;; Returns 2 
```
Defining a private function with name `max-of` that takes two parameters `i1` and `i2` of type integer and return which is greater will be defined as below.

```js
(define-private (max-of (i1 int) (i2 int))
  (if (> i1 i2)
      i1
      i2))
(max-of 4 6) ;; Returns 6
```
You get the Idea? 

In Clarity, you can define functions as `define-private` , `define-public`, `define-read-only`.

Now that we have touched enough bascis of what Stack blockchain is and Clarity programming language. Let us write and deploy a smart contract on Clarity testnet, our smart contract is a simple one with functions to add, multiply, subtract and divide numbers of type integers.

## Setup Clarity Software

Before we start writing any code, we need to setup twi things.

1. Install **Clarinet**, runtime packaged as a command line tool, designed to facilitate smart contract development. Think of it as hardhat console or Truffle console.
2. Install **Clarity plugin for Visaul studio** editor for syntax higlighting and auto suggestions.

## Clarinet Installation

### Install on macOS (Homebrew)

```bash
brew install clarinet
```

### Install on Windows

```bash
winget install clarinet
```

### Install from a pre-built binary
```bash
wget -nv https://github.com/hirosystems/clarinet/releases/download/v0.27.0/clarinet-linux-x64-glibc.tar.gz -O clarinet-linux-x64.tar.gz
tar -xf clarinet-linux-x64.tar.gz
chmod +x ./clarinet
mv ./clarinet /usr/local/bin
```
You can read more about other installation options [here](https://github.com/hirosystems/clarinet).

Upon successful installation, open your terminal and run 
```bash
clarinet
```

Usage of how to use the command will be printed in the terminal. 

```bash
clarinet-cli 0.33.0

USAGE:
    clarinet <SUBCOMMAND>

OPTIONS:
    -h, --help       Print help information
    -V, --version    Print version information

SUBCOMMANDS:
    chainhooks      Subcommands for working with chainhooks
    check           Check contracts syntax
    completions     Generate shell completions scripts
    console         Load contracts in a REPL for an interactive session
    contracts       Subcommands for working with contracts
    dap             Step by step debugging and breakpoints from your code editor (VSCode, vim,
                        emacs, etc)
    deployments     Manage contracts deployments on Simnet/Devnet/Testnet/Mainnet
    help            Print this message or the help of the given subcommand(s)
    integrate       Start a local Devnet network for interacting with your contracts from your
                        browser
    lsp             Get Clarity autocompletion and inline errors from your code editor (VSCode,
                        vim, emacs, etc)
    new             Create and scaffold a new project
    requirements    Interact with contracts deployed on Mainnet
    run             Execute Clarinet extension
    test            Execute test suite

```

### Visual Studio Clarity Setup
Open your visual studio

## Setup Project

Navigate to your development project directory and run the following command. Enter **Y** when requested to do so, you will get a bunch of logs about directory and files created if this was successful.

```bash
clarinet new clarity-calculator-demo
```

The log may look somewhat like this
 ![Clarinet output](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-24+at+6.34.15+PM.png)

Navigate to the newly created directory from your terminal
```bash
cd clarity-calculator-demo
```

And inside the clarity-calculator-demo directory, create a new smart contract with name summation by running.

```bash
clarinet contract new summation
```

This will create `summation.clar` in the contracts directory and `summation_test.ts` in the tests directory. Open the your Visual studio IDE and add the first function to add two numbers of type integer in `contracts/summation.clar`

By default the place holder comment there will be 

```js
;; summation
;; <add a description here>

;; constants
;;

;; data maps and vars
;;

;; private functions
;;

;; public functions
;;
```

Replace it with 

```js
;; summation
;; Calculator contract

;; Adds two integer inputs and returns the result. 
;; In the event of an overflow, throws a runtime error.
(define-read-only (add-number (a int) (b int) ) (ok (+ a b)))
```

Here we define a `read-only` function `add-number` i.e it cannot modify data. The function takes in two integer `a` and `b` and sum them together. It then returns the result back. Thekeyword `ok` is used to return value in this function as seen above.

Similarly we can add more functions to subtract, multiply and divide two number of type integers.

```js
;; Subtract two integer inputs and returns the result. 
;; In the event of an overflow, throws a runtime error.
(define-public (sub-number (a int) (b int) ) (ok (- a b)))

;; Multiply two integer inputs and returns the result. 
;; In the event of an overflow, throws a runtime error.
(define-public (mul-number (a int) (b int) ) (ok (* a b)))

;; Divide two integer inputs and returns the result. 
;; In the event of an overflow, throws a runtime error.
(define-public (div-number (a int) (b int) ) (ok (/ a b)))
```

Open your Visual studio IDE terminal in the project directory and open `clarinet console` by running.

```bash
clarinet console
```
![Clarity console](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-24+at+6.51.20+PM.png)

Table one list contracts we have and their respective functions and in our case we only have `summation` contract, while table two list prefunded addresses.

To interact with the function inside clarinet console. Let us try add two numbers, to do so run this in your console.

**Add Function**
```bash
(contract-call? .summation add-number 1 2)
```

Output:
```bash
(ok 3)
```

**Subtract Function**
```bash
(contract-call? .summation sub-number 5 2)
```
Output:
```bash
(ok 3)
```
**Multiply Function**
```bash
(contract-call? .summation mul-number 5 2)
```
Output:
```bash
(ok 10)
```

**Divide Function**
```bash
(contract-call? .summation div-number 6 2)
```
Output:
```bash
(ok 3)
```

## Deploy Contract on Testnet

### Install Hiro Wallet
To deploy on Stacks blockchain, we would be needing a deployer wallet with some test token. 

Stacks blockchain is compatitble with Hiro Wallet, so head to chrome store and download the [browser extension](https://chrome.google.com/webstore/detail/hiro-wallet/ldinpeekobnhjjdofggfgjlcehhmanlj).

Setup **Hiro Wallet** by clicking on **Create New Wallet** and following the prompt. Copy the seed phrase of the wallet and use it to replace mnemonic value in `settings/Testnet.toml` 

```toml
mnemonic = "<YOUR PRIVATE TESTNET MNEMONIC HERE>"
```

### Fund Wallet in Faucet

Switch your wallet network from mainnet to testnet.

![Request Stacks token](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-19+at+7.56.39+PM.png)

Visit https://explorer.stacks.co/sandbox/faucet?chain=testnet to request some test STX tokens. Connect your Hiro wallet and click on **Request STX**

![enter image description here](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-19+at+7.54.50+PM.png)

![enter image description here](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-19+at+7.55.13+PM.png)

You will receive **500 STX** in your wallet.  Now we are ready to deploy our smart contract to testnet.

### Deploy

In your terminal, run teh below command to generate the deployment file for testnet.

```bash
clarinet deployments generate --testnet
```

Output:

```bash
Generated file deployments/default.testnet-plan.yaml
```

Use this deployment file, to deploy to testnet.

```bash
clarinet deployments apply -p deployments/default.testnet-plan.yaml
```

Output
```bash
The following deployment plan will be applied:
---
id: 0
name: Testnet deployment
network: testnet
stacks-node: "https://stacks-node-api.testnet.stacks.co"
bitcoin-node: "http://blockstack:blockstacksystem@bitcoind.testnet.stacks.co:18332"
plan:
  batches:
    - id: 0
      transactions:
        - contract-publish:
            contract-name: summation
            expected-sender: ST1HBAYT92N9T7XW7PZ20FTDQKKPSTTCQRZ63VR4S
            cost: 7200
            path: contracts/summation.clar
            anchor-block-only: true


Total cost:	0.00007200 STX
Duration:	1 blocks

Continue [Y/n]?
y
✔ Transactions successfully confirmed on Testnet
```

### Congratulations 🎉🎉
You have successfully written and deployed a smart contract on stacks blockchain.

### Confirmation 

If you go back to the sandbox https://explorer.stacks.co/sandbox/faucet?chain=testnet and clikc on the left hamburger menu to see the last transaction history. You will see its the new deployed contract and the balance has reduced by a very small fraction because of the fee paid for the deployment.

![Hiro Wallet](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-24+at+8.17.17+PM.png)
