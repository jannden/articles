# Web3 boilerplate for Smart Contracts in HTML and JS

*Originally published on Apr 9, 2022 [here](https://medium.com/@jannden/web3-boilerplate-for-smart-contracts-in-html-and-js-c50c1d7e292e).*

---

Sometimes, all you need is a simple boilerplate to communicate with your Smart Contract. So let's create a Web3.js connection to blockchain using just HTML and JavaScript.

You can see the completed boilerplate on [GitHub](https://github.com/jannden/boilerplate-web3js-html5).

### Import Web3.js to HTML

First things first, we have to import Web3.js to our project. For the sake of simplicity, we are not using any bundlers here, just a plain HTML project.

So create a new file `index.html` and include Web3.js the simple way straight into the `<body>` of our HTML page:

```html
<script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
```

We will also include our own JS file, which we will create later:

```html
<script src="./index.js"></script>
```

Lastly, let's add a button and a place to display information in order to make the interaction with the page easier:

```html
<button class="start">Start</button>
<ul class="info"></ul>
```

### Create JS for all the magic

Now we will create a JavaScript file `index.js`, which we will use to initialize Web3, connect to a wallet like MetaMask, and communicate with a Smart Contract deployed to Blockchain.

First, we will add an event listener checking whether the html loaded. It will be a wrapper for all of our JS logic. Since all blockchain action is asyncronious, we will use an async function as the second parameter:

```javascript
window.addEventListener('load', async () => {
  ...
}
```

### Connecting to Ethereum and MetaMask

The MetaMask wallet injects an object `ethereum` to every webpage we visit. We can check whether this object exists and so determine, whether MetaMask is installed.

```javascript
window.addEventListener('load', async () => {
  if (typeof ethereum === 'undefined') {
    console.log('Metamask not connected.');
    return;
  }
  // ... continue with initializing Web3
}
```

If MetaMask is available, we can proceed with initializing Web3. We can also get the address of user's wallet in MetaMask:

```javascript
window.addEventListener('load', async () => {
  // ...
  web3 = new Web3(window.ethereum); // initialize Web3
  const accounts = await ethereum.request({ method: "eth_requestAccounts" });
  const account = accounts[0];
}
```

MetaMask gives an array of all user's addresses, that's why we specified we want the first address that is stored in `accounts[0]`.

### Communicating with a Smart Contract

Next step is to connect to your Smart Contract that is deployed to the Blockchain. You will need the contract's address and ABI.

Establishing a connection with the Smart Contract is then just one line of code:

```javascript
window.addEventListener('load', async () => {
  // ...
  const contract = new web3.eth.Contract(contractABI, contractAddress);
}
```

Let's say the contract is written in the ERC20 standard, so it represents a fungible token (sort of a crypto-currency). As we have already established connection to it, let's see, how we can interact with it now.

### The call() method

Our ERC20 contract would have a `balanceOf()` method. Since such method is `view` only and doesn't cost gas, we can use `call()` to invoke it.

```javascript
window.addEventListener('load', async () => {
  // ...
  try {
    const balance = await contract.methods.balanceOf(account).call();
    console.log(`Account ${readableAddress(account)} has ${web3.utils.fromWei(balance)} tokens.`);
  } catch (error) {
    console.log(`Error: ${error.message}`);
  }
}
```

### The send() method

If we want to invoke a method that costs gas, such as `transfer()`. A method costs gas (and thus real money if we are not on testnet) if it needs to calculate something or save some data to the blockchain. In such a case, we cannot use `call()` like in the example above. Instead, we have to use `send()` with the parameter `from`. The account specified as `from` will pay the gas fee.

```javascript
window.addEventListener('load', async () => {
  // ...
  try {
    const result = await contract.methods.transfer(to, amount).send({ from: account });
    console.log(`Result: ${result.status}`);
  } catch (error) {
    console.log(`Error: ${error.message}`);
  }
}
```

### Listening to events

We can also listen to events happening on the blockchain. Coming back to the ERC20 standard, there must be a `transfer()` method which evokes an event. We can subscribe to this event like this:

```javascript
contract.events.Transfer(from, to, value)
  .on("data", console.log)
  .on("error", console.error);
```

The code above could be placed right after we establish the connection with the smart contract. You can replace the console logs with whatever functions suit your needs.

Events inform our front-end that something happened on the blockchain that we might be interested in. In this case somebody transfered tokens, so we may want to display that information to the user.

### Conclusion

You have learned how to connect to the blockchain using the Web3.js library, link the webpage to user's MetaMask and communicate with a deployed Smart Contract.

You now know when to use the `call()` and `send()` methods, as well as how to use blockchain events to update your front-end.

Here is the GitHub repo with the complete [boilerplate for Web3.js in HTML](https://github.com/jannden/boilerplate-web3js-html5).

If you found this article helpful, please click the clap 👏button, you can also follow me on Medium, and feel free to comment! I'd be happy to help :)
