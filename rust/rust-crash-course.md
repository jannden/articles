## Rust Crash Course for the Impatient

_Originally published on Feb 12, 2024 [here](https://medium.com/rustaceans/rust-crash-course-for-the-impatient-d812e1fb4c05)._

---

Here is what we will cover in this crash course:

1.  Basics of Rust: Learn about variables, data types, and control flow structures.
2.  Functions and Error Handling: Understand how to write functions and handle errors.
3.  Ownership and Borrowing: grasp the fundamental concepts of Rust's ownership, borrowing, and lifetimes.
4.  Structs, Enums, and Collections: explore how to use structs, enums, and collections.
5.  Building a Project: apply the knowledge to build a simple to-do list application.
6.  Debugging and Testing: learn how to debug and write tests.

### What is Rust?

Rust is a modern programming language focused on safety, speed, and concurrency. Rust is a systems programming language, but it's also very versatile and can be used for web applications, game development, and other programming tasks.

### Brief History and Purpose

Rust was first created by Graydon Hoare in 2006 and later sponsored by Mozilla. It emerged from the need to have a language that can provide memory safety (preventing common bugs like buffer overflows) without sacrificing performance.

### Key Features

- Safety: Rust's biggest selling point is its emphasis on safety. It prevents common programming errors that lead to crashes or security vulnerabilities, thanks to its unique ownership system.
- Speed: Rust is designed to be as fast as traditionally fast languages like C and C++. It achieves this speed by doing more checks at compile time, rather than runtime.
- Concurrency: Rust makes it easier and safer to write programs that do many things at once (parallel programming). It offers powerful features to handle concurrent programming without the usual headaches.

Rust has been gaining popularity rapidly because of these features. It's particularly loved for its powerful compiler and effective memory management. It's also open-source, which means it's continually improved by a community of developers.

In the next section, we'll look at how to install Rust and set up your first project. This will be your first step into the world of Rust programming!

### Setting Up the Environment

To start coding in Rust, you first need to set up your programming environment. This involves installing Rust and its package manager, Cargo. Cargo is an essential tool for Rust development, handling project builds, dependencies, and more.

### Download and Install Rust

Go to the official Rust website: <https://www.rust-lang.org/>

Follow the instructions to download and install `rustup`, the Rust toolchain installer.

- On Windows, `rustup` installs both Rust and Cargo automatically.
- On macOS and Linux, after installing `rustup`, run `rustup-init` in your terminal to install Rust and Cargo.

### Verify Installation

Open your terminal or command prompt.

Run `rustc --version` and `cargo --version` to check if Rust and Cargo are installed correctly. You should see the version numbers displayed.

### Create a New Project

Open your terminal or command prompt.

Choose a directory where you want to create your project.

Run `cargo new hello_world`. Cargo will create a new binary project named`hello_world` with the necessary files.

### Explore the Project Structure

Navigate to your project directory.

You'll see a `Cargo.toml` file, which is used to manage dependencies and project settings.

The `src` folder contains your Rust source files. The `main.rs` file is where you will write your Rust code.

### Build and Run Your Project

Inside your project directory, run `cargo build` to compile your project. Cargo creates an executable in the `target/debug` directory.

To run your project, use `cargo run`. This command compiles your project (if necessary) and runs the executable.

You have now created your first Rust project.

In the next sections, we'll dive into writing actual Rust code, starting with the classic "Hello, World!" program.

### Hello, World!

Let's write your first Rust program.

1.  Open `main.rs` File:

2\. Go to the `src` folder in your Rust project.

3\. Open the `main.rs` file. This file will contain your Rust source code.

Replace any existing code in `main.rs` with the following lines:

```react
fn main() {
    println!("Hello, World!");
}
```

This code defines a function `main` that is the entry point of every Rust program.

The `println!` macro prints text to the console. In this case, it prints "Hello, World!".

### Understanding the Code

- `fn` declares a new function.
- `main` is the name of the function. In Rust, `main` is special: it's the first code that runs in every executable Rust program.
- `{}` enclose the function's body.
- `println!` is a macro (indicated by the `!`) that prints text to the console.

### Compiling and Running the Program

Open your terminal or command prompt.

Navigate to your project's root directory.

Run `cargo build`. Cargo compiles your project and creates an executable file. You can run the executable file in your terminal:

- if on Mac, use `./target/debug/hello_world`
- if on Windows, use `.\target\debug\hello_world.exe`

You can also use the command `cargo run` to automatically run the executable file after building.

You should see "Hello, World!" printed to the console.

Congratulations! You've just written, compiled, and run your first Rust program. This simple program is just the beginning.

As you progress through this tutorial, you'll learn how to use Rust to write more complex and interesting programs.

### Understanding Variables and Data Types

In Rust, variables and data types are the fundamental building blocks of programming. Let's explore how you can use variables to store data and the different types of data you can work with in Rust.

### Declaring Variables

Use the `let` keyword to declare a variable.

Example: `let x = 5;` declares a variable `x` and initializes it with the value `5`.

Variables are immutable by default in Rust, meaning their value cannot be changed once set.

### Mutable Variables

To make a variable mutable (changeable), use the `mut` keyword.

Example: `let mut y = 10;` allows you to change the value of `y` later in the program.

### Variable Types

Rust is a statically typed language, which means the type of each variable must be known at compile time.

However, Rust often infers the variable type based on the value assigned, so you don't always need to explicitly state the type.

### Basic Data Types

Rust has several primitive data types. Here are a few:

Integers:

- Types like `i32`, `u32`, `i64`, `u64`, etc.
- `i32` is the default integer type. It's a 32-bit integer.
- `u32` is an unsigned 32-bit integer (cannot be negative).

Floating-Point Numbers:

- Types include `f32` and `f64`.
- `f64` is the default, offering double precision.

Booleans:

- The `bool` type represents a value that can either be `true` or `false`.

Characters:

- The `char` type is a single character.
- Defined with single quotes, e.g., `let letter = 'a';`.

### Example: Using Variables and Types

```react
fn main() {
    let x = 5; // immutable integer
    let mut y = 10; // mutable integer
    y = 15; // changing the value of y
    let is_active = true; // boolean
    let grade = 'A'; // character
    let score: f64 = 92.5; // floating-point with explicit type annotation
    println!("x = {}, y = {}", x, y);
    println!("Is active: {}", is_active);
    println!("Grade: {}, Score: {}", grade, score);
}
```

In this example, we've declared various types of variables and printed their values.

In the next section, we'll look at Rust's control flow structures.

### Control Flow Structures

Control flow refers to the direction the program takes depending on certain conditions or loops.

### Conditions in Rust

The `if` statement allows you to execute certain code based on a condition.

```react
if condition {
     // code to execute if the condition is true
}
```

For example:

```react
let number = 7;
if number < 10 {
    println!("The number is less than 10");
}
```

You can add `else` and `else if` to handle multiple conditions.

```react
if number % 2 == 0 {
    println!("The number is even");
} else {
    println!("The number is odd");
}
```

### Looping in Rust

Rust provides several ways to loop; the most common are `while`, `for`, and `loop`.

The `while` loop repeats a block of code as long as a condition is true.

```react
let mut count = 0;
while count < 5 {
    println!("count = {}", count);
    count += 1;
}
```

The `for` loop iterates over elements of a collection, like an array.

```react
let numbers = [10, 20, 30, 40, 50];
for num in numbers.iter() {
    println!("The number is {}", num);
}
```

The `loop` keyword creates an infinite loop. You usually break out of it using `break`.

```react
loop {
    println!("This will print until we stop it");
    // some condition to break the loop
    if stop_condition {
        break;
    }
}
```

Read more about Rust Flow Control in my [detailed article](https://medium.com/rustaceans/rust-conditionals-a-fun-guide-with-examples-85e831c86501) with metaphors from the world of The Dune.

In the next section, we'll learn about functions in Rust, which allow you to organize and reuse your code effectively. Functions are crucial for building more complex and modular programs.

### Functions in Rust

Functions are central to Rust programming, allowing you to organize your code into reusable blocks. Understanding functions is key to writing efficient and clean Rust code.

### Writing and Calling Functions

Use the `fn` keyword to define a function.

Functions usually have a name, parameters, a body, and optionally, a return type.

```react
fn function_name(param1: Type1, param2: Type2) -> ReturnType {
    // function body
}
```

To call a function, use its name followed by arguments in parentheses. For example: `let result = function_name(arg1, arg2);`

A simple function might look like this:

```react
fn main() {
    let number1 = 10;
    let number2 = 20;
    let sum = add_numbers(number1, number2);
    println!("The sum is: {}", sum);
}

fn add_numbers(x: i32, y: i32) -> i32 {
    x + y // the return value is the result of this expression
}
```

In this example, `add_numbers` is a function that takes two `i32` numbers as parameters and returns their sum. Notice how we call this function in the `main` function and print the result.

### Function with References

Rust's ownership system is a unique feature that helps manage memory safely and efficiently. Here's a simplified overview (we will dig deeper later):

Ownership:

- Each value in Rust has a variable that's its "owner".
- There can only be one owner at a time.
- When the owner goes out of scope, the value is dropped (or memory is freed).

Borrowing:

- You can borrow references to a variable instead of taking ownership.
- References can be mutable (`&mut`) or immutable (`&`).
- Rust enforces rules at compile time to ensure safe memory access.

A function with references using ownership and borrowing might look like this:

```react
fn main() {
    let mut number = 10;
    increase(&mut number);
    println!("Number after increase: {}", number);
}

fn increase(n: &mut i32) {
    *n += 1; // use * to dereference and modify the value
}
```

The `increase` function takes a mutable reference to an `i32` and increases its value by 1. The main function then prints the modified value.

In the next section, we'll explore structs and enums, which add a new layer of power and flexibility to Rust programs.

### Structs and Enums

Structs and enums are powerful features in Rust that help organize data and define its behavior.

### Structs in Rust

Structs (short for structures) are used to create custom data types.

```react
struct StructName {
    field1: Type1,
    field2: Type2,
}
```

For example:

```react
struct Point {
    x: i32,
    y: i32,
}
```

To use a struct, you create an instance of it.

```react
let point = Point { x: 0, y: 0 };
```

To access a field in struct, use the dot notation.

```react
let x = point.x;
```

Read more about Structs in my [detailed article](https://medium.com/rustaceans/rust-structs-a-fun-guide-with-examples-c2851dfb96c6) with metaphors from the Game of Thrones.

### Enums in Rust

Enums (short for enumerations) allow you to define a type by enumerating its possible variants.

```react
enum EnumName {
    Variant1,
    Variant2(Type1),
    Variant3 { field1: Type2 },
}
```

For example:

```react
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

To access an Enum value, use the double dot notation.

```react
let movement = Direction::Up;
```

Read more about Enums in my [detailed article](https://medium.com/rustaceans/rust-enums-a-fun-guide-with-examples-f3bb0f590d93) with metaphors from The Matrix.

### Practical Example: Creating a Basic Game Character

Let's combine structs and enums to create a basic representation of a game character:

```react
struct Character {
    name: String,
    health: i32,
    direction: Direction,
}

#[derive(Debug)] // Necessary to println! the value of the enum
enum Direction {
    North,
    East,
    South,
    West,
}

fn main() {
    let hero = Character {
        name: String::from("Aragon"),
        health: 100,
        direction: Direction::North,
    };

    println!("Character: {} is facing {:?}", hero.name, hero.direction);
}
```

In this example, we defined a `Character` struct with a name, health, and direction. The `Direction` enum lists possible directions the character can face. We then create a character named "Aragon" facing north and print the details.

We will explore error handling next.

### Error Handling

Error handling is a critical aspect of programming in Rust. Rust encourages a proactive approach to handling errors, primarily through the use of the `Result` and `Option` types.

### The `Result` Type

The `Result` type is used for functions that can return an error.

It is an enum with two variants: `Ok(T)` and `Err(E)`, where `T` is the type of the successful value and `E` is the type of the error.

```react
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

When writing functions that can fail, return a `Result`.

Check the result with pattern matching or methods like `unwrap` or `expect`.

```react
fn divide(a: i32, b: i32) -> Result<i32, &'static str> {
    if b == 0 {
        Err("Cannot divide by zero")
    } else {
        Ok(a / b)
    }
}
```

### The `Option` Type

The `Option` type is used when a value may or may not exist.

It is an enum with two variants: `Some(T)` and `None`.

```react
enum Option<T> {
    Some(T),
    None,
}
```

`Option` is useful for functions that might not return a value.

Use pattern matching or methods like `unwrap` or `unwrap_or` to handle `Option` values.

```react
fn find_item(items: &[i32], target: i32) -> Option<usize> {
    for (index, &item) in items.iter().enumerate() {
        if item == target {
            return Some(index);
        }
    }
    None
}
```

### Practical Example: Error Handling in File Operations

Create a file called *example.txt* at the root level of your project and fill it with any text.

```react
use std::fs;
use std::io;

fn read_file_contents(path: &str) -> Result<String, io::Error> {
    let contents = fs::read_to_string(path)?;
    Ok(contents)
}

fn main() {
    match read_file_contents("example.txt") {
        Ok(contents) => println!("File contents: \n{}", contents),
        Err(e) => println!("Error reading file: {}", e),
    }
}
```

In this example, `read_file_contents` tries to read the contents of your file with `fs::read_to_string(path)` and returns a `Result`. The `?` operator at the end of the line reading the string is used to return an `Error` early if the operation fails. If there is no error, we return an `Ok` with the contents of the file. In the `main` function, we use pattern matching to handle the `Result`.

Next, we'll explore Rust's collections and how they can be used to manage groups of data.

### Collections in Rust

Rust offers several types of collections for storing multiple values in a single data structure. The three most important are vectors, strings, and hash maps.

### Vectors: Working with Lists of Items

Vectors (`Vec<T>`) are resizable arrays. They can store multiple values of the same type.

Example of creating and modifying a vector:

```react
let mut numbers = Vec::new();
numbers.push(1);
numbers.push(2);
numbers.push(3);
```

Access elements using indexing or get method.

- Example: `let first = numbers[0];` or `let first = numbers.get(0);`

### Strings: Storing Text

The `String` type is a growable, mutable, owned, UTF-8 encoded string type.

Example of creating and modifying a string:

```react
let mut greeting = String::from("Hello");
greeting.push_str(", world!");
```

You can access parts of a string with string slicing.

- Example: `let hello = &greeting[0..5];`

### Hash Maps: Associating Values

`HashMap<K, V>` stores a mapping of keys of type `K` to values of type `V`.

Example of creating and using a hash map:

```react
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert("Blue", 10);
scores.insert("Yellow", 50);
```

Access values in a hash map using the `get` method.

- Example: `let blue_score = scores.get("Blue");`

### Practical Example: Managing a List of Tasks

```react
fn main() {
    let mut tasks = Vec::new();
    tasks.push("Learn");
    tasks.push("Code");
    tasks.push("Test");

    // Vector could also be initialized with a macro:
    // let tasks = vec!["Learn", "Code", "Test"];

    for task in tasks {
        println!("Task: {}", task);
    }
}
```

In this example, we create a vector of tasks and iterate over it to print each task. This demonstrates how collections like vectors are used to manage groups of data in Rust.

Understanding collections is important for handling grouped data effectively in Rust. In the next section, we will explore Rust's ownership system, a core feature that makes Rust unique and powerful.

### Rust's Ownership System

Rust's ownership system is a set of rules that the compiler checks at compile time. Those rules are designed to improve memory safety without needing a garbage collector. The rules deal with ownership, borrowing, and lifetimes.

### Ownership

Basic Rules of Ownership:

- Each value in Rust has a variable that's called its owner.
- There can be only one owner at a time.
- When the owner goes out of scope, the value will be dropped.

Ownership and Functions:

- Passing a variable to a function will either move or copy the variable, depending on its type.
- After a move, the original variable is no longer valid.

### Borrowing

Instead of taking ownership, you can pass references to a value. This is known as borrowing.

Syntax: `&` for creating a reference, and `&mut` for a mutable reference.

While a mutable reference exists, no other references (mutable or immutable) to that value are allowed.

Rules of References:

- You can have either one mutable reference or any number of immutable references.
- References must always be valid.

### Lifetimes

Lifetimes ensure that references are valid for as long as we need them.

The compiler often infers lifetimes without needing explicit annotation.

When lifetimes must be annotated, the syntax is `<'a>`, where `'a` is a lifetime parameter.

### Example: Demonstrating Ownership and Borrowing

```react
fn main() {
    let string1 = String::from("Hello");
    let string2 = string1;  // string1 is moved to string2

    let string3 = &string2; // borrowing string2
    println!("{}", string3); // this works fine

    // println!("{}", string1); // This line would cause an error because string1 was moved
}
```

In this example, `string1` is moved to `string2`, so `string1` is no longer valid. We then borrow `string2` to `string3` and use it.

The ownership rules are very important for writing safe and efficient programs.

Now we can move on and build a simple project.

### Creating a To-Do List Application

Now that you've learned the fundamentals of Rust, go ahead and create a simple project that puts your knowledge to work.

You could build a simple command-line to-do list application, just to put what you've learned to use. This will help you get practice working with structs, enums, vectors, and handling user input.

### Project Setup

Use Cargo to create a new project: `cargo new rust_todo_app`.

Navigate into your project directory: `cd rust_todo_app`.

### Defining the Task Struct

Define a `Task` struct in `src/main.rs` with properties like `name` and `completed`.

```react
struct Task {
    name: String,
    completed: bool,
}
```

### Adding Tasks to a List

Implement a function to add tasks.

```react
fn add_task(tasks: &mut Vec<Task>, name: &str) {
    tasks.push(Task {
        name: name.to_string(),
        completed: false,
    });
}
```

### Implementing Task Completion

Implement a function to mark tasks as completed.

```react
fn complete_task(tasks: &mut Vec<Task>, index: usize) {
    if let Some(task) = tasks.get_mut(index) {
        task.completed = true;
    }
}
```

### Displaying the Task List

Implement a function to display the list of tasks with their completion status.

```react
fn display_tasks(tasks: &[Task]) {
    for (index, task) in tasks.iter().enumerate() {
        println!("{}: {} - {}", index, task.name, if task.completed { "Completed" } else { "Not Completed" });
    }
}
```

### User Interaction and putting it all together

In the `main` function, we will first create a mutable `tasks` vector.

Then we will create a loop and using the `std::io` library, we will ask the user to write a command (either "add", "complete", "show", or "exit").

We will match the user input and call the appropriate function.

Here is the full code:

```react
use std::io;

struct Task {
    name: String,
    completed: bool,
}

fn main() {
    let mut tasks = Vec::new();

    loop {
        println!("Enter command (add / complete / show / exit");
        let mut command = String::new();
        io::stdin().read_line(&mut command).expect("Failed to read line");

        match command.trim() {
            "add" => {
                println!("Enter task name:");
                let mut task_name = String::new();
                io::stdin().read_line(&mut task_name).expect("Failed to read line");
                add_task(&mut tasks, &task_name);
            }
            "complete" => {
                println!("Enter task index:");
                let mut index_str = String::new();
                io::stdin().read_line(&mut index_str).expect("Failed to read line");
                let index: usize = index_str.trim().parse().expect("Please type a number!");
                complete_task(&mut tasks, index);
            }
            "show" => display_tasks(&tasks),
            "exit" => break,
            _ => println!("Invalid command"),
        }
    }
}

fn add_task(tasks: &mut Vec<Task>, name: &str) {
    tasks.push(Task {
        name: name.to_string(),
        completed: false,
    });
}

fn complete_task(tasks: &mut Vec<Task>, index: usize) {
    if let Some(task) = tasks.get_mut(index) {
        task.completed = true;
    }
}

fn display_tasks(tasks: &[Task]) {
    for (index, task) in tasks.iter().enumerate() {
        println!("{}: {} - {}", index, task.name, if task.completed { "Completed" } else { "Not Completed" });
    }
}
```

Congratulations! You've just built a simple but functional to-do list application in Rust. This project covers many of the fundamental concepts of Rust, like structs, enums, vectors, and handling user input.

As next steps, consider adding more features to your application, such as:

- Deleting tasks.
- Saving tasks to a file.
- Reading tasks from a file.
- Implementing error handling for invalid input.

Continuing with the Rust Crash Course, we will now look at debugging and testing.

### Debugging and Testing

When you're working on larger projects in Rust, it is important to check your code with care. You should ensure that your code produces the correct results without any problems. In this part, we will discuss some simple programming methods for finding and solving problems with Rust code; also, how to write tests for it.

### Debugging in Rust

The simplest form of debugging involves printing values to the console using `println!` or `eprintln!` for standard output and error output, respectively.

```react
println!("Value of x: {:?}", x);
```

You can also use the `dbg!` macro to inspect the value of any given expression.

```react
fn main() {
    let my_number = 9;
    dbg!(my_number + 10);
}
```

The code above will print `my_number + 10 = 19` to the console.

### Writing Tests in Rust

Rust has first-class support for unit testing, allowing you to test your code's functionality directly.

Tests are usually written in the same file as the code.

Use the `#[test]` attribute to mark a function as a test.

```react
#[test]
fn test_add() {
    assert_eq!(add(1, 2), 3);
}
```

Run tests using `cargo test`.

Cargo compiles your code in test mode and runs the annotated test functions.

### Example: Testing the To-Do List Application

```react
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add_task() {
        let mut tasks = Vec::new();
        add_task(&mut tasks, "Learn Rust");
        assert_eq!(tasks.len(), 1);
    }

    #[test]
    fn test_complete_task() {
        let mut tasks = vec![Task { name: String::from("Learn Rust"), completed: false }];
        complete_task(&mut tasks, 0);
        assert!(tasks[0].completed);
    }
}
```

In this example, we're testing the functionality of adding and completing tasks in our to-do list application. We first add a task and then check if the length of the `tasks` vector is 1. Next, we test the `complete_task` function by marking a task as complete and asserting that its `completed` field is `true`.

### Full Code

Finally, it's time to put it all together and run it in the terminal with the `cargo run` command.

You can also open it live in [the playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=6184ecab5fb6916887f115efa519233d) and see the final code [my repo](https://github.com/jannden/rust-examples/tree/main/simple-to-do-list).

### Additional Resources

As you continue to develop your skills in Rust, make sure to engage with the community which is very welcoming and beginner's friendly.

### Further Learning Resources

I write about Rust regularly and post examples of many small projects. Read more on my [Medium profile](https://medium.com/@jannden) and consider subscribing.

In addition to the [official documentation](https://doc.rust-lang.org/book/title-page.html), I also find these resources helpful for beginners:

1.  Tutorials:
    [Educative](https://www.educative.io/courses/learn-rust-from-scratch)
    [Codecademy](https://www.codecademy.com/courses/rust-for-programmers)
2.  Exercises:
    [Rustlings](https://github.com/rust-lang/rustlings/)
    [Exercism](https://exercism.org/tracks/rust)
3.  Examples:
    [Rust By Example](https://doc.rust-lang.org/rust-by-example/)
    [Creative Projects](https://github.com/PacktPublishing/Creative-Projects-for-Rust-Programmers)

### Conclusion

Your journey with Rust is just beginning. With its growing popularity and strong community support, Rust offers a comfy ecosystem to learn and grow in. Just remember, consistent practice and engagement with the community are key to becoming proficient.

If you found this crash course helpful, please click the clap 👏 button.

Rust on!
