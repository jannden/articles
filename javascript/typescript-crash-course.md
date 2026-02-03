# TypeScript --- Crash Course

*Originally published on Feb 17, 2022 [here](https://medium.com/@jannden/typescript-crash-course-79675b69d803).*

---

TypeScript is an open-source language built on top of JavaScript. It offers everything that JavaScript does, plus some additional features.

The main benefit of TypeScript is the ability to use static types. Hence the name "type-script".

What do I mean by static types?

### Dynamically and statically typed languages

Dynamically typed languages (JavaScript, Python, Ruby, PHP) don't specify types when creating variables. For example, you can do the following in JavaScript:

```javascript
let price; // defining a variable, not specifying any type
price = 123; // now the variable holds a number type
price = "123"; // now the same variable holds a string type
// That might cause hard-to-spot bugs...
```

Statically typed languages (TypeScript, Java, C++, Rust, Go) require that you explicitly assign types to variables, function parameters, return values, etc. Such languages might require some extra work from the programmer upfront, but ultimately help to prevent bugs and undesirable program behavior later.

Soon, we will get to an example of what TypeScript code looks like. But first, let's install it.

### How to use TypeScript

Run the following command to install TypeScript as a dev dependency:

```bash
npm install typescript --save-dev
```

Since TypeScript is just a super-set of JavaScript, it needs to be compiled down to regular JavaScript so that the code can actually be executed. The TSC (TypeScript Compiler) takes care of that. You can use a special file [tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) to configure how TypeScript works in your environment and how it should be compiled into JavaScript. You don't have to worry about that now, the default settings are sufficient.

Start using TypeScript by creating a file `index.ts`. Notice the file extension. Instead of a regular JavaScript file extension .js, we use .ts for TypeScript (or .tsx, if you want to use JSX syntax like in React).

Let's rewrite the JavaScript code from the beginning of this article into TypeScript:

```javascript
let price: number; // Here we explicitly state that the variable price should be a number
price = 123; // This will work
price = "123"; // This will throw an error
let anotherPrice: number = 123; // We can define the type and assign value on one line as well
```

You see that the difference is really tiny. We just declared a variable type on row #1.

When you delete row #3 which now throws an error, you can run the TypeScript Compiler on the file by writing this command in your console:

```bash
tsc index.ts
```

This will generate a new file `index.js` which contains the code converted to simple JavaScript so that the browsers and Node.JS understand it.

You can also set the TypeScript Compiler to automatically compile your file on every save with this command:

```bash
tsc --watch index.ts
```

Let's now have a look at the different types of variables that we can actually declare.

### Basic Types in TypeScript

The most basic types are:

-   `number`
-   `string`
-   `boolean`

If you want your variable to take any type, you can simply use:

-   `any`

Let's have a look at the actual code:

```javascript
let price: number = 5;
let product: string = "Daily Prophet";
let isInStock: boolean = true;
let buyers: any = "Wizarding World"; // this variable can take any type

buyers = true; // so we can assign a value of another type to buyers;
```

If we want to be a little more specific than `any`, we can use a `union` of several types. The syntax for that uses the pipe character dividing any possible types as in: `type | type | type`.

```javascript
let sellers: number | string = "Ten sellers";
sellers = 10; // number is accepted as well
```

### Type Assertions

Let's say we have a variable that can hold `any` type. We want to create another variable with a more specific type and assign it the value of the first one. That's when we can use Type Assertion. For example:

```javascript
let headcount: any = 1000; // any value type accepted here

// We want to have only the number type in a new variable studentsAtHogwarts
// So we can use Type Assertion

// Syntax option #1, which works in .ts as well as .tsx
let studentsAtHogwarts1 = headcount as number;

// Syntax option #2, which works in .ts only
let studentsAtHogwarts2 = <number> headcount;
```

### Arrays in TypeScript

An array holds multiple values in a sequence. There are two ways of declaring an array in TypeScript:

1.  Using square brackets with syntax `elementType[]`
2.  Using a generic array type with syntax `Array<elementType>`

Here comes a demonstration of both types.

```javascript
let games: string[] = ["Wizard's Chess", "Quidditch", "Gobstones"]
let publications: Array<string> = ["Daily Prophet", "The Quibbler", "Witch Weekly"];
```

### Tuples in TypeScript

Similar to arrays, tuples hold multiple elements in a sequence. However, we can specify a different type for each element.

The syntax is `[elementType, elementType, elementType]` and so on.

It's also possible to create an array of tuples, which would have a syntax like this: `[elementType, elementType, elementType][]` --- notice the extra pair of square brackets at the end.

```javascript
let funSpell: [number, string, boolean] = [1, "Alohomora", true]; // simple tuple
let idList: [number, number, number, string] = [21, 12, 33, "sixty nine"]; // another simple tuple

// And now an array of tuples:
let tupleArray: [number, string][] = [ [5, "Expelliarmus"], [7, "Lumos"] ];
```

### Functions in TypeScript

We can specify the types of parameters as well as the types of function's return value.

```javascript
// Functions
function castSpell(spellId: number | string, target: string) {
  // This function takes two parameters
  // The first parameter spellId can hold either a numeric or a string value
  // The second parameter target can hold a string value
}

// The previous function could return any value as we haven't specified the return value
// Let's fix it now
function castAdvancedSpell(spellId: number | string, target: string): string {
  // This function has to return a string
  return "Success!";
}

// If we don't want the function to return anything, we can use the void keyword
function noReturn(): void {
  // This function shouldn't return a value
}
```

### Generic functions in TypeScript

Generics make our code more dynamic. They are especially helpful for creating reusable components.

For example, let's say we have a function for scoring students' exams. Sometimes we might score them numerically (eg. 100 points), other times we would use verbal descriptions ("excellent").

Let's write a generic function to find the most frequent score from all students.

```javascript
// Notice the keyword "ScoreType"
// which I chose to represent a placeholder for a type that will be specified later
function mostCommonScore<ScoreType> (allScores: ScoreType[]): ScoreType {
  return allScores.sort((a,b) =>
        allScores.filter(v => v===a).length
      - allScores.filter(v => v===b).length
  ).pop();
}

// Now we can use fill the placeholder with a desired type
console.log(
  mostCommonScore<number>([50, 75, 75, 100])
); // -> 75
console.log(
  mostCommonScore<string>(["Average", "Good", "Good", "Excellent"])
); // -> Good
```

### Enum in TypeScript

Enums allow us to declare a set of named constants with numeric or string values. Best to explain in code:

```javascript
// These will get automatically assigned numerical values
enum bookCategories {
  History, // -> 0
  Beasts, // -> 1
  Spells, // -> 2
}

// We can also assign our own numeric values
// If we omit a numeric value, automatically it will be the next in the sequence
enum spellStrengths {
  Defensive = 5, // -> 5
  Offensive, // -> 6
  Fun = 10, // -> 10
}

// String values are not a problem either and we can mix them in with numerical values
enum spellNames {
  Defensive = "EXPELLIARMUS", // -> EXPELLIARMUS
  Offensive = "OBLIVIATE", // -> OBLIVIATE
  Fun = "ALOHOMORA", // -> ALOHOMORA
  SpellCount = 3, // -> 3
  AnotherCount, // -> 4
}
```

### Interface in TypeScript

An interface defines a specific structure for an object.

```javascript
// First we define an interface
interface StudentOfHogwarts {
  id: number,
  name: string,
  magic(spell:string): void,
  age?: number, // notice the question mark which means this property is optional
  readonly house: string, // the readonly property can be assigned only when the object is instantialized and can never be changed again
}

// Now we can create an object with that interface
const harry: StudentOfHogwarts = {
  id: 1,
  name: "Harry",
  magic: (spell) => console.log(spell),
  house: "Gryffindor"
}
```

### Class in TypeScript

Similarly to objects, classes can also use interfaces. We can even re-use the interface from the previous example. Notice that we assign the type to the class with the `implements` keyword.

```javascript
// We will re-use the StudentOfHogwarts interface from the previous example
// Notice the "implements" keyword
class Student implements StudentOfHogwarts {
  id: number
  name: string

  constructor(id: number, name: string) {
    // Constructor is basically a method that gets called on object instantiation
    this.id = id
    this.name = name
  }

  magic(spell: string): string {
    return `${this.name} cast ${spell}!`
  }
}

const potter = new Student(1, "Harry");

// We can also re-use the enum type example from earlier
console.log(potter.magic(spellNames.Fun));
// -> Harry cast ALOHOMORA!
```

We can further extend our class. Let's say we won't create a class for the next generation of students, which would inherit the parent's properties.

In addition to the `extends` keyword, let's also demonstrate the `private` and `protected` keywords.

```javascript
// Extending the previous example with classes
class YoungerStudent extends Student {
  private parents: string[]; // this property is only accessible in this class
  protected relatives: string[]; // this is only accessible in this class and it's sub-classes

  constructor(id: number, name: string, parents: string[], relatives: string[]) {
    super(id, name); // We use the same initialization of properties as the parent-class
    // But we don't have "parents" and "relatives" in the parent-class, so we initialize those here
    this.parents = parents;
    this.relatives = relatives;
  }

  meetTheParents():void {
    console.log(this.parents);
  }
}

const potter2 = new YoungerStudent(2, "James", ["Harry", "Ginny"], ["Lilly", "Molly"]);

// We cannot directly read the private property "parents"
potter2.parents; // this will throw an error
// Instead, we can use a method, that reads the property within the class
potter2.meetTheParents();
```

### TypeScript in React

A slightly different extension .tsx enables us to write JSX code in TypeScript. Assuming you know React, this is what a simple component supporting TypeScript might look like. We create an interface for props:

```javascript
import * as React from "react";
import { render } from "react-dom";

export interface CustomPropsType {
  title: string;
  message?: string; // we set this prop to be optional
}

function App(props: CustomPropsType) {
  return (
    <>
      <h1>{props.title}</h1>
      <p>{props.message ? props.message : "No message available."}</p>
    </>
  );
}

render(<App title="Welcome" />, document.getElementById("root"));
```

### Conclusion

Using TypeScript will make your code more resistant to bugs and unexpected behavior. Hope this article showed you that it's not actually that hard to start using it.

If you found this article helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
