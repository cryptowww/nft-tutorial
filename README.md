# OpenSea NFT tutorial

## Begin the Project

1. new project

> mkdir nft-tutorial & cd nft-tutorial

2. init the project

> npm init

3. install the dependencies

> npm install --save-dev hardhat
...

> npm install --save-dev @nomiclabs/hardhat-ethers
...

> npm install --save-dev @nomiclabs/hardhat-etherscan
...

> npm install @openzeppelin/contracts
...

> npm install dotenv --save
...

> npm install --save-dev ethers@^5.0.0
...

> npm install --save-dev node-fetch@2

4. initialize the Hardhat

> npx hardhat

chose the :

Create an empty hardhat.config.js

5. directory structure

> mkdir contracts scripts


6. write the smart contract

> cd contracts

> vim NFT.sol

then copy below code to NFT.sol

```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract NFT is ERC721 {
    using Counters for Counters.Counter;
    Counters.Counter private currentTokenId;
    
    constructor() ERC721("NFTTutorial", "NFT") {}
    
    function mintTo(address recipient)
        public
        returns (uint256)
    {
        currentTokenId.increment();
        uint256 newItemId = currentTokenId.current();
        _safeMint(recipient, newItemId);
        return newItemId;
    }
}

```

7. set the env

> cd ../

ensure you are in the main folder.

> vim .env

the content:

```shell
ALCHEMY_KEY = "alchemy-api-key"
ACCOUNT_PRIVATE_KEY = "private-key"

```

8. compile the contract

> npx hardhat compile

9. deploy the contract

> cd scripts
> vim deploy.js

paste the code :

```solidity
async function main() {

    // Get our account (as deployer) to verify that a minimum wallet balance is available
    const [deployer] = await ethers.getSigners();

    console.log(`Deploying contracts with the account: ${deployer.address}`);
    console.log(`Account balance: ${(await deployer.getBalance()).toString()}`);

    // Fetch the compiled contract using ethers.js
    const NFT = await ethers.getContractFactory("NFT");
    // calling deploy() will return an async Promise that we can await on 
    const nft = await NFT.deploy();

    console.log(`Contract deployed to address: ${nft.address}`);
}

main()
.then(() => process.exit(0))
.catch((error) => {
    console.error(error);
    process.exit(1);
});

```

> npx hardhat run srcipts/deploy.js --network rinkeby


10. Minting from your new NFT contract using Hardhat

> cd scripts
> rename deploy.js deploy_v1.js

> vim deploy.js

then paste this code:

```solidity
const { task } = require("hardhat/config");
const { getAccount } = require("./helpers");


task("check-balance", "Prints out the balance of your account").setAction(async function (taskArguments, hre) {
    const account = getAccount();
    console.log(`Account balance for ${account.address}: ${await account.getBalance()}`);
});

task("deploy", "Deploys the NFT.sol contract").setAction(async function (taskArguments, hre) {
    const nftContractFactory = await hre.ethers.getContractFactory("NFT", getAccount());
    const nft = await nftContractFactory.deploy();
    console.log(`Contract deployed to address: ${nft.address}`);
});
```


create a new helpers.js file like this:

```solidity

const { ethers } = require("ethers");


// Helper method for fetching environment variables from .env
function getEnvVariable(key, defaultValue) {
    if (process.env[key]) {
        return process.env[key];
    }
    if (!defaultValue) {
        throw `${key} is not defined and no default value was provided`;
    }
    return defaultValue;
}

// Helper method for fetching a connection provider to the Ethereum network
function getProvider() {
    return ethers.getDefaultProvider(getEnvVariable("NETWORK", "rinkeby"), {
        alchemy: getEnvVariable("ALCHEMY_KEY"),
    });
}

// Helper method for fetching a wallet account using an environment variable for the PK
function getAccount() {
    return new ethers.Wallet(getEnvVariable("ACCOUNT_PRIVATE_KEY"), getProvider());
}

module.exports = {
    getEnvVariable,
    getProvider,
    getAccount,
}

```

modify the hardhat.config.js :

