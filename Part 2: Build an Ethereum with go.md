## Part 2: Building a Complete API to Interact with an P2P Escrow Smart Contract using go-ethereum Client & GoFiber Framework

This is the second part of the two post in this series
- [Part 1: Project Overview, Setup Fiber, Generate Go Binding, Deploy Contract, Add get Escrow & Wallet Logic Address endpoint](https://)
- [Part 2: Add and Conclude P2P API endpoints](https://)

You can fork the codebase here on [github](https://github.com/alofeoluwafemi/smart-contract-api-go-ethereum)  as a reference to follow along. 

## Introduction

[Go ethereum](https://geth.ethereum.org/) is the official go implementation of the ethereum protocol. Miners can use it to run an ethereum node and Dapp developers such as you can use some of its packages to compile your smart contract to bytecode, deploy it to a network and use it to interact with your deployed smart contract.

[Go Fiber framework](https://gofiber.io/) on the other hand is a Go web framework built on top of Fasthttp, the fastest HTTP engine for Go. Its similarities to Node express framwork and ease of use makes it attractive to new go developer to use for building API's for thier project. I for one found it easy to relate and quickly build with when I started geting into writing go lang.

### What you Will  Learn

For this tutorial, I would be using an existing smart contract which is one of the first draft for an Escrow to statisfy a P2P platform  on [Pay by PiggyFi](https://piggyfi.africa), it was written during one of the Hackthons we participated in as a Proof of concept. You can [find the codebase here](https://github.com/orignellc/piggyfi-contracts-poc) and since the goal of this series is not to focus on building smart contract but  an API in go to interact with it, this contract serve our usage. 

And don't worry, I will be doing a breakdown of the smart contract shortly below


### Difficulty
- Beginner :white_check_mark:
- Intermidiate :white_check_mark:

### Requirement
To get the best out of this tutorial, some [basic go lang](https://go.dev/learn/) understanding is needed. For my editor of choice I will be using [goland](https://www.jetbrains.com/go/), which in my opinion is by far the best go lang editor out there.

Other than that having latest [solidity compile installed](https://docs.soliditylang.org/en/v0.8.9/installing-solidity.html) and obviously the latest [go compiler](https://go.dev/doc/install) is a requirement.

Also make sure to get some [test ETH from Goerli](https://goerlifaucet.com/) , it will be needed to pay gas fees while deploying and interacting with the smart contracts.

### Table of Content

#### Part 1 of this article covers
- Project Overview
- Setup Fiber, .env & Install dependecies
- Deploy Smart Contracts on Goeril Testnet
- Generate Go Binding for Contracts
- Build out API Endpoints
	- POST request to Deploy Contracts
		- USDC
		- Factory
	- GET request to get CustodianWalletLogic  Address
	- GET request to get Escrow Contract Address

####  Part 2 of this article covers
- POST request to set USDC address on the Escrow contract
- POST request to Deploy a new custodian wallet address
- POST request to open a buy P2P order with USDC Balance
- POST request to open a sell P2P order with USDC Balance
- Handle Revert Errors in response
- GET request to Get open orders for a vendor
- GET request to Get order by ID

## P2P Order Webhooks

### Set USDC Address

In the first part of this article we deployed a USDC contract, which  I mentioned serves as a Mock USDC used as means of exchange in or P2P system against Fiat. In production environment we will change the USDC address to the actual USDC address used on ethereum or USDT or Dai whichever Stable currency suit our purpose works.

Just as we have been doing previously, first add route to set USDC in the `api_router.go` file.

```go
escrow := api.Group("/escrow")

escrow.Post("/set-usdc-address", controllers.SetUSDCTokenAddress)
```

In the controller package, create `escrow_controllers.go` to hold controller methods for Escrow contract related actions.

```go
package controllers

import (
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/blockchain"
	"github.com/gofiber/fiber/v2"
)

func SetUSDCTokenAddress(c *fiber.Ctx) error {
	conn := blockchain.CurrentConnection
	type Request struct {
		Address string `json:"address"`
	}

	request := new(Request)

	if err := c.BodyParser(request); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
			"status":  "error",
			"message": "Malformed data",
			"data":    err,
		})
	}

	err := conn.SetUSDCAddress(request.Address)

	if err != nil {
		return c.Status(fiber.StatusUnauthorized).JSON(fiber.Map{
			"status":  "error",
			"message": err,
			"data":    nil,
		})
	}

	return c.Status(fiber.StatusOK).JSON(fiber.Map{
		"status": "success",
		"data":   request.Address,
	})
}
```

And finally in the blockchain package, add a new file `escrow.go` and add the following.

```go
package blockchain

import (
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/ethereum"
	"github.com/ethereum/go-ethereum/common"
	"log"
)

var EscrowInstance *Escrow

const (
	EscrowAddress = "0xd27adc3848dE1324AF87e5C235355e4a017Aa1CF"
)

type Escrow struct {
	Address  common.Address
	Instance *ethereum.Escrow
}

func (clientCon ClientConnection) newEscrow(address string) *ethereum.Escrow {
	EscrowInstance = new(Escrow)

	contractAddress := common.HexToAddress(address)

	EscrowInstance.Address = contractAddress

	instance, err := ethereum.NewEscrow(contractAddress, clientCon.Client)
	if err != nil {
		log.Fatalln("Cannot get Factory contract at address ", address, " due to: ", err)
	}

	return instance
}

func (clientCon ClientConnection) SetUSDCAddress(address string) error {

	_, err := getEscrow().SetUsdcTokenAddress(clientCon.trxOpts, common.HexToAddress(address))

	if err != nil {
		log.Printf("Cannot set new USDC token on Escrow due to: %v", err)

		return err
	}

	return nil
}

func getEscrow() *ethereum.Escrow {
	return CurrentConnection.newEscrow(EscrowAddress)
}
```

This doesn't need more explanation, since its the same format we have followed previously.  If you have not read Part 1 of this series, I suggest you do.

Restart fiber server and  in Postman make a POST request to http://127.0.0.1:3000/api/v1/escrow/set-usdc-address the Factory contract will be deployed.

**Success API Call**

![Postman call smart contract](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-13+at+7.44.48+PM.png)


**Error API Call**

This will happen after you make the second request to the API just immediately which uses the same nonce.
 
![Postman call smart contract](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-13+at+7.42.17+PM.png)


### Create Custodian Wallet

The next route we will create will be `/api/v1/factory/new-wallet/:uuid`, such that when we make a POST rquest to this endpoint it calls the function `newCustodian` on the Factory contract. This will Deploy a Proxy Contract called `CustodianWalletProxy` that delegates call to `CustodianWalletLogic`. 

 To begin with, as usual. Add the route to `api_router.go`.
```go
api.Post("/factory/new-wallet/:uuid", controllers.NewWallet)
```

Then next the controller, this time inside the `factory_controller` file.

```go
func NewWallet(c *fiber.Ctx) error {
	conn := blockchain.CurrentConnection
	uuid := c.Params("uuid")

	trx, err := conn.NewWallet(uuid)

	if err != nil {
		return c.Status(fiber.StatusUnauthorized).JSON(fiber.Map{
			"status":  "error",
			"message": err,
			"data":    nil,
		})
	}

	return c.Status(fiber.StatusOK).JSON(fiber.Map{
		"status": "success",
		"hash":   trx.Hash(),
	})
}
```

As you can see, to create a new wallet, the smart contract requires a unique identifier to save it in a mapping which is accepted in the controller via a URL parameter. And each unique ID cannot be assigned to another user as you would see shorlty.

Finally add the `NewWallet` method to the `blockchain` package in `factory.go` file.

```go
func (clientCon ClientConnection) NewWallet(uuid string) (*types.Transaction, error) {

	trx, err := getFactory().NewCustodian(clientCon.trxOpts, uuid)

	if err != nil {
		log.Printf("Cannot create new wallet: %v", err)

		return new(types.Transaction), err
	}

	return trx, nil
}
```

Restart the Fiber server as usual and make a POST request to http://127.0.0.1:3000/api/v1/factory/new-wallet/cec3dd14-339a-11ed-a261-0242ac120002  and http://127.0.0.1:3000/api/v1/factory/new-wallet/b93e42b0-33a2-11ed-a261-0242ac120002. This creates account for each uuid appended at the end of both URL's.

The purpose of creating two account is so that when we create order futher down the line we can use both accounts.

![enter image description here](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-14+at+1.26.58+AM.png)

If you retry again with the same uuid, you will get account exist error. 
![enter image description here](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-14+at+1.14.36+AM.png)

### Use UUID to get the wallet address

This will be a no brainer, add the route to get the wallet address using UUID.

```go
api.Get("/factory/wallet/:uuid", controllers.GetWallet)
```

In the `factory_controller.go` 

```go
func GetWallet(c *fiber.Ctx) error {
	conn := blockchain.CurrentConnection
	uuid := c.Params("uuid")

	address, err := conn.GetAccountByUUID(uuid)

	if err != nil {
		return c.Status(fiber.StatusUnauthorized).JSON(fiber.Map{
			"status":  "error",
			"message": err,
			"data":    nil,
		})
	}

	return c.Status(fiber.StatusOK).JSON(fiber.Map{
		"status": "success",
		"hash":   address.String(),
	})
}
```

In the blockchain package, add the `GetAccountByUUID` method to call the contract method. 

```go
func (clientCon ClientConnection) GetAccountByUUID(uuid string) (*common.Address, error) {

	address, err := getFactory().Accounts(clientCon.callOpts, uuid)

	if err != nil {
		log.Printf("Cannot get account: %v due to error %v", uuid, err)

		return new(common.Address), err
	}

	return &address, nil
}
```

Restart the server and using Postman, make a GET request to http://127.0.0.1:3000/api/v1/factory/wallet/b93e42b0-33a2-11ed-a261-0242ac120002. Remember to change the uuid to the once you used ealier on and note both account address returned.

In my own case my two acccount address are 

| UUID | Account Address |
|--|--|
| b93e42b0-33a2-11ed-a261-0242ac120002 | 0xC1F07Db647Aa3002c12BbaF8D598F0ef19c4ddd3 |
|cec3df26-339a-11ed-a261-0242ac120002|0x63cc2DD4d1836bA66B40B3dEBfcA7896256c24d0|