
# NFT Minter

```
// nftMinter.js

const Web3 = require('web3');

class NFTMinter {
    constructor(rpcURL, contractABI, contractAddress) {
        if (!rpcURL) throw new Error("RPC URL is required");
        this.web3 = new Web3(new Web3.providers.HttpProvider(rpcURL));
        this.contract = new this.web3.eth.Contract(contractABI, contractAddress);
    }

    async mintNFT(fromAddress, toAddress, tokenId, privateKey) {
        const txData = this.contract.methods.mint(toAddress, tokenId).encodeABI();

        const tx = {
            from: fromAddress,
            to: this.contract.options.address,
            data: txData,
            gas: await this.web3.eth.estimateGas({from: fromAddress, to: this.contract.options.address, data: txData}),
            gasPrice: await this.web3.eth.getGasPrice()
        };

        const signedTx = await this.web3.eth.accounts.signTransaction(tx, privateKey);
        return this.web3.eth.sendSignedTransaction(signedTx.raw || signedTx.rawTransaction);
    }
}

module.exports = NFTMinter;
```

## Usage
```
const NFTMinter = require('./nftMinter');
const YOUR_CONTRACT_ABI = []; // Your NFT contract ABI here
const YOUR_CONTRACT_ADDRESS = "0x..."; // Your NFT contract address here

const minter = new NFTMinter("YOUR_RPC_URL", YOUR_CONTRACT_ABI, YOUR_CONTRACT_ADDRESS);

// Example usage
async function main() {
    const result = await minter.mintNFT('FROM_ADDRESS', 'TO_ADDRESS', 'TOKEN_ID', 'PRIVATE_KEY');
    console.log("Transaction Hash:", result.transactionHash);
}

main().catch(console.error);

```
# Get NFT Owner
```
// Add these functions inside the NFTMinter class:

async getOwnerOf(tokenId) {
    try {
        return await this.contract.methods.ownerOf(tokenId).call();
    } catch (error) {
        console.error("Error fetching owner:", error);
        return null;
    }
}
```
# Get Metadata
```
async getMetadataURI(tokenId) {
    try {
        return await this.contract.methods.tokenURI(tokenId).call();
    } catch (error) {
        console.error("Error fetching metadata URI:", error);
        return null;
    }
}
```
## Usage
```
const NFTMinter = require('./nftMinter');
const YOUR_CONTRACT_ABI = []; // Your NFT contract ABI here
const YOUR_CONTRACT_ADDRESS = "0x..."; // Your NFT contract address here

const minter = new NFTMinter("YOUR_RPC_URL", YOUR_CONTRACT_ABI, YOUR_CONTRACT_ADDRESS);

// Example usage
async function main() {
    const tokenId = "YOUR_DESIRED_TOKEN_ID";

    // 1. Mint an NFT
    // NOTE: NEVER use private keys directly in code. This is just for demonstration purposes.
    const result = await minter.mintNFT('FROM_ADDRESS', 'TO_ADDRESS', tokenId, 'PRIVATE_KEY');
    console.log("Transaction Hash:", result.transactionHash);

    // 2. Get owner of the NFT
    const owner = await minter.getOwnerOf(tokenId);
    console.log(`Owner of token ${tokenId} is: ${owner}`);

    // 3. Get metadata URI of the NFT
    const metadataURI = await minter.getMetadataURI(tokenId);
    console.log(`Metadata URI of token ${tokenId} is: ${metadataURI}`);
}

main().catch(console.error);
```
