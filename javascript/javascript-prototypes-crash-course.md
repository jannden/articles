# JavaScript Prototypes — Crash Course

*Originally published on Feb 22, 2022 [here](https://medium.com/@jannden/prototypes-a-gentle-explanation-in-javascript-470eb0e372af).*

---

When we create an empty object in JavaScript, it looks empty.

```javascript
const geralt = {};
```

We haven't set any properties to this object, so this will throw an error:

```javascript
console.log( geralt.fight() );
```

However, this will not throw an error:

```javascript
console.log( geralt.hasOwnProperty() );
```

It seems that the method `hasOwnProperty()` actually exists, even though we haven't defined it.

Where does it come from?

### Secret properties of objects

When we create an object, JavaScript automatically adds an invisible property to it. This property is `__proto__`.

It's quite an ugly name, so it's usually pronounced as prototype. Let's check it out:

```javascript
console.log( geralt.__proto__ );
 // → {...}
```

Remember, prototype is just the pronunciation of `__proto__`, and not the actual name, so this doesn't work:

```javascript
console.log( geralt.prototype );
 // → undefined
```

### Object.prototype

We established that JavaScript adds this secret property to our object. But what it actually is? Well, `__proto__` is simply an object with it's own default properties.

To step back a little bit, `Object` is a constructor function that refers to all the objects created in the document. We use `Object.prototype` to add properties or methods to all of the objects that inherit from `Object` and `__proto__` is actually a reference to this default prototype of all objects.

That's why any custom object you create can access the `hasOwnProperty()`.

If we simplify a little bit, arrays and functions are also objects in JavaScript and they have their own default prototypes, `Array.prototype` and `Function.prototype` respectively.

```javascript
console.log(Object.getPrototypeOf({}) === Object.prototype);
 // → trueconsole.log(Object.getPrototypeOf(function () {}) === Function.prototype);
 // → trueconsole.log(Object.getPrototypeOf([]) === Array.prototype);
 // → true
```

### Prototype's properties

And here comes the fun part. When we try to access a property that our object does not have, the object's prototype is searched instead to check whether such property exists at least there.

Back to the first example, when we called `hasOwnProperty()` on our empty object `geralt`, this is what happened:

-   browser sees that `geralt.hasOwnProperty()` does not exist
-   so it looks in `geralt.__proto__`
-   finds that `geralt.__proto__.hasOwnProperty()` exists and calls it

The prototype is full of surprises. For example, it has several other properties in addition to `hasOwnProperty()`. For example:

-   `isPrototypeOf()` --- we can check the parent of the prototype.
-   `propertyIsEnumerable()` --- we can check whether it's possible to run a loop on a property.
-   `toLocaleString()` --- we can format dates.
-   `toString()` --- we can represent the object with a simple string.
-   `__proto__` --- WAIT WHAT?

Yes, indeed, there is another `__proto__` object in `__proto__`.

### Prototype chain

Since prototype is an object (and every object has a prototype), our prototype has also it's own prototype. This is called a prototype chain. The chain ends when we reach a prototype that has null for its own prototype:

```javascript
geralt.__proto__.__proto__
 // → null
```

That's the end of the chain, in which:

-   geralt is our custom object
-   the first `__proto__` is a reference to `Object.prototype` and inherits all of its properties
-   the second `__proto__` is `null` and thus ends the chain

We can create objects with custom prototypes. In another words, we can add links to the chain.

First, we create a helper object:

```javascript
const schoolOfTheWolf = {
  rest: function(witcherName) {
    console.log(`The witcher ${witcherName} rests at Kaer Morhen.`)
  }
}
```

Now we can use that helper object as a prototype for our main object with a special keyword `Object.create` like so:

```javascript
const witcher = Object.create(schoolOfTheWolf);
witcher.rest("Geralt");
// → The witcher Geralt rests at Kaer Morhen.
```

Now the prototype chain would look like this:

```javascript
witcher.__proto__.__proto__.__proto__
```

In this extended chain:

-   witcher is our custom object
-   the first `__proto__` is a reference to `schoolOfTheWolf` and inherits all of its properties
-   the second `__proto__` is a reference to `Object.prototype` and inherits all of its properties
-   the third `__proto__` is `null` and thus ends the chain

Now you understand why the following two lines of code work and are essentially the same:

```javascript
witcher.rest("Geralt");
// → The witcher Geralt rests at Kaer Morhen.witcher.__proto__.rest("Geralt")
// → The witcher Geralt rests at Kaer Morhen.
```

And why we can call `hasOwnProperty("rest")` on the first prototype and get positive result.

```javascript
console.log(witcher.__proto__.hasOwnProperty("rest"))
// → true
```

### Conclusion

We have learned about the secret properties of objects that come from prototypes. We saw a prototype chain and added our own link to it.

If you found this article helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
