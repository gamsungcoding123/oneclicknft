
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

# Usage
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
