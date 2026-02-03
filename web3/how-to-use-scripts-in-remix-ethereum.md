# How to use scripts in Remix.Ethereum.org

*Originally published on Apr 6, 2023 here.*

---

Remix is a great IDE for smart contract development. Most people use it for its support of the Solidity language. But it has much more potential.

It features full support of JavaScript and a ready-to-go playground with pre-installed Web3.js and Ethers libraries.

That's a powerful environment.

Many times it's faster and easier to use Remix than Truffle or Hardhat for deploying simple contracts and interacting with them --- especially for learning purposes.

I will show you a quick way to use the full potential of JavaScript in Remix.

### Default workspace in Remix

Open [remix.ethereum.org](https://remix.ethereum.org/).

Click on the File Explorer icon in the top left corner:

![](https://miro.medium.com/v2/resize:fit:54/1*V3cLamqOF6thGomQ9AraIw.png)

The default workspace (as of the current version 0.31.2) contains 3 directories:

1.  `./contracts` Holds three contracts with increasing levels of complexity.
2.  `./scripts` Contains four typescript files to deploy a contract. It is explained below.
3.  `./tests` Contains one Solidity test file for 'Ballot' contract & one JS test file for 'Storage' contract.

## Smart contracts in Remix

You can add your own contract to the `./contracts` directory, or you can use one of the premade contracts.

I will use the premade `Storage` contract available in the file `./contracts/1_Storage.sol` for the purpose of this article.

We should first compile the contract. Click on `1_Storage.sol` in the file explorer:

![](https://miro.medium.com/v2/resize:fit:574/1*Z-tDmfN9VROnTJy584vMIg.png)

Now click on the Solidity Compiler icon in the left menu:

![](https://miro.medium.com/v2/resize:fit:56/1*ufmEhu-zcoTtsxwNmiZLwQ.png)

And compile the contract by clicking on the blue button:

![](https://miro.medium.com/v2/resize:fit:444/1*clrm95t0EJodLR8gsJHChg.png)

To deploy the contract, go back to the File Explorer page:

![](https://miro.medium.com/v2/resize:fit:54/1*V3cLamqOF6thGomQ9AraIw.png)

Right click on `./scripts/deploy_with_ethers.ts` and choose Run:

![](https://miro.medium.com/v2/resize:fit:875/1*xNNqXRTflLy5NDIIG9Yukg.png)

The contract is now deployed. Notice that the console log in the bottom part of Remix shows the deployed contract's new address:

![](https://miro.medium.com/v2/resize:fit:434/1*JC4Jn73e4VMtQjghPIt1dw.png)

If you wanted to deploy a different contract (which would have to be compiled first as well), you would change the name 'Storage' on the 9th line in the script to whatever name your contract has:

![](https://miro.medium.com/v2/resize:fit:824/1*yhU7Y7NMuT74VRDLITetTQ.png)

## Interacting with a deployed contract in Remix

Once your contract is deployed, you can connect to it and interact with it by creating a new script, for example:

```javascript
import { ethers } from 'ethers'

// Update these two variables as per your needs
const contractName = "YOUR_CONTRACT_NAME"; // The name of your contract (make sure the contract has been compiled in Remix)
const contractAddress = "YOUR_CONTRACT_ADDRESS"; // Assuming your contract has already been deployed, update the address

// To run the script in Remix, right click on file name in Remix file explorer and click 'Run'.
(async () => {
  try {
    console.log("Setting things up...");
    const artifactsPath = `browser/contracts/artifacts/${contractName}.json`;
    const metadata = JSON.parse(await remix.call("fileManager", "getFile", artifactsPath));

    const signer = (new ethers.providers.Web3Provider(web3Provider)).getSigner()

    const contract = new ethers.Contract(contractAddress, metadata.abi, signer)

    // This example interacts with the default contract provided by Remix called Storage
    // The Storage contract has two methods - store(value) and retrieve()
    console.log("Storing value...");
    await contract.connect(signer).store(100);

    console.log("Retrieving value...");
    const value = await contract.connect(signer).retrieve();

    console.log(`Retrieved value is: ${value}`);

  } catch (e) {
    console.log(e.message);
  }
})();
```

## Conclusion

This short article showed how to use scripts in Remix.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
