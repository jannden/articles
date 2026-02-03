# Create a Presidential Election smart contract in Solidity

*Originally published on Mar 21, 2023 [here](https://medium.com/@jannden/create-a-presidential-election-smart-contract-in-solidity-ec8145330c17).*

---

This tutorial will teach you core concepts of Solidity on a practical example.

We will create a smart contract for a Presidential Election with the following features:

-   The contract owner can send a state result with information on how many votes each candidate has won and the seats awarded for the particular state.
-   The seats will be accumulated after each state result is received.
-   Anyone can check which candidate is in the lead at any point.
-   The contract owner can end the election.

*The complete code can be *[*found here*](https://github.com/jannden/solidity-contracts/blob/main/Election.sol)*.*

### Ownable contract

Our main contract will inherit from an `Ownable` contract to manage ownership and restrict access to certain functions (for example, the function for adding new books should be available only to the owner --- the librarian).

Here is a simple version of an Ownable contract:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

contract Ownable {
  address public owner;
  constructor() {
    owner = msg.sender;
  }
  modifier onlyOwner() {
    require(owner == msg.sender, "Not invoked by the owner");
    _;
  }
}
```

When the contract is initialized, the `constructor` saves the address of the original sender as the owner.

The functionality of a modifier `onlyOwner` will be used for functions with restricted access.

### Election contract: Setting the variables

Let's first initialize the variables. We will need to keep track of:

-   who is the `CANDIDATE_A` and `CANDIDATE_B`
-   whether the `electionEnded` already,
-   mappings of `seats` and `resultsSubmitted`,
-   struct for `StateResult`
-   and two events: `LogStateResult `and `LogElectionEnded`

The code looks like this so far:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;
pragma abicoder v2;

import "./Ownable.sol";

contract Election is Ownable {
  uint8 public constant CANDIDATE_A = 1;
  uint8 public constant CANDIDATE_B = 2;
  bool public electionEnded;

  mapping(uint8 => uint8) public seats;
  mapping(string => bool) public resultsSubmitted;

  struct StateResult {
    string name;
    uint256 votesCandidateA;
    uint256 votesCandidateB;
    uint8 stateSeats;
  }

  event LogStateResult(uint8 winner, uint8 stateSeats, string state);
  event LogElectionEnded(uint256 winner);

  // ... The rest of the code will go here

}
```

### Modifier onlyActiveElection

We want to limit the use of certain functions to active elections only, so we will create a modifier next:

```solidity
modifier onlyActiveElection() {
  require(!electionEnded, "The election has ended already");
  _;
}
```

### Submitting results

The function `submitStateResult` makes it possible to submit results to active elections by the contract owner. Notice that the function has two custom modifiers --- `onlyActiveElection` and `onlyOwner`.

```solidity
function submitStateResult(StateResult calldata result)
         public onlyActiveElection onlyOwner {
  require(result.stateSeats > 0, "States must have at least 1 seat");
  require(
    result.votesCandidateA != result.votesCandidateB,
    "There cannot be a tie"
  );
  require(
    !resultsSubmitted[result.name],
    "This state result was already submitted!"
  );

  uint8 winner;
  if (result.votesCandidateA > result.votesCandidateB) {
    winner = CANDIDATE_A;
  } else {
    winner = CANDIDATE_B;
  }

  seats[winner] += result.stateSeats;
  resultsSubmitted[result.name] = true;

  emit LogStateResult(winner, result.stateSeats, result.name);
}
```

### Current leader

At any time, we can check the current leader:

```solidity
function currentLeader() public view returns (uint8) {
  if (seats[CANDIDATE_A] > seats[CANDIDATE_B]) {
    return CANDIDATE_A;
  }
  if (seats[CANDIDATE_B] > seats[CANDIDATE_A]) {
    return CANDIDATE_B;
  }
  return 0;
}
```

### End the election

The owner of the contract is able to end the election:

```solidity
function endElection() public onlyActiveElection onlyOwner {
  electionEnded = true;
  emit LogElectionEnded(currentLeader());
}
```

### Conclusion

We created a smart contract for the Presidential Election project. I hope you have learned some interesting concepts of programming in Solidity along the way. You will find the [whole finished code here](https://github.com/jannden/solidity-contracts/blob/main/Election.sol).

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
