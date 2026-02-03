# JavaScript Arrow Functions

*Originally published on Apr 23, 2022 [here](https://medium.com/@jannden/back-to-basics-arrow-functions-in-javascript-eeb74c0548bc).*

---

Arrow functions are one of the most useful features of modern JavaScript. They are concise and easy to use. We will now cover all you need to know to start using them right away.

### The syntax

A simple arrow function could look like this:

```javascript
let multiply = (a, b) => a * b;
```

Let's compare it with a normal function:

```javascript
function multiply (a, b) {
  return a * b;
}
```

Do you see how the normal function takes several lines of code, yet the arrow function fits into one? The arrow functions increase the readability of your code.

### Multi-line Arrow Functions

If you want to fit more code inside an arrow function, you can use the curly brackets.

Let's rewrite the previous example to illustrate the difference:

```javascript
let multiply = (a, b) => {
  return a * b;
}
```

Notice, that in addition to the curly brackets, we had to add the `return` keyword as well.

You can write as much code before the `return` statement as you need.

### Returning objects from Arrow Functions

Single line arrow functions can omit the `return` keyword as they simple return what's behind the arrow. Considering this, it might be a challenge to return an object. For example, this wouldn't work:

```javascript
let multiply = (a, b) => {result: a * b};
// -> error
```

The parser expects the curly brackets enclose a statement, not an object. In order to make it an object, we have to wrap it in simple parentheses:

```javascript
let multiply = (a, b) => ({result: a * b});
```

### Parameters in arrow functions

The previous example of an arrow function took two parameters.

If your function needs only a single parameter, you can simplify it even further by removing the parentheses around the parameter:

```javascript
let display = text => alert(text);
```

If you don't need any parameters, the parentheses will be empty, but they must be there:

```javascript
let display = () => alert('hello');
```

### Absence of the function keyword

Arrow functions shine especially when they are passed to higher order functions.

Let's first have a look at the standard way without using an arrow function:

```javascript
[2,3,1].sort(function(a, b) {
  return a --- b;
});
// -> [1,2,3]
```

That code could be much more readable by using an arrow function. We omit the `function` and `return` keywords and fit everything in one line.

```javascript
[2,3,1].sort( (a, b) => a --- b );
```

### Function constructors

The arrow functions cannot be used as a function constructor. For example, this cannot be rewritten to an arrow function:

```javascript
// defining a constructor function
function Person () {
  this.name = 'John',
  this.age = 40
}// creating an object
const person = new Person();
```

Keep that in mind when creating your objects.

### The 'this' keyword

The `this` keyword also behaves differently. Usually, it represents an object that executed the function.

```javascript
const friend = {
  friendsName: 'John',
  introduce: function () {
    return `${this.friendsName} is my friend.`;
  },
};friend.introduce();
// -> John is my friend.
```

However, that's not the case for arrow functions. They don't have their own `this` context. They inherit it from the parent scope instead. So the following would not work:

```javascript
const friend = {
  friendsName: 'John',
  introduce: () => {
    return `${this.friendsName} is my friend.`;
  },
};friend.introduce();
// -> error
```

In that case, `this` refers to the parent scope, which is the `window` object, which doesn't have a `friendsName` property.

If you want to read more about the magical `this` keyword in JavaScript, open [my article explaining it on fun examples](https://medium.com/@jannden/the-this-keyword-a-gentle-explanation-in-javascript-bdb30a68a5d2).

### Conclusion

We have learned the syntax of arrow functions and how they compare to normal functions. Use them to make your code more compact and easier to read.

If you found this article helpful, please click the clap 👏button, you can also follow me on Medium, and feel free to comment! I'd be happy to help :)
