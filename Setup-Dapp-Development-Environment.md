


## Overview
While building Dapps especially for a small project, it common place for developers to have seperate project folders for the frontend, backend and smart contract. Similarly some tools that I consider time saving and bug-saving (assuming that is a word) that developers can take advantage of such as **sollint**, **prettier**, **husky**, **slither**..., the list continues, are usually not used. 

## Disclaimer
This is based on my personal opinion on how I've attempted to boostrap some persoanl Dapps, I've worked on and should not be considered as general opinion. However I accept suggestions and critics.  

## What Will I Learn
- How to structure your Dapp project
- Configure Vscode with pretttier and solhint
- Run Slither to catch common vulnerabilities in your contracts
- Check  that you don't commit Private keys to Github
- Configure husky for pre-push git-hook
	- Run linter and formatter 
	- Run vunurability checks using slither
	- Check your smart contract test are passing

You will get the whole idea of having your own custom preset. You can star my [preset structure here on github](https://github.com/alofeoluwafemi/dapp-development-preset) for your reference.

## Requirement
I would expect that you have `node` installed. I recommend using `nvm` to manage your `node` versions. Also this tutorial assumes you are using the Vscode editor.

## Difficulty
- Beginner :white_check_mark:
- Intermidiate :white_check_mark:
- Advance :white_check_mark:

I created a Telegram group for anyone learning web3, you can using https://t.me/+Og9C3Z23lpc5MWRk. You will get access to free web3 resource, 1-on-1 and group live coding session and many other web3 resources.


## Step 1

From your terminal, navigate to your project directory or create a new one as mine, initialize it as a npm project.

```bash
mkdir dapp-setup-preset
cd dapp-setup-preset && code . // Open folder in Vscode
npm init
git init
```

## Step 2

Create the following files & folders:
- src
	- frontend
	- backend
	- smart-contract
		-  .solhintignore
		- solhint.json
- .vscode
	- settings.json
	- extensions.json
- .gitignore
- .prettierignore
- prettierrc.json

## Step 3

Installing dependecies & plugins. 

**Install the Prettier Plugin for Vscode**
![Install Prettier Plugin for Vscode](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-08-30+at+8.40.43+PM.png)

**Install solhint, prettier & configure both to work together**
Run the commond below from the project root directory
```
npm install --save-dev solhint prettier prettier-plugin-solidity husky solhint-plugin-prettier
```

Open `.solhint.json` and paste below defualt configuration.

```json
{
"extends":  "solhint:recommended",
"plugins":  [],
"rules": {
	"prettier/prettier":  "error",
	"compiler-version":  ["error",  "^0.8.9"],
	"func-param-name-mixedcase":  "error",
	"modifier-name-mixedcase":  "error",
	"private-vars-leading-underscore":  "error",
	"avoid-throw":  false,
	"avoid-suicide":  "error",
	"avoid-sha3":  "warn"	
  }
}
``` 

For more configuration options check the officel [solhint docs](https://github.com/protofire/solhint#configuration).

In your `.solhintignore` file add

```
cache
artifacts
node_modules
test
scripts
types
```
Open `prttierrc.json` file and paste the below configuration in.

```json
{
    "overrides": [
        {
            "files": "*.js",
            "options": {
                "trailingComma": "none",
                "tabWidth": 4,
                "semi": true,
                "singleQuote": true
            }
        },
        {
            "files": "*.json",
            "options": {
                "trailingComma": "none",
                "tabWidth": 4,
                "semi": true,
                "singleQuote": true
            }
        },
        {
            "files": "*.sol",
            "options": {
                "printWidth": 80,
                "tabWidth": 4,
                "useTabs": false,
                "singleQuote": false,
                "bracketSpacing": false,
                "explicitTypes": "always"
            }
        }
    ]
}


```

You can read the [prettier documentation](https://prettier.io/docs/en/options.html) for further configuration.

Finally lets us configure solhint to work nicely with prettier. [Prettier-solidity](https://github.com/prettier-solidity/prettier-plugin-solidity) is a Prettier for solidity files that works hand-in-hand with Solhint. It helps to automatically fix many of the errors that Solhint finds.

Update `.solhint.json` to include `prettier` in the plugins option. Now your configuration should look something like this.

```json
{
  "extends": "solhint:recommended",
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error",
    "compiler-version": ["error", "^0.8.9"],
    "func-param-name-mixedcase": "error",
    "modifier-name-mixedcase": "error",
    "private-vars-leading-underscore": "error",
    "avoid-throw": false,
      "avoid-suicide": "error",
      "avoid-sha3": "warn"
  }
}
```
## Step 4
Setup hardhat in the smart-contract directory.  Navigate to `src/smart-contracts` and run the below command.

⚠️ You should be running `npm` version 7.0.0+

```bash
npm init
npm install --save-dev hardhat
npx hardhat
```
⚠️ Click enter for all prompts, while running `npx hardhat`

## Step 5

Update your `package.json`, in the root folder , with some scripts to show you how you can run command in any of your project workspace under the `src` folder directly from the project root.

```json
{
    "name": "project-setup-demo",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "hardhat:lint": "yarn workspace smart-contract sollint:solidity",
        "hardhat:prettier": "yarn workspace smart-contract prettier:solidity",
        "prepare": "husky install",
        "hardhat:delpoy-local": "yarn workspace smart-contract deploy:local",
        "hardhat:test": "yarn workspace smart-contract test"
    },
    "author": "",
    "license": "ISC",
    "devDependencies": {
        "husky": "^8.0.1",
        "prettier": "^2.7.1",
        "prettier-plugin-solidity": "^1.0.0-beta.24",
        "solhint": "^3.3.7",
        "solhint-plugin-prettier": "^0.0.5"
    },
    "private": true,
    "workspaces": {
        "packages": [
            "src/*"
        ]
    }
}
``` 

Nex update the `src/smart-contract/package.json` to match below.

```json
{
    "name": "smart-contract",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "npx hardhat test",
        "sollint:solidity": "../../node_modules/.bin/solhint -f table contracts/**/*.sol",
        "prettier:solidity": "../../node_modules/.bin/prettier --write contracts/**/*.sol"
    },
    "author": "",
    "license": "ISC",
    "devDependencies": {
        "@nomicfoundation/hardhat-toolbox": "^1.0.2",
        "hardhat": "^2.10.2"
    }
}
```

To test if this is work, run `npm run hardhat:lint` in the project root directory, you will see it runs successfully and present the following warnings to based on the `Lock.sol` file.

![npm run lint-sol](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-08-31+at+3.18.46+AM.png)

## Step 6
Vscode allows us to define a settings specfic to projects which I really find useful, mainly due to the fact that, you can have a consistent global settings with your team if you choose to do so.

So inside, `.vscode/settings.json` paste the following configuration code inside of it.

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": false,
  "prettier.useEditorConfig": false,
  "prettier.useTabs": false,
  "prettier.configPath": "prettierrc.json"
}
```

What the above configuration does is auto format your files watched by prettier when you use `cmd+S` or `ctrl+S`, paste a content into a `js` , `json` or `sol` file.

To also enforce some extensions, we need for solidity suggestions and tips using Vscode. Paste this in the `.vscode/extensions.json`.

```json
{
    "recommendations": [
        "JuanBlanco.solidity",
        "esbenp.prettier-vscode",
        "nomicfoundation.hardhat-solidity"
    ]
}
```

You can try this out by opening `src/smart-contract/Lock.sol` in your editor and control save by holding `cmd + S` for mac and `ctrl+S` for window, you would see how it auto formats and also corrects `uint` to `uint256`. That is solhint and prettier doing easy fixes for you.

![solhint+prettier](https://s3.amazonaws.com/alofe.oluwafemi/ezgif.com-gif-maker+%281%29.gif)

## Step 7

Setup a pre-commit hook using husky, you already have husky installed if you followed this tutorial correctly. From the project root terminal run:

```bash
npm set-script prepare "husky install"
npm run prepare
npx husky add .husky/pre-commit "npm run hardhat:lint"
npx husky add .husky/pre-commit "npm run hardhat:prettier"
npx husky add .husky/pre-commit "npm run hardhat:prettier" 
```



## Step 8

Now lets, test our pre-commit hook. First don't for get to add `node_modules` to your `.gitignore` as we don't to commit it.

Now run

```bash
git add .
git commit -m"Preset Boostraping"
```

![Pre commit hook](https://s3.amazonaws.com/alofe.oluwafemi/ezgif.com-gif-maker+%282%29.gif)

To see how it fails, add below code on line 11 of `Lock.sol`. This violate the rule of not adding underscore behind private variable.

```javascript
address private isprivate;
```

![Pre-commit hook fails](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-08-31+at+2.18.22+PM.png)

## Step 9

Install slither for solidity source analysis. To install slither globally on your machine use `pip3 install slither-analyzer`.

Add the command to our pre-hook. 

```
npx husky add .husky/pre-commit "slither src/smart-contract/*/*.sol"
```

The `.husky/pre-commit` file should look something like this

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run hardhat:lint
npm run hardhat:prettier
npm run hardhat:test
slither src/smart-contract/*/*.sol
```

When you perform a new commit now, you will not only see warning from linter, you will also see warning from slither based on the following [detector documentation](https://github.com/crytic/slither/wiki/Detector-Documentation).

![Detector Slither](https://s3.amazonaws.com/alofe.oluwafemi/Screen+Shot+2022-08-31+at+2.50.38+PM.png)

## Step 10 (Final Step)

Let ensure you don't commit your Private Key to Gihub ever. Run this command to create and add the right permission to our simple bash script in the project root directory.

```bash
touch secret-check.sh
chmod +x secret-check.sh
```

Copy and paste this simple shell script inside `secret-check.sh`

```bash
#!/usr/bin/env sh

OUTPUT="$(grep "(0x)?[a-fA-F0-9]{64}" src -r -E -o --exclude-dir=artifacts)"
RED='\033[0;31m'          # Red
GREEN='\033[0;32m'        # Green

if [ -n "$OUTPUT" ]; then
    echo "${GREEN} You have a Private Key Committed"
    echo -e ${RED} $OUTPUT
    exit 1
fi
```

In my opinion it makes sense to check for private key before running linter and slither. Fail fast!

With that in mind, edit `.husky/pre-commit` to run the check as the first thing.

```bash
#!/usr/bin/env sh

OUTPUT="$(grep "(0x)?[a-fA-F0-9]{64}" src -r -E -o --exclude-dir=artifacts)"
RED='\033[0;31m'          # Red
GREEN='\033[0;32m'        # Green

if [ -n "$OUTPUT" ]; then
    echo "${GREEN} You have a Private Key Committed"
    echo -e ${RED} $OUTPUT
    exit 1
fi
```

Now, proceed to commit your files and see how this run. To confirm this work, add to your `Lock.js` test file

```js
PRIVATE_KEY="0x596F752063616E27742077697468647261772079657400000000000000000000"
```

The commit process will exit with a status 1.

Thank you for sticking this far with me, If you enjoyed this article you can support me by Clapping for this post and subscribing to my [Youtube Channel](https://www.youtube.com/channel/UCO3mWoCZ_iqRPRvUeg9oG2A).
