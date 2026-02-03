# Rust: Junior vs Senior Code Comparisons

*Originally published on Oct 10, 2024 [here](https://medium.com/@jannden/rust-junior-vs-senior-code-comparisons-ad43be44d193?sk=b5d53e0cbba2ace69bbdc82dfed45156).*

---

What's the idiomatic use of Rust?

Have a look at different approaches dealing with common syntax. Calling it *junior* and *senior* is a little bit of a healthy exaggeration as you can imagine. The point is to understand the language better and get a feel for its nuances.

## 1\. Fibonacci

### Junior Code

```rust
fn fibonacci(term: u32) -> u32 {
    if term == 0 {
        0
    } else if term == 1 {
        1
    } else {
        fibonacci(term - 1) + fibonacci(term - 2)
    }
}
```

### Senior Code

When dealing with several conditions, consider using `match` for increased readability.

```rust
fn fibonacci(term: u32) -> u32 {
    match term {
        0 | 1 => term,
        _ => fibonacci(term - 1) + fibonacci(term - 2),
    }
}
```

### Epic code

Instead of recursion, we have functional programming here. In this particular case, we use the `fold` method to generate a tuple `(a, b)` where `a` is the *(n-1)th* Fibonacci number and `b` is the *nth* Fibonacci number. We start with `(0, 1)` as the initial tuple, and for each iteration, we generate a new tuple `(b, a + b)`. After `term` iterations, `b` will be the `term`th Fibonacci number, which we return.

```rust
fn fibonacci(term: u32) -> u32 {
    match term {
        0 | 1 => term,
        _ => {
            let fib = (0..term).fold((0, 1), |(previous_value, current_value), _| {
                (current_value, previous_value + current_value)
            });
            fib.1
        }
    }
}
```

## 2\. Looping Structures

### Junior Code

```rust
fn main() {
    let mut number = 3;
    while number > 0 {
        println!("{number}");
        number -= 1;
    }

    println!("Liftoff!");
}
```

### Senior Code

Using a `for` loop with a range rather than `while` is more concise and with a lower risk of errors related to manual control of the loop variable.

```rust
fn main() {
    for number in (1..=3).rev() {
        println!("{number}");
    }

    println!("Liftoff!");
}
```

### Epic code

It's possible to avoid loops altogether and use functional programming to achieve the same result.

```rust
fn main() {
    (1..=3).rev().for_each(|number| println!("{}", number));

    println!("Liftoff!");
}
```

## 3\. Returning Result

### Junior Code

```rust
fn read_file(file_path: &str) -> String {
    let contents = fs::read_to_string(file_path).unwrap();
    contents
}
```

### Senior Code

Return the full `Result` type, which allows the caller to handle potential errors gracefully.

```rust
fn read_file(file_path: &str) -> io::Result<String> {
    fs::read_to_string(file_path)
}
```

## 4\. Option Handling

### Junior Code

```rust
fn find_item(items: &[i32], target: i32) -> i32 {
    for &item in items {
        if item == target {
            return item;
        }
    }
    -1 // Indicates not found
}
```

### Senior Code

Use the `Option` type to represent the possibility of the absence of a value. Leverage iterator methods for concise and clear code.

```rust
fn find_item(items: &[i32], target: i32) -> Option<i32> {
    items.iter().find(|&&item| item == target).copied()
}
```

## 5\. Map with Options

### Junior Code

```rust
fn add_one_to_option(option: Option<i32>) -> Option<i32> {
    match option {
        Some(i) => Some(i + 1),
        None => None,
    }
}
```

### Senior Code

Using the `map` method for `Option` is a more concise way to update the contained value.

```rust
fn add_one_to_option(option: Option<i32>) -> Option<i32> {
    option.map(|i| i + 1)
}
```

## 6\. Using Iterators

### Junior Code

Beginner developers might unnecessarily use the vector's length like `for i in 0..vec.len()`.

```rust
fn sum_elements(vec: Vec<i32>) -> i32 {
    let mut sum = 0;
    for i in 0..vec.len() {
        sum += vec[i];
    }
    sum
}
```

### Senior Code

A more streamlined approach would be `for <item> in &vec`.

```rust
fn sum_elements(vec: Vec<i32>) -> i32 {
    let mut sum = 0;
    for item in &vec {
        sum += item;
    }
    sum
}
```

### Epic Code

Take advantage of iterator methods and eliminate the need for explicit `for` loops altogether.

```rust
fn sum_elements(vec: &[i32]) -> i32 {
    vec.iter().sum()
}
```

## 7\. Factorial

There are four major ways to calculate factorial in Rust.

### Junior code

Using for loops is the least ideal. Rust's iterator methods are often just as fast as manual for loops due to the compiler's ability to optimize them. Also, writing a for loop requires manual management of the loop variable and accumulator which is more error-prone. And it's less readable.

```rust
fn factorial(num: u64) -> u64 {
    if num == 0 || num == 1 {
        return 1;
    }

    let mut result = 1;
    for i in 2..=num {
        result *= i;
    }
    result
}
```

### Recursion

Elegant and clean.

```rust
fn factorial(num: u64) -> u64 {
    match num {
        0 | 1 => 1,
        _ => num * factorial(num - 1),
    }
}
```

### Senior Code

Using functional programming with the fold() method.

```rust
fn factorial(num: u64) -> u64 {
    (1..=num).fold(1, |acc, x| acc * x)
}
```

### Epic Code

The product() method is built for this purpose.

```rust
fn factorial(num: u64) -> u64 {
    (1..=num).product()
}
```

## 8\. Unwrap

### Junior Code

```rust
match option {
    Some(value) => value,
    None => 0,
}
```

### Senior Code

Panic-safe unwrap variants are a more concise way of providing a default value for an `Option`. Consider these:

-   `unwrap_or` --- when you have a default value to provide
-   `unwrap_or_default `--- when the type implements `Default`
-   `unwrap_or_else` --- when the default value needs computation

For example, the previous code could be refactored to:

```rust
option.unwrap_or_default()
```

## 9\. Encapsulation

### Junior Code

```rust
struct User {
    username: String,
    age: u32,
}

fn create_user(username: String, age: u32) -> User {
    User { username, age }
}
```

### Senior Code

Use methods, associated functions, and modules to organize your code.

```rust
mod user {
    pub struct User {
        username: String,
        age: u32,
    }

    impl User {
        pub fn new(username: String, age: u32) -> Self {
            User { username, age }
        }
    }
}
```

## 10\. Match vs If-Let

### Junior Code

```rust
match option_value {
  Some(value) => println!("{}", value),
  None => (),
}
```

### Senior Code

Use `if let` for simplicity when only one pattern is of interest.

```rust
if let Some(value) = option_value {
  println!("{}", value);
}
```

## 11\. Effective Use of Traits

### Junior Code

```rust
fn compare_areas(shape1: &impl Shape, shape2: &impl Shape) {
    if shape1.area() == shape2.area() {
        println!("The areas are the same!");
    }
}
```

### Senior Code

In the code above, `shape1` and `shape2` can be of different types as long as they both implement the `Shape` trait. This might not be desirable if we want both shapes to be of the same type. By introducing a generic type parameter `T`, we can enforce that both `shape1` and `shape2` are of the same concrete type that implements the `Shape` trait, if that's what you need.

```rust
fn compare_areas<T: Shape>(shape1: &T, shape2: &T) {
    if shape1.area() == shape2.area() {
        println!("The areas are the same!");
    }
}
```

## 12\. String parameters

### Junior Code

```rust
fn calculate_length(s: &String) -> usize {
    s.len()
}
```

### Senior Code

Using `&str` instead of `&String` as a function parameter is more flexible as it allows the function to accept both `String` and `&str` types.

```rust
fn calculate_length(s: &str) -> usize {
    s.len()
}
```

### Epic Code

Using `impl AsRef<str>` provides the most flexibility, allowing the function to accept any type that can be referenced as a string slice, including `String`, `&str`, and other types that implement `AsRef<str>`. The main benefit is to improve the developer experience of the consumer of the API, as a function like this can be called both with a wider variety of parameter types.

```rust
fn calculate_length(s: impl AsRef<str>) -> usize {
    s.as_ref().len()
}
```

## 13\. Vector Iteration

### Junior Code

```rust
let numbers = vec![1, 2, 3];
let mut squares = Vec::new();
for &number in &numbers {
    squares.push(number * number);
}
```

### Senior Code

Embrace the functional approach and use iterator methods when possible.

```rust
let numbers = vec![1, 2, 3];
let squares: Vec<_> = numbers.iter().map(|&number| number * number).collect();
```

## 14\. Function returning boolean

### Junior Code

```rust
fn is_even(num: i32) -> bool {
    if num % 2 == 0 {
        return true;
    } else {
        return false;
    }
}
```

### Senior Code

When returning a `boolean` from a function, it's usually possible to make it in a very concise way.

```rust
fn is_even(num: i32) -> bool {
    num % 2 == 0
}
```

## 15\. Closures

### Junior Code

```rust
fn square(num: i32) -> i32 {
    num * num
}

let result = square(4);
```

### Senior Code

Using a closure for a simple operation is usually more practical.

```rust
let square = |num: i32| num * num;

let result = square(4);
```

## 16\. Unwrapping Vectors

### Junior Code

```rust
let numbers = vec![1, 2, 3];
let first = numbers[0];
```

### Senior Code

Safely access vector elements using `get()` together with `unwrap_or()`, `unwrap_or_default()`, `unwrap_or_else()`, `map()`, etc. to handle potential out-of-bounds errors gracefully.

```rust
let numbers = vec![1, 2, 3];
let first = numbers.get(0).unwrap_or(&1);
```

## 17\. Controlling flow

### Junior Code

```rust
fn positive(n: i32) -> Option<i32> {
    if n > 0 {
        Some(n)
    } else {
        None
    }
}
```

### Senior Code

You might see people using `then` when dealing with Options.

```rust
fn positive(n: i32) -> Option<i32> {
    (n > 0).then(|| n)
}
```

## 18\. Using Map on Option

### Junior Code

```rust
let maybe_number = Some(5);
let maybe_string = match maybe_number {
  Some(n) => Some(n.to_string()),
  None => None,
};
```

### Senior Code

`map` is often preferred for transforming an `Option<T>` to `Option<U>`.

```rust
let maybe_number = Some(5);
let maybe_string = maybe_number.map(|n| n.to_string());
```

### Epic Code

Instead of using a closure `|n| n.to_string()`, we can reference the method directly.

```rust
let maybe_string = maybe_number.map(ToString::to_string);
```

## That's it!

Hope you enjoyed these comparisons and got a good overview of Rust code patterns.

If you liked this guide, please hit the clap 👏 button.

Rust on!

*PS. *[*Follow me on Medium*](https://medium.com/@jannden)* for more Rust awesomeness!*