```solidity
...

require("./scripts/deploy.js");
...

```

now ,you can see the task throw the command:

> npx hardhat

11. Adding a minting task

create a new file mint.js in scripts folder:

```solidity
const { task } = require("hardhat/config");
const { getContract } = require("./helpers");

task("mint", "Mints from the NFT contract")
.addParam("address", "The address to receive a token")
.setAction(async function (taskArguments, hre) {
    const contract = await getContract("NFT", hre);
    const transactionResponse = await contract.mintTo(taskArguments.address, {
        gasLimit: 500_000,
    });
    console.log(`Transaction Hash: ${transactionResponse.hash}`);
});

```

12. update the .env file


```shell
...

NETWORK="rinkeby"
NFT_CONTRACT_ADDRESS="YOUR_CONTRACT_ADDRESS"
```

13. update the helpers.js

```solidity
const { ethers } = require("ethers");
const { getContractAt } = require("@nomiclabs/hardhat-ethers/internal/helpers");


// Helper method for fetching environment variables from .env
function getEnvVariable(key, defaultValue) {
    if (process.env[key]) {
        return process.env[key];
    }
    if (!defaultValue) {
        throw `${key} is not defined and no default value was provided`;
    }
    return defaultValue;
}

// Helper method for fetching a connection provider to the Ethereum network
function getProvider() {
    return ethers.getDefaultProvider(getEnvVariable("NETWORK", "rinkeby"), {
        alchemy: getEnvVariable("ALCHEMY_KEY"),
    });
}

// Helper method for fetching a wallet account using an environment variable for the PK
function getAccount() {
    return new ethers.Wallet(getEnvVariable("ACCOUNT_PRIVATE_KEY"), getProvider());
}

// Helper method for fetching a contract instance at a given address
function getContract(contractName, hre) {
    const account = getAccount();
    return getContractAt(hre, contractName, getEnvVariable("NFT_CONTRACT_ADDRESS"), account);
}

module.exports = {
    getEnvVariable,
    getProvider,
    getAccount,
    getContract,
}

```

14. update hardhat.config.js

```solidity
...

require("./scripts/mint.js");

...

```

15. Mint NFT to your wallet


> npx hardhat mint --address your_wallet_address


16. add nft to your wallet

open your metamask wallet, and switch to rinkeby network

click the "add collections"

address: your_contract_address
id: your_nft_id

---------------------------------------------------------------------


17. add metadata

17.1 make a new folder images

> mkdir images

then download 3 images ,1.png, 2.png, 3.png

17.2 install ipfs-car

> npm install ipfs-car

17.3 package images

> npx ipfs-car --pack images --output images.car

17.4 upload the images.car to ipfs

open (nft storage)[https://nft.storage/files/]

register and upload the images.car file.

17.5 prepare metadata

> mkdir metadata

and make 1 2 3 three files .

17.6 upload again

> npx ipfs-car --pack metadata --output metadata.car


17. add metadata

17.1 make a new folder images

> mkdir images

then download 3 images ,1.png, 2.png, 3.png

17.2 install ipfs-car

> npm install ipfs-car

17.3 package images

> npx ipfs-car --pack images --output images.car

17.4 upload the images.car to ipfs

open (nft storage)[https://nft.storage/files/]

register and upload the images.car file.

17.5 prepare metadata

> mkdir metadata

and make 1 2 3 three files .

17.6 upload again

> npx ipfs-car --pack metadata --output metadata.car

17.7 mint the NFT

> npx hardhat compile

> npx hardhat deploy

> copy the contract address and update .env

> npx hardhat set-base-token-uri --base-url "https://bafybeigyyedzxsnneov5qix5ay7g6i5risch3auef2sc4dddan5rnld53m.ipfs.dweb.link/metadata/"

> npx hardhat mint --address <your_wallet_address>


18. opensea browser 

in browser:

https://testnets.opensea.io/assets/<contract_address>/<nft_id>

Please reference the link:

https://docs.opensea.io/docs/minting-from-your-new-contract-and-improvements
