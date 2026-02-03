# Create an Auction smart contract in Solidity

*Originally published on Mar 15, 2023 [here](https://medium.com/@jannden/create-a-blind-auction-smart-contract-in-solidity-2d3ab231784f).*

---

This tutorial will teach you core concepts of Solidity on a practical example.

We will create a smart contract for an Auction with the following features:

-   there is a minimum bid amount,
-   anyone can become a bidder by sending their bid higher than the minimum,
-   the manager can stop the auction, which makes the bidder with the highest amount automatically the winner of the auction,
-   the contract keeps track of all past winners.

*The finished code can be *[*found here*](https://github.com/jannden/solidity-contracts/blob/main/Auction.sol)*.*

### Setting the variables

Let's first initialize the variables. We will need to keep track of the following:

-   who is the `manager` --- the person who deploys the contract,
-   array of `pastWinners`,
-   array of `bidders` of the current auction,
-   mapping `bidAmountsToBidders` to keep track of who bid how much,
-   `minPriceToEnter`

The constructor of the function automatically sets the `manager` to be the `msg.sender`, as well as a hard-coded value for the `minPriceToEnter`.

The code looks like this so far:

```solidity
// SPDX-License-Identifier: MITpragma solidity ^0.8.7;contract Auction {
  address manager;
  address[] pastWinners;
  address[] bidders;
  mapping(address => uint256) bidAmountsToBidders;
  uint256 public minPriceToEnter; constructor() {
    manager = msg.sender;
    minPriceToEnter = 0.01 ether;
  } // ... The rest of the code will go here}
```

### Bidding

We will create a function that lets bidders enter the auction.

```solidity
function enter() public payable {
  require(
    msg.value >= minPriceToEnter,
    "You haven't contributed enough Ether."
  );
  bidders.push(msg.sender);
  bidAmountsToBidders[msg.sender] = msg.value;
}
```

### Modifier for Managers Only

A modifier will help us give access to specific functions only to managers.

```solidity
modifier managersOnly() {
  require(msg.sender == manager, "Only managers can perform this action.");
  _; // this will get replaced by the function which uses this modifier
}
```

### Picking a Winner

Managers can pick a winner with the highest bid. This function will also return funds to losing bidders and reset current bids.

```solidity
function pickWinner() public managersOnly {
  require(bidders.length > 0, "No bidders entered.");
  uint256 highestBid;
  address currentWinner;// Select the highest bidder
  for(uint i = 0; i < bidders.length; i++) {
    if(bidAmountsToBidders[bidders[i]] > highestBid) {
      currentWinner = bidders[i];
      highestBid = bidAmountsToBidders[currentWinner];
    }
  }// Return funds to losing bidders and reset current bids
  for(uint i = 0; i < bidders.length; i++) {
    if(bidders[i] != currentWinner) {
      payable(bidders[i]).transfer(bidAmountsToBidders[bidders[i]]);
    }
    bidAmountsToBidders[bidders[i]] = 0;
  }// Save current winner
  pastWinners.push(currentWinner);// Reset bidders
  delete bidders;
}
```

### List of past winners

And for the sake of completeness, we will expose a function to see all past winners.

```solidity
function getPastWinners() public view returns (address[] memory) {
  return pastWinners;
}
```

### Conclusion

We created a full contract for the Auction project. I hope you have learned some interesting concepts of programming in Solidity along the way. You will find [the whole finished code here](https://github.com/jannden/solidity-contracts/blob/main/Auction.sol).

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
