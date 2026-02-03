# How to shuffle an array in Solidity

*Originally published on Mar 30, 2023 [here](https://medium.com/@jannden/how-to-shuffle-an-array-in-solidity-fe08b028287d).*

---

This is a simple function to help you shuffle arrays in Solidity.

The function takes two parameters: an array of uints and a random number.

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.2 <0.9.0;

contract Shuffler {
  function shuffleArray(uint256[] memory _array, uint256 _randomNumber) public pure returns (uint256[] memory) {
    require(_array.length > 0, "Array is empty");
    for (uint256 i = 0; i < _array.length; i++) {
      uint256 n = i + (_randomNumber % (_array.length - i));
      if(i != n) {
        uint256 temp = _array[n];
        _array[n] = _array[i];
        _array[i] = temp;
      }
    }
    return _array;
  }
}
```

It works in a few simple steps:

1.  checks whether the array is not empty,
2.  it runs the `for` loop iterating through each element of the array,
3.  a new position for the element is generated,
4.  if the new position is different from the old position, a switch is made between the two array elements.

## Conclusion

This short article showed how to shuffle an array in Solidity.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
