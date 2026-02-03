# JavaScript Class Example --- with Constructor, Getter, Setter, Extends

_Originally published on Jun 20, 2023 [here](https://medium.com/@jannden/javascript-class-example-with-constructor-getter-setter-extends-4635347b7ceb)._

---

This article serves as a cheat sheet for Classes in JavaScript.

Here is an example of a complete class with explanatory comments:

```react
class Person {
  constructor(name) {
    // We use _ convention to create properties (and prevent a stack overflow)
    // However such a property is not private:
    this._name = name;
  }

  get name() {
    return this._name;
  }

  set name(newName) {
    this._name= newName;
  }
}

class Programmer extends Person {
  // To create a private property, we use the # prefix
  // We have to initialize a private property in the upper scope of the class:
  #language;

  constructor(name, language) {
    // Accessing a property of the parent class with super():
    super(name);

    // Setting a private property:
    this.#language = language;
  }

  writesCode() {
    console.log(
      this._name + " writes code in "+ this.#language
    )
  }
}

// Initializing the original class:
const freelancer = new Person("John");

// Using the getter:
console.log(freelancer.name);

// NOT using the getter (accessing the property directly, bypassing the getter):
console.log(freelancer._name);

// Using the setter:
freelancer.name = "Doe";
console.log(freelancer.name);

// Initializing extended class:
const coder = new Programmer("Tony", "JavaScript");
coder.writesCode();

// Private property is not accessible:
// This would throw a syntax error: console.log(freelancer.#language);
```

### Comparison of constructor functions VS classes

ES6 offered the possibility of using the `class` keyword, but it's still possible to use the old Prototypal syntax to get the same result.

This shows the exact same class as above, but defined in the 'old way':

```react
function Person2(name) {
    this._name = name;
};

// Adding getters and setters (ES5)
Object.defineProperty(Person2.prototype, "name", {
    get: function name() {
        return this._name
    },
    set: function name(newName) {
        this._name = newName
    }
});

// Initializing:
const freelancer2 = new Person2("John");

// Using the getter:
console.log(freelancer2.name);

// NOT using the getter (accessing the property directly, bypassing the getter):
console.log(freelancer2._name);

// Using the setter:
freelancer2.name = "Doe";
console.log(freelancer2.name);

// And it's possible to extend the prototype like so:
Person2.prototype.writesCode = function writesCode(language) {
  console.log(`${this.name} writes code in ${language}`);
};
freelancer2.writesCode('JavaScript');
```

### Conclusion

This short article showed an example of a JavaScript class with a Constructor, Getter, Setter, and Extends. It also showed how to create such a class with Constructor Function.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
