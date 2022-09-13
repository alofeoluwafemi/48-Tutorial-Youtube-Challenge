


## Part 1: Building a Complete API to Interact with an P2P Escrow Smart Contract using go-ethereum Client & GoFiber Framework

This is the first of the two post of this series
- [Part 1: Project Overview, Setup Fiber, Generate Go Binding, Deploy Contract, Add get Escrow & Wallet Logic Address endpoint](https://)
- [Part 2: Add and Conclude Orders endpoint](https://)

## Introduction

[Go ethereum](https://geth.ethereum.org/) is the official go implementation of the ethereum protocol. Miners can use it to run an ethereum node and Dapp developers such as you can use some of its packages to compile your smart contract to bytecode, deploy it to a network and use it to interact with your deployed smart contract.

[Go Fiber framework](https://gofiber.io/) on the other hand is a Go web framework built on top of Fasthttp, the fastest HTTP engine for Go. Its similarities to Node express framwork and ease of use makes it attractive to new go developer to use for building API's for thier project. I for one found it easy to relate and quickly build with when I started geting into writing go lang.

## What you Will  Learn

For this tutorial, I would be using an existing smart contract which is one of the first draft for an Escrow to statisfy a P2P platform  on [Pay by PiggyFi](https://piggyfi.africa), it was written during one of the Hackthons we participated in as a Proof of concept. You can [find the codebase here](https://github.com/orignellc/piggyfi-contracts-poc) and since the goal of this series is not to focus on building smart contract but  an API in go to interact with it, this contract serve our usage. 

And don't worry, I will be doing a breakdown of the smart contract shortly below


## Difficulty
- Beginner :white_check_mark:
- Intermidiate :white_check_mark:

## Requirement
To get the best out of this tutorial, some [basic go lang](https://go.dev/learn/) understanding is needed. For my editor of choice I will be using [goland](https://www.jetbrains.com/go/), which in my opinion is by far the best go lang editor out there.

Other than that having latest [solidity compile installed](https://docs.soliditylang.org/en/v0.8.9/installing-solidity.html) and obviously the latest [go compiler](https://go.dev/doc/install) is a requirement.

Also make sure to get some [test ETH from Goerli](https://goerlifaucet.com/) , it will be needed to pay gas fees while deploying and interacting with the smart contracts.

## Table of Content

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

## Project Overview
This version of the Piggyfi Smart contract is simple Escrow system to carry out a P2P transaction between a vendor and a customer who wants to exchange Fiat for USD.

**Factory:** This smart contract is responsible for deploying a new CustodianWalletProxy which serve as a wallet for a vendor or customer everytime a new user signs up depending on the user profile. 
**Escrow:** This is the actual Escrow contract that carries out actions on orders for both vendors and customers
**CustodianWalletLogic:** This contract is an upgradable contract that holds all the Logic of what a wallet can do.
**CustodianWalletProxy:** This is the contract that is deployed as a wallet for every users. It uses delegate call to use actual Logic in the CustodianWallet Logic contract, in itself does't hold any meaningful functions asides the call back methods and contructor.
**Types:** This  hold common variable shared between the Proxy and Logic Wallet contracts to avoid the dread storage overiding problem common to not well written proxy contracts.
**USDC:** This contract is never deployed to production, its simply used for Mocking USDC in test environment to give us flexibility and control over minting. In Production the address will be for the USDC used in the Escrow contract.

## Setup Fiber, .env & Install dependecies

If you are using Goland IDE, creating a new project is as simple as opening your IDE, and on the menu click on **File > New > Project**. Don't forget to change the default name `awesomeProject` to your desired name, I used `go-ethereum-api` for mine.

![Goland IDE New Project](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-07+at+5.36.23+AM.png)

For other IDE such as VScode, create your project director , then from the terminal, run command below to to generate your `go.mod` file.

```
go mod init github.com/GITHUB_USER_NAME/go-ethereum-api
```

Remember to substitute `GITHUB_USER_NAME` with your github username.

**Next**, Install Fiber framework by running

```bash
go get github.com/gofiber/fiber/v2
```

Now, you want to create a `package main.go` file and paste the below code inside.

```go
package main

import (
	"github.com/gofiber/fiber/v2"
	"log"
)

func main() {
	app := fiber.New()

	app.Get("/", func(c *fiber.Ctx) error {
		return c.SendString("Hello, World!")
	})

	err := app.Listen(":3000")
	if err != nil {
		return
	}
}
```

Run, from your terminal
```
go run server.go
```

You should see somthing similar printed in your terminal to indicate everything is running well.

```
 ┌───────────────────────────────────────────────────┐ 
 │                   Fiber v2.37.0                   │ 
 │               http://127.0.0.1:3000               │ 
 │       (bound on host 0.0.0.0 and port 3000)       │ 
 │                                                   │ 
 │ Handlers ............. 2  Processes ........... 1 │ 
 │ Prefork ....... Disabled  PID ............. 57556 │ 
 └───────────────────────────────────────────────────┘ 
```

Visit http://127.0.0.1:3000 and you will see **Hello, World!**. Not much but setting up http server was that fast using go Fiber.

Now let us properly setup our app structure but before we go ahead, install the following dependecies.

```bash
go get github.com/joho/godotenv
go get github.com/ethereum/go-ethereum
```

### Folder Structure

Update your project structure to match the image below. Don't worry everything will make sense soon.

![Folder structure](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-07+at+10.30.00+AM.png)

**router:** The router package is where all of our routes will be defined. Here we defined a router interface that allows us have diferent type of routers such as api routers, http routers e.t.c such that its possible to define methods on the interface that all the router must implement and we can be sure our routers will behave in a certain way as you would see below is the case of `InstallRouter` method.

**pkg/router/router_interface.go**
```go
package router

import "github.com/gofiber/fiber/v2"

type Router interface {
	InstallRouter(app *fiber.App)
}
``` 

**pkg/router/api_router.go**
```go
package router

import (
	"github.com/alofeoluwafemi/go-ethereum-api/app/controllers"
	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/limiter"
)

type ApiRouter struct {
}

func (h ApiRouter) InstallRouter(app *fiber.App) {
	api := app.Group("/api/v1", limiter.New())

	api.Get("/", controllers.RenderHello)
}

func NewApiRouter() *ApiRouter {
	return &ApiRouter{}
}
```
**pkg/router/setup.go**

```go
package router

import (
	"github.com/gofiber/fiber/v2"
)

func InstallRouter(app *fiber.App) {
	setup(app, NewApiRouter())

	//If no route was matched
	app.Use(func(c *fiber.Ctx) error {
		err := c.SendStatus(404)
		panic(err)
	})
}
func setup(app *fiber.App, router ...Router) {
	for _, r := range router {
		r.InstallRouter(app)
	}
}
```

⚠️  Remember to substitute the github username in the imports with the one corresponding to your `go.mod`.

You will notice that we imported the controller package in the route but it can't be resolved. Next, let us fix that.

**controller:** The purpose of the controller package is simply straight forward, and it it to abstarct out logic from the route package and keep things simple.

**app/controllers/main_controller.go**

```go
package controllers

import "github.com/gofiber/fiber/v2"

func RenderHello(c *fiber.Ctx) error {
	return c.Status(fiber.StatusOK).JSON(fiber.Map{
		"message": "Hello from api",
	})
}
```

**env:** The env package is a helper utility that uses the `godotenv` package to read environment variable we dont want to publish such as our private key and Alchemy API key.

**pkg/env/env.go**

```go
package env

import "github.com/joho/godotenv"

var Env map[string]string

func GetEnv(key, def string) string {
	if val, ok := Env[key]; ok {
		return val
	}
	return def
}

func SetupEnvFile() {
	envFile := ".env"
	var err error
	Env, err = godotenv.Read(envFile)
	if err != nil {
		panic(err)
	}

}
```

**bootstrap:** The boostrap package is what you can refer to as the app ignition. It creates a new **App** named instance, registers router, middleware and if you want to initialize your database in the future this is where you can do so.

**bootstrap/boostrap.go**

```go
package bootstrap

import (
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/env"
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/router"
	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/logger"
	"github.com/gofiber/fiber/v2/middleware/recover"
)

func NewApplication() *fiber.App {
	env.SetupEnvFile()
	app := fiber.New(fiber.Config{
		AppName: "Go Ethereum API",
		CaseSensitive: true,
	})

	//Middleware
	app.Use(recover.New())
	app.Use(logger.New())

	//Setup routes
	router.InstallRouter(app)

	return app
}
``` 

Finally in your `main.go` replace everything with this.

```go
package main

import (
"fmt"
"github.com/alofeoluwafemi/go-ethereum-api/bootstrap"
"github.com/alofeoluwafemi/go-ethereum-api/pkg/env"
"log"
)

func main() {
	app := bootstrap.NewApplication()
	log.Fatal(app.Listen(
		fmt.Sprintf("%s:%s",
			env.GetEnv("APP_HOST", "localhost"),
			env.GetEnv("APP_PORT", "3000"),
		)))
}
```

And in your `.env` file update the details to match below. Don't worry about the Private key and Alchemy Api key just yet, in time we will update it.

**.env**

```bash
DEPLOYER_PRIVATE_KEY=0x0
ALCHEMY_KEY=
APP_HOST=localhost
APP_PORT=3000
```

Stop the currently running `go run main.go` process and restart it. If you visit the only registered route we have `http://127.0.0.1:3000/api/v1`  you should see json return.

```json
{
	"message": "Hello from api"
}
```

## Deploy Smart Contracts on Goeril Testnet

### Clone Contract
Clone the Piggyfi smart contract repository into `smart-contracts` folder inside our project. To do so run the command below.

```bash
git clone https://github.com/orignellc/piggyfi-contracts-poc.git smart-contracts
```

Open the 

### Compile contracts

Run `npm install` in the smart-contracts directory to get the openzeppelin dependecy that has smart contracts refrenced by some of our contracts

```bash
solc --abi smart-contracts/contracts/*.sol -o builds --bin --include-path smart-contracts/node_modules --base-path .
```

`--base-path` represents the root of your own source tree while `--include-path` allows you to specify extra locations containing external code.

If you look inside the `builds` folder you would see a list of `.abi` and `.bin` files generated.

### Generate Contract Go  Bindings

**abigen** is a package that comes with go-ethereum. It is a source code generator to convert Ethereum contract definitions into easy-to-use, compile-time type-safe Go packages.

Simply put generate go code for our smart contract, so that we can interact with our contract in go code.

To install **abigen**

```bash
cd $GOPATH/src/github.com/ethereum/go-ethereum/
make
make devtools
```

Once this is done, if you run `abigen` on the command line you should get an output  saying.
```bash
Fatal: No destination package specified (--pkg)
```

Now that we have `abigen` tool installed. Create two more new packages in the **pkg** folder, the first one is `go-binding`. 

And inside `go-binding`, create a new file `generate_binding.go`  . This file will hold a script that will help us generate our go bining when we run it, so copy and paste the script below into the file. Its self explanatory but I will put some context to it shortly.

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"os/exec"
	"strings"
)

const (
	buildDir  = "../../builds/"
	targetDir = "../ethereum/"
)

func main() {
	contractNames := []string{"CustodianWalletLogic", "Factory", "Escrow", "USDC"}

	cmd := exec.Command("which","abigen")
	err := cmd.Run()

	if err != nil {
		log.Fatal("ethereum-abigen is not a command: ",err)
	}

	for _, contractName := range contractNames {
		buildFilename := buildDir + contractName
		bindingFilename := targetDir + contractName + ".go"
		cmd = exec.Command(
			"abigen", fmt.Sprintf("--bin=%s.bin", buildFilename),
			fmt.Sprintf("--abi=%s.abi", buildFilename),
			fmt.Sprintf("--pkg=%s", contractName),
			fmt.Sprintf("--out=%s", bindingFilename),
		)

		err := cmd.Start()
		if err != nil {
			log.Fatalln(err)
		}
		log.Printf("Running... %s", cmd)
		err =  cmd.Wait()
		if err != nil {
			log.Fatalln(err)
		}
		bindFile, err := ioutil.ReadFile(bindingFilename)
		if err != nil {
			log.Fatalln("Cannot open: ",bindingFilename, err)
		}

		fileOutput := strings.Replace(string(bindFile), "package " + contractName, "package ethereum", 1)
		err = ioutil.WriteFile(bindingFilename, []byte(fileOutput), 0644)
		if err != nil {
			log.Fatalln("Cannot write to: ",bindingFilename,err)
		}
	}
}
```

The script, loops through the supplied `contractNames` ,  appends the `.abi` and `.bin`  to the filename, then passes them as option to the `--abi` and `--bin`  of the  `abigen` command, to generate the go bindings.

Before running the go file, create the second package folder I mentioned earlier. Name it **ethereum**, don't forget this should be under **pkg**.

From your terminal,  in your project root run the following command.

```bash
cd pkg/go-binding 
go run generate_binding.go
```

If everything goes well, your output should look like this.
![Generate go binding](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-08+at+2.08.13+AM.png)

If you open up the ethereum package folder, you would see your generated go binding.

![Generate contract go binding](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-08+at+2.13.10+AM.png)

### Get Alchemy API Key

Alchemy is provides web3 development tools to build and scale your dApp. And we we would be making use of one of thier service that allows you connect to the blockchain node and in our case `Goerli`.

[Create a new account](https://dashboard.alchemy.com/) if you have not, click on the **+ Create App** button and select Ethereum and Goerli.

![Create App Alchemy](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+2.28.25+PM.png)
 
 And once the app is created, a list of current Apps you have will be listed. By the side of your newly created app, click on **View Key** and copy the key and url to the `.env` files as values to **ALCHEMY_KEY** and **RPC_URL**.
 ![Alchemy key](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+2.34.00+PM.png)
 By now your `.env` file should look like this.
```bash
DEPLOYER_PRIVATE_KEY=bef9e118a31a0af39585d7e82cae42a...  
ALCHEMY_KEY=obEYiMcZIMHXZC...
RPC_URL=https://eth-goerli.g.alchemy.com/v2/obEYiMcZIMHXZC...
APP_HOST=localhost
APP_PORT=3000
```

### Create Package Blockchain 
We need to create a, blockchain package in the `pkg` directory amd inside it create a new go file  `blockchain.go`.

![Create blockchain package](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+1.12.12+PM.png)

This package abstract most of how we connect to the ethereum node and communicate with our Smart contracts, to avoid code repetition and keep the controller lean. I will proceed by explaining each part of the code then we can put it together to see how it runs.

First we define a variable `CurrentConnection` that will be exported from this package. This variable will hold an instance of `ClientConnection`.

```go
var CurrentConnection *ClientConnection
```
Next we define `ClientConnection` struct, which holds a collection of details we need in communicating to the Ethereum node such as RPC URL, Call request options, e.t.c and also an instance to the node connection itself.

```go
type ClientConnection struct {
	RpcURL      string
	SignerKey   string
	Client      *ethclient.Client
	NetworkId   *big.Int
	NetworkName string
	ctx         context.Context
	callOpts    *bind.CallOpts
	trxOpts     *bind.TransactOpts
}
```

Some of the struct fields above such as the `RpcURL`, `NetworkName`, `NetworkId`, are self explanatory, I will take time to describe in more details the rest of the fields.

**SignerKey:** This will be picked from the environment file and its the, private key of the smart contract deployer without the `0x` prefix. 
**Client:** This holds the instance of the connection to the ethereum Alchemy node using.
**ctx:** The context used for all smart contract call 
**callOpts:** It is the collection of options used to make a contract call request that doesn't change state.
**trxOpts:** It is the collection of authorization data required to create valid Ethereum transaction that causes state changes.

Then we have the `New()` to create a new connection and set fields in the `ClientConnection`.

```go
func New() {
	clientCon := new(ClientConnection)
	clientCon.RpcURL = env.GetEnv("RPC_URL", "")

	client, err := ethclient.Dial(clientCon.RpcURL)
	defer client.Close()
	if err != nil {
		log.Fatalln("Could not connect to client: ", err)
	}

	clientCon.Client = client
	clientCon.SignerKey = env.GetEnv("DEPLOYER_PRIVATE_KEY", "")
	clientCon.NetworkName = "Goerli"
	clientCon.NetworkId = new(big.Int).SetInt64(5)
	clientCon.ctx = context.Background()

	clientCon.setTransactOpts()
	clientCon.setCallOpts()

	CurrentConnection = clientCon
}
```

Add a method to extract Public key, Public key ECDSA from our Private key.

```go
func (clientCon *ClientConnection) extractSignerKeys() (*ecdsa.PrivateKey, standardcryto.PublicKey, *ecdsa.PublicKey, error) {
	privateKey, err := crypto.HexToECDSA(clientCon.SignerKey)
	if err != nil {
		log.Fatalln("Invalid private key: ", err)
	}

	publicKey := privateKey.Public()
	publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)

	if !ok {
		log.Fatalln("cannot assert type: publicKey is not of type *ecdsa.PublicKey")
	}

	return privateKey, publicKey, publicKeyECDSA, err
}
```
Add another method to get the next `Nonce` for any address passed to it. We mostly and might only need it to get the nonce for our deployer address.

```go
func (clientCon *ClientConnection) nonceAt(address common.Address) uint64 {

	nonce, err := clientCon.Client.PendingNonceAt(clientCon.ctx, address)

	if err != nil {
		log.Fatalln("Cannot get nonce: ", err)
	}

	return nonce
}
```

Finally we need methods to  set the `callOpts` and `trxOpts` fields on `ClientConnection`.

```go
func (clientCon *ClientConnection) setTransactOpts() *bind.TransactOpts {
	signer, _, publicKeyECDSA, _ := clientCon.extractSignerKeys()

	from := crypto.PubkeyToAddress(*publicKeyECDSA)

	opts, err := bind.NewKeyedTransactorWithChainID(signer, clientCon.NetworkId)
	if err != nil {
		log.Fatalln("Error with NewKeyedTransactorWithChainID ", err)
	}

	priorityFee, err := clientCon.Client.SuggestGasTipCap(clientCon.ctx)
	if err != nil {
		log.Fatalln("Cannot suggest Tip: ", err)
	}

	opts.Nonce = new(big.Int).SetUint64(clientCon.nonceAt(from))
	opts.GasTipCap = priorityFee

	clientCon.trxOpts = opts

	return opts
}
```

```go
func (clientCon *ClientConnection) setCallOpts() *bind.CallOpts {
	opts := new(bind.CallOpts)
	return opts
}
```

Putting it all together.

```go
package blockchain

import (
	"context"
	standardcryto "crypto"
	"crypto/ecdsa"
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/env"
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/ethereum"
	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
	"log"
	"math/big"
)

var CurrentConnection *ClientConnection

type ClientConnection struct {
	RpcURL      string
	SignerKey   string
	Client      *ethclient.Client
	NetworkId   *big.Int
	NetworkName string
	ctx         context.Context
	callOpts    *bind.CallOpts
	trxOpts     *bind.TransactOpts
}

func New() {
	clientCon := new(ClientConnection)
	clientCon.RpcURL = env.GetEnv("RPC_URL", "")

	client, err := ethclient.Dial(clientCon.RpcURL)
	defer client.Close()
	if err != nil {
		log.Fatalln("Could not connect to client: ", err)
	}

	clientCon.Client = client
	clientCon.SignerKey = env.GetEnv("DEPLOYER_PRIVATE_KEY", "")
	clientCon.NetworkName = "Goerli"
	clientCon.NetworkId = new(big.Int).SetInt64(5)
	clientCon.ctx = context.Background()

	clientCon.setTransactOpts()
	clientCon.setCallOpts()

	CurrentConnection = clientCon
}

func (clientCon *ClientConnection) nonceAt(address common.Address) uint64 {

	nonce, err := clientCon.Client.PendingNonceAt(clientCon.ctx, address)

	if err != nil {
		log.Fatalln("Cannot get nonce: ", err)
	}

	return nonce
}

func (clientCon *ClientConnection) setTransactOpts() *bind.TransactOpts {
	signer, _, publicKeyECDSA, _ := clientCon.extractSignerKeys()

	from := crypto.PubkeyToAddress(*publicKeyECDSA)

	opts, err := bind.NewKeyedTransactorWithChainID(signer, clientCon.NetworkId)
	if err != nil {
		log.Fatalln("Error with NewKeyedTransactorWithChainID ", err)
	}

	priorityFee, err := clientCon.Client.SuggestGasTipCap(clientCon.ctx)
	if err != nil {
		log.Fatalln("Cannot suggest Tip: ", err)
	}

	opts.Nonce = new(big.Int).SetUint64(clientCon.nonceAt(from))
	opts.GasTipCap = priorityFee

	clientCon.trxOpts = opts

	return opts
}

func (clientCon *ClientConnection) setCallOpts() *bind.CallOpts {
	opts := new(bind.CallOpts)
	return opts
}

func (clientCon *ClientConnection) extractSignerKeys() (*ecdsa.PrivateKey, standardcryto.PublicKey, *ecdsa.PublicKey, error) {
	privateKey, err := crypto.HexToECDSA(clientCon.SignerKey)
	if err != nil {
		log.Fatalln("Invalid private key: ", err)
	}

	publicKey := privateKey.Public()
	publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)

	if !ok {
		log.Fatalln("cannot assert type: publicKey is not of type *ecdsa.PublicKey")
	}

	return privateKey, publicKey, publicKeyECDSA, err
}

func (clientCon *ClientConnection) DeployUSDC() (common.Address, *types.Transaction) {
	contract, trx, _, err := ethereum.DeployUSDC(clientCon.trxOpts, clientCon.Client)

	if err != nil {
		log.Fatalln("Could not deploy USDC contract: ", err)
	}

	return contract, trx
}

```

Now that we have this methods in place, we can start calling methods on our Smart contracts. Open up `USDC.go` file under the ethereum package, around line `46` you will notice the function `DeployUSDC` which can be called to Deploy the `USDC` contract.

![Deploy USDC go Bindning function](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+3.28.19+PM.png)

We need to call this function from a controller and pass the two parameters required to it. Luckily we can provide both, but first under controllers, create a new go file `deployer_controller.go` then add the following to define a `DeployUSDC` Controller that will be called from `/api/v1/deploy/usdc`.

Inside `deployer_controller.go` paste this inside of it.

```go
package controllers

import (
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/blockchain"
	"github.com/gofiber/fiber/v2"
)

func DeployUSDC(c *fiber.Ctx) error {

	conn := blockchain.CurrentConnection

	address, transaction := conn.DeployUSDC()

	return c.Status(fiber.StatusOK).JSON(fiber.Map{
		"address": address.String(),
		"hash":    transaction.Hash(),
	})
}
```

The `DeployUSDC` method on the blockchain package is not yet defined. Let us fix that by define it, it simply calls the `DeployUSDC`  function on `USDC.go`. 

```go
func (clientCon *ClientConnection) DeployUSDC() (common.Address, *types.Transaction) {
	contract, trx, _, err := ethereum.DeployUSDC(clientCon.trxOpts, clientCon.Client)

	if err != nil {
		log.Fatalln("Could not deploy USDC contract: ", err)
	}

	return contract, trx
}
```

And inside `api_router.go`, add router to the new URL

```go
	deploy := api.Group("/deploy")
	deploy.Get("/usdc", controllers.DeployUSDC)
```

Finally if you stop and rerun the command to resstart the Fiber web server using.
```go
go run main.go
```

When you visit http://127.0.0.1:3000/api/v1/deploy/usdc you will get a response with contract address and hash of the transaction for deploying the USDC contract address.

```json
{
	"address": "0xb3f2504110eeea6a522218048D0519B829788DDD",
	"hash": "0xaeabc29be2bb6d2e84466edebf8e38b0ca833f250bc70eeb733509365b5f787b"
}
```

In my case if I visit etherscan and search using the transaction hash.

![Deploy Contract on Goerli](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+5.07.18+PM.png)

### Use POST instaed of GET

The current http method we are using on the deploy usdc route is GET, now change it to POST such that.

```go
deploy.Post("/usdc", controllers.DeployUSDC)
```

Again restart the server and from Postman call your URL and a new smart contract will be deployed. You can confirm using the transaction hash.

![Deploy Contract on Post man](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+5.24.09+PM.png)

Make sure you note the address of the Deployed contract somewher for our later usage.

Next let us create a new route to Deploy Factory contract. Right under the `DeployUSDC` add this controller function.

```go
func DeployFactory(c *fiber.Ctx) error {

	conn := blockchain.CurrentConnection

	address, transaction := conn.DeployFactory()

	return c.Status(fiber.StatusOK).JSON(fiber.Map{
		"address": address.String(),
		"hash":    transaction.Hash(),
	})
}
```

It isn't any difference from the previous controller function so no explanation is needed.

Same applies for  the blockchain package right under the `DeployUSDC` method. Add a new method `DeployFactory`, which calls the DeployFactory in the Factory.go file. The `DeployFactory` also deploys the `Escrow` and `CustodianWalletLogic` contracts as you will see futher down the line.

```go
func (clientCon *ClientConnection) DeployFactory() (common.Address, *types.Transaction) {
	contract, trx, _, err := ethereum.DeployFactory(clientCon.trxOpts, clientCon.Client)

	if err != nil {
		log.Fatalln("Could not deploy Factory contract: ", err)
	}

	return contract, trx
}
```

Now in Postman if you make a POST request to http://127.0.0.1:3000/api/v1/deploy/factory the Factory contract will be deployed.

![Deploy Contract with Postman](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+11.06.11+PM.png)

Remember, I mentioned that Deploying the Factory Contract also deploys Escrow & WalletLogic Contracts. You can confirm that below by looking at the Factory Contract contrustor.

![Factory Constructor](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-11+at+11.35.37+PM.png)

### Get CustodianWalletLogic Address

To use the Factory contract, we will create a `factory.go` in the blockchain package. And just like the `blockchain.go` file where we defined a Singlton variable to hold the connection Instance, we will aslo define a variable  `FactoryInstance` to hold an instance of the deployed Factory contract.

Then a `NewFactory` method to use the contract address to get an instance.

```go
var FactoryInstance *Factory

type Factory struct {
	Address  common.Address
	Instance *ethereum.Factory
}

func (clientCon ClientConnection) NewFactory(address string) {
	FactoryInstance = new(Factory)

	contractAddress := common.HexToAddress(address)

	FactoryInstance.Address = contractAddress

	instance, err := ethereum.NewFactory(contractAddress, clientCon.Client)
	if err != nil {
		log.Fatalln("Cannot get Factory contract at address ", address, " due to: ", err)
	}

	FactoryInstance.Instance = instance
}
```

And Finnally the `GetLogicAddress` method.

```go
func (clientCon ClientConnection) GetLogicAddress() common.Address {

	address, err := FactoryInstance.Instance.CustodianWalletLogic(clientCon.callOpts)

	if err != nil {
		log.Fatalln("Cannot make call to Factory at ", FactoryInstance.Address, "due to: ", err)
	}

	return address
}
```

putting it all together `pkg/blockchain/factory.go` should look completely like this.

```go
package blockchain

import (
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/ethereum"
	"github.com/ethereum/go-ethereum/common"
	"log"
)

var FactoryInstance *Factory

type Factory struct {
	Address  common.Address
	Instance *ethereum.Factory
}

func (clientCon ClientConnection) NewFactory(address string) {
	FactoryInstance = new(Factory)

	contractAddress := common.HexToAddress(address)

	FactoryInstance.Address = contractAddress

	instance, err := ethereum.NewFactory(contractAddress, clientCon.Client)
	if err != nil {
		log.Fatalln("Cannot get Factory contract at address ", address, " due to: ", err)
	}

	FactoryInstance.Instance = instance
}

// GetLogicAddress 0x505A066E89Be22D3e56f16e1666de31f9328572e
func (clientCon ClientConnection) GetLogicAddress() common.Address {

	address, err := FactoryInstance.Instance.CustodianWalletLogic(clientCon.callOpts)

	if err != nil {
		log.Fatalln("Cannot make call to Factory at ", FactoryInstance.Address, "due to: ", err)
	}

	return address
}
```

In the `api_router.go` file, add the route to call  the new controller method you just added.

```go
api.Get("/wallet-logic-address", controllers.GetCustodianWalletLogicAddress)
```

The `GetCustodianWalletLogicAddress` controller method is not yet defined. Lets separate, factory logics from main controller and deployer controller, for these reasons create a new controller `factory_controller.go` and paste this inside.

```go
package controllers

import (
	"github.com/alofeoluwafemi/go-ethereum-api/pkg/blockchain"
	"github.com/gofiber/fiber/v2"
)

func GetCustodianWalletLogicAddress(c *fiber.Ctx) error {
	conn := blockchain.CurrentConnection

	address := conn.GetLogicAddress()

	return c.Status(fiber.StatusOK).JSON(fiber.Map{
		"address": address.String(),
	})
}
```
Finally in `factory.go` add the method called from the controller.

```go
func (clientCon ClientConnection) GetLogicAddress() common.Address {

	address, err := getFactory().CustodianWalletLogic(clientCon.callOpts)

	if err != nil {
		log.Fatalln("Cannot make call to Factory at ", FactoryInstance.Address, "due to: ", err)
	}

	return address
}
```

Using Postman, make a GET request to http://127.0.0.1:3000/api/v1/wallet-logic-address.

![Wallet Logic Address](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-12+at+1.40.37+AM.png)

Add a new similarl route, to get the Escrow contract address deployed by the Factory.

```go
api.Get("/escrow-address", controllers.GetEscrowAddress)
```

Create the `GetEscrowAddress` function in the factory controller, almost same thing as the GetLogicAddress controller.

```go
func GetEscrowAddress(c *fiber.Ctx) error {  
   conn := blockchain.CurrentConnection  
  
  address := conn.GetEscrowAddress()  
  
   return c.Status(fiber.StatusOK).JSON(fiber.Map{  
      "address": address.String(),  
   })  
}
```

Finally add the `GetEscrowAddress` method to the `factory.go` 

```go
func (clientCon ClientConnection) GetEscrowAddress() common.Address {  
  
   address, err := getFactory().EscrowContractAddress(clientCon.callOpts)  
  
   if err != nil {  
      log.Fatalln("Cannot make call to Factory at ", FactoryInstance.Address, "due to: ", err)  
   }  
  
   return address  
}
```
Using Postman, make a GET request to http://127.0.0.1:3000/api/v1/escrow-address.

![Get Escrow Address](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-09-12+at+2.19.47+AM.png)

So far we have be able to do the following:
- Setup Go Fiber framework and structure it for building out API
- Setup API end points to deploy both Factory & USDC Contract
- Setup API endpoint to get our deployed Escrow and Wallet Logic Address deployed when we deployed the Factory Contract. We are directly reading this information from the deployed Factory Contract

In the concluding article we are going to finish up the other endpoint that interacts with the Escrow contract to perform actions related to orders.

### What's Next ?
[Read Part 2 of Building a Complete API to Interact with an P2P Escrow Smart Contract using go-ethereum Client & GoFiber Framework](https://)
