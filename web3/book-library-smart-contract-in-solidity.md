# Create a Book Library smart contract in Solidity

*Originally published on Mar 8, 2023 [here](https://medium.com/@jannden/create-a-book-library-smart-contract-in-solidity-232c4be0bdbe).*

---

This tutorial will teach you core concepts of Solidity on a practical example.

We will create a smart contract for a Book Library with the following features:

-   a librarian can add new books to the library, each with a different amount of copies,
-   users can see the available books,
-   users can borrow one copy per each available book,
-   users can return borrowed books,
-   anyone is able to see the history of all people who borrowed a book.

You will find the [whole finished code here](https://github.com/jannden/solidity-contracts/blob/d76f300733c35ddf0895396eaa53b48465581105/BookLibrary.sol).

### Ownable contract

Our main contract will inherit from an `Ownable` contract to manage ownership and restrict access to certain functions (for example, the function for adding new books should be available only to the owner --- the librarian).

Here is a simple version of an Ownable contract:

```solidity
*// SPDX-License-Identifier: UNLICENSED
*pragma solidity ^0.8.0;contract Ownable {
  address public owner; constructor() {
    owner = msg.sender;
  } modifier onlyOwner() {
    require(owner == msg.sender, "Not invoked by the owner");
    _;
  }
}
```

When the contract is initialized, the `constructor` saves the address of the original sender as the owner.

The functionality of a modifier `onlyOwner` will be used for functions with restricted access.

### Book Library contract

Let's create a shell for the main contract, which will be inheriting from the Ownable contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;import "./Ownable.sol";contract BookLibrary is Ownable {

  // ... The rest of our code will go here}
```

### Tracking books and their copies

We will first create a struct that holds the information of each book (name, number of all copies, number of borrowed copies, addresses of all past borrowers).

```solidity
// Struct - holding info of each book
struct Book {
  string name;
  uint256 copies;
  uint256 borrowed;
  address[] allBorrowers;
}
```

Next comes an array of such structs to represent all the books in the library.

```solidity
// Array of structs - all books in the library
Book[] books;
```

Lastly, we will create a mapping that lets us access the id of a book based on its name.

```solidity
// Mapping - for easier access of ids based on book names
mapping(string => uint256) bookNamesToIds;
```

### Is a new book really new?

The librarian should be able to add new books with different amounts of copies.

When adding books to the library, we should first check, whether the particular title already exists (and we just increase the number of copies), or whether we create a completely new record. Let's create a helper function `_isNewBook` that will take care of such checks.

```solidity
function _isNewBook(string calldata _name) private view returns (bool) {
  bool newBook = false; // ... More code return newBook;
}
```

This function receives only one parameter --- book name. It assumes that a book is new if our mapping bookNamesToIds doesn't hold any information about an id for such a name. When there isn't any information, the mapping holds a default value of zero.

```solidity
function _isNewBook(string calldata _name) private view returns (bool) {
  bool newBook = false; *// Mapping holds the default value of zero for non-existent keys*
  if (bookNamesToIds[_name] == 0) {newBook = true; // ... More code } return newBook;
}
```

However, we have to take care of a case where the id of a book is actually zero. So we check the name of the first book in our library (which has id zero) as well.

```solidity
function _isNewBook(string calldata _name) private view returns (bool) {
  bool newBook = false; *// Mapping holds the default value of zero for non-existent keys*
  if (bookNamesToIds[_name] == 0) {newBook = true; *// For the very first book in our array, the key actually is 0
    // So we need to verify such key by name* if (books.length > 0) {
      if (keccak256(abi.encodePacked(books[0].name)) == keccak256(abi.encodePacked(_name))) {
         newBook = false;
      }
    } } return newBook;
}
```

### Adding books to the library

Now we create an `addBook` function for the librarian. It takes two parameters --- book title and amount of copies. It's executable only by the owner of the contract (the librarian).

We run the helper function `_isNewBook` and based on the result:

-   if such a book doesn't exist yet, we will add it with all info,
-   if such a book already exists in the library, we will just increase the number of copies.

```solidity
function addBook(string calldata _name, uint256 _copies) public onlyOwner {
  require(_copies > 0, "Please add at least one copy."); if (_isNewBook(_name)) { *// If such book doesn't exist yet, we will add it with all info* Book memory newBook = Book(_name, _copies, 0, new address[](0));
    books.push(newBook);
    bookNamesToIds[_name] = books.length - 1; } else { *// If such book already exists in the libary, we will just increase the number of copies* books[bookNamesToIds[_name]].copies = books[bookNamesToIds[_name]].copies + _copies; }}
```

### Getting available copies

Now we will create a function with which users can check which books they can borrow.

So, we will go through each book in our `Books` array and find all books where the amount of borrowed copies is less than the total. We will save information about available books in a new array of structs that will hold books' names and ids:

```solidity
struct AvailableBook {
  uint256 id;
  string book;
}
```

Now came the time for the getAvailableBooks() function, which will return an array of such structs.

```solidity
function getAvailableBooks() public view returns (AvailableBook[] memory) {
  // ... more code
  return availableBooks;
}
```

However, in order to create an array in the memory, we have to give it a fixed size. In other words, we have to know, how many available books there are, before we start gathering information about them.

So we proceed step by step:

1.  Create a `for` loop that gets the number of available books.
2.  Create a fixed-size array of `availableBooks` in the memory.
3.  Fill the array with the desired information within a second `for` loop.

The function would look like this:

```solidity
function getAvailableBooks() public view returns (AvailableBook[] memory) { // We will first find out the number of available book titles
  // This is due to the fact, that memory arryas have to be fixed-sized
  uint256 counter = 0;
  for (uint256 i = 0; i < books.length; i++) {
    if (books[i].copies - books[i].borrowed > 0) {
      counter++;
    }
  }// Then we will fetch the data of the available books
  AvailableBook[] memory availableBooks = new AvailableBook[](counter);
  counter = 0;
  for (uint256 i = 0; i < books.length; i++) {
    if (books[i].copies - books[i].borrowed > 0) {
      availableBooks[counter] = AvailableBook(
        bookNamesToIds[books[i].name],
        books[i].name
      );
      counter++;
    }
  } return availableBooks;}
```

### Book must exist

For borrowing and returning books, we have to make a logical test, whether the desired book actually exists in our library.

Let's create a modifier function for that purpose.

```solidity
modifier bookMustExist(uint256 _id) {
  require(books.length > 0, "No books in the library.");
  require(_id <= books.length - 1, "Book with this ID doesn't
exist.");
  _;
}
```

### Borrowing books

Since the users already know which books they can borrow, let's give them a chance to actually do so.

Let's first create another mapping that will list all books a user has borrowed as well as two constants that we will use as statuses.

```solidity
mapping(address => mapping(uint256 => uint256)) borrowerToBookIdsToStatus;uint256 constant BORROWED = 1;
uint256 constant RETURNED = 2;
```

Now we are ready to create a function `borrowBook` that takes the id of the desired book. It also uses that id for the modifier `bookMustExist`.

The function checks whether a user hasn't already borrowed that particular book and whether that book is actually available.

Then it adds the user to the full list of all borrowers of that book and updates our mapping `borrowerToBookIdsToStatus` for the book with the status `BORROWED`.

```solidity
function borrowBook(uint256 _id) public bookMustExist(_id) {
  require(
    borrowerToBookIdsToStatus[msg.sender][_id] != BORROWED,
    "Please return the book first."
  );
  require(
    books[_id].copies - books[_id].borrowed > 0,
    "No available copies."
  ); // Add to allBorrowers if it's the first time user borrows this book
  if (borrowerToBookIdsToStatus[msg.sender][_id] != RETURNED) {
    books[_id].allBorrowers.push(msg.sender);
  } // Borrow this book
  borrowerToBookIdsToStatus[msg.sender][_id] = BORROWED;
  books[_id].borrowed = books[_id].borrowed + 1;
}
```

### Returning books

Returning books is quite straightforward. First, we check whether the user account has such a book, then we update the global variable `borrowerToBookIdsToStatus` as well as the amount of borrowed copies from the global `books` array. Notice that we also use the `bookMustExist` for this function.

```solidity
function returnBook(uint256 _id) public bookMustExist(_id) {
  require(
    borrowerToBookIdsToStatus[msg.sender][_id] == BORROWED,
    "You don't currently have this book."
  );
  borrowerToBookIdsToStatus[msg.sender][_id] = RETURNED;
  books[_id].borrowed = books[_id].borrowed - 1;
}
```

### Get all borrowers

The last function needed to fulfill the requirements from the beginning of this article is to return all borrowers that have ever borrowed a particular book.

Nothing complicated there, we just return the `allBorrowers` property of the book's struct. For logical reasons, we also use the `bookMustExist` modifier for this function.

```solidity
function getAllBorrowers(uint256 _id) public view bookMustExist(_id) returns (address[] memory) {
  return books[_id].allBorrowers;
}
```

### Conclusion

We created a full contract for the Book Library project. I hope you have learned some interesting concepts of programming in Solidity along the way. You will find the [whole finished code here](https://github.com/jannden/solidity-contracts/blob/d76f300733c35ddf0895396eaa53b48465581105/BookLibrary.sol).

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
