## Rust Traits and Generics: A Fun Guide with Examples

_Originally published on Mar 28, 2024 [here](https://medium.com/rustaceans/rust-traits-and-generics-a-fun-guide-with-examples-fb94c0bad81d)._

---

Welcome to the vigorous world of Rust. I will explain all about Traits and Generics, drawing parallels with iconic characters from Peaky Blinders to make the journey fun and practical at the same time.

You can practice the code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium).

### 1\. Traits

In Rust, a Trait is like a person's character. It defines a set of behaviors (methods) that can implemented.

Every person has certain abilities (such as speaking) but each does them differently. Similarly, Rust Traits define what a type must do, but not how to do it.

Imagine defining the traits of the Shelby family members.

```react
// This trait defines how to speak but not what to speak
trait Shelby {
    fn speak(&self);
}

struct ThomasShelby;

struct ArthurShelby;

impl Shelby for ThomasShelby {
    fn speak(&self) {
        // What Tommy says
        println!("I'm Thomas Shelby, and this is my business.");
    }
}

impl Shelby for ArthurShelby {
    fn speak(&self) {
        // What Arthur says
        println!("I'm Arthur Shelby, and I fight for my family.");
    }
}

fn main() {
    let tommy = ThomasShelby;
    let arthur = ArthurShelby;
    tommy.speak();
    arthur.speak();
}
```

As you can see, the `Shelby` trait defines a `speak` method. Both `ThomasShelby` and `ArthurShelby` implement this trait, but each in their way.

### 2\. Traits with Default Methods

Thomas and Arthur are special, but some characters are mundane and don't have their own way of expressing themselves. We can set a default implementation for the `speak` method for such cases.

```react
// This trait defines how to speak and sets the default implementation
trait Shelby {
    fn speak(&self) {
        // Default implementation
        println!("How are ya?");
    }
}

struct FinnShelby;

impl Shelby for FinnShelby {}

fn main() {
    let finn = FinnShelby;
    finn.speak();
}
```

### 3\. Trait Bounds

We can require function parameters to have specific traits.

Let's say we want to make sure that a Shelby family member replies to a greeting from a stranger on the streets of Birmingham. We can make sure that the `meet_and_greet` function accepts only a parameter that implements the `Shelby` trait:

```react
trait Shelby {
    fn speak(&self) {
        println!("How are ya?");
    }
}

struct FinnShelby;

impl Shelby for FinnShelby {}

fn meet_and_greet(shelby: impl Shelby) {
    println!("Stranger: Nice to meet you!");
    print!("Shelby: ");
    shelby.speak();
}

fn main() {
    let finn = FinnShelby;
    meet_and_greet(finn);
}
```

### 4\. Generics

Generics give us the power to write reusable code because we can work with many different data types.

Suppose the Shelbys are planning a new business venture and need a business plan that works with various goods.

```react
struct BusinessPlan<T> {
    goods: T,
}

fn main() {
    let whiskey_plan = BusinessPlan { goods: "Whiskey" };
    let amount_plan = BusinessPlan { goods: 100 };

    println!("Business Plan 1: {}", whiskey_plan.goods);
    println!("Business Plan 2: {}", amount_plan.goods);
}
```

Here, `BusinessPlan<T>` is a generic struct that has a flexible field `goods`. It can take different types, such as string literal `Whiskey` in the first case or a number `100` in the second case.

_Read more about Structs in _[_my previous article_](https://medium.com/rustaceans/rust-structs-a-fun-guide-with-examples-c2851dfb96c6?sk=a1493ed9de65e20c38aa7960d4312588)_._

### 5\. Generics with Trait Bounds

Trait bounds specify what functionality a generic type must provide.

Consider a family meeting where only the members of the Shelby family are allowed.

```react
trait Shelby {
  fn speak(&self);
}

struct ThomasShelby;

impl Shelby for ThomasShelby {
  fn speak(&self) {
      println!("I'm Thomas Shelby");
  }
}

// It's required that only a Shelby member can attend the meeting
// In other words, the parameter accepts only such an argument
// that implements the Shelby trait
fn attend_meeting<T: Shelby>(member: T) {
  member.speak();
}

fn main() {
  let tommy = ThomasShelby;
  attend_meeting(tommy);
}
```

Here, `attend_meeting` is a function that requires its parameter to implement the `Shelby` trait.

It does this by first telling us that it will require a generic type codenamed T that has to have implemented the Shelby trait expressed as `<T: Shelby>`.

Then it specifies that the parameter `member` has to be of that `T` type as in `(member: T)`.

Let's extend the example now to demonstrate multiple trait bounds.

The meeting can be attended by two families now, Shelby and Gold. We would update the function signature accordingly:

```react
fn attend_meeting<T: Shelby + Gold>(member: T) {
  member.speak();
}
```

There is another way to require multiple trait bounds, which is using the `where` syntax. It is usually considered more readable especially as the number of trait bounds grows:

```react
fn attend_meeting<T>(member: T)
where
  T: Shelby + Gold,
{
  member.speak();
}
```

### 6\. Traits with Generics

Combining Traits and Generics brings together the best of both worlds.

Let's expand Shelby's operations to include different types of businesses, each requiring a unique strategy.

```react
trait Operation {
    fn run(&self);
}

struct Business<T> {
    name: T,
}

impl<T> Operation for Business<T> {
    fn run(&self) {
        let _ = &self.name; // Accessing name
        println!("Business run successfully.");
    }
}

fn main() {
    let illegal_business = Business {
        name: "Gambling",
    };

    illegal_business.run();
}
```

In this example, the `Operation` trait is implemented for the `Business` struct which also uses generics.

We can't do much with the `name` field of the struct though. Since it's generic, Rust doesn't know what it will be and what it can do. That's why usually, traits with generics rely on trait bounds.

For example, we can require the generic field `name` to implement the `Display` trait from the standard library, so that Rust is sure it can be printed out.

```react
impl<T: std::fmt::Display> Operation for Business<T> {
    fn run(&self) {
        println!("Business {} run successfully.", self.name);
    }
}
```

### 7\. Trait Objects

Sometimes, you may want to use different types that implement the same trait interchangeably.

Suppose we have Arthur and Ada as different types both implementing the trait Shelby:

```react
trait Shelby {
    fn speak(&self);
}

struct ArthurShelby;
struct AdaShelby;

impl Shelby for ArthurShelby {
    fn speak(&self) {
        println!("I'm Arthur Shelby");
    }
}

impl Shelby for AdaShelby {
    fn speak(&self) {
        println!("I'm Ada Shelby");
    }
}
```

As before, we will create a function that lets them speak.

```react
fn main() {
    let arthur = ArthurShelby;
    let ada = AdaShelby;

    greet_shelby(arthur);
    greet_shelby(ada);
}

fn greet_shelby<T: Shelby>(shelby: T) {
    println!("Stranger: Nice to meet you!");
    print!("Shelby: ");
    shelby.speak();
}
```

That's all fine. However, we had to call the `greet_shelby` function separately for Arthur and Ada.

If we had more Shelbys, it would be more intuitive to store all Shelbys in a vector and then iterate over the vector and call the function within the iteration. For example:

```react
// This won't work

fn main() {
    let arthur = ArthurShelby;
    let ada = AdaShelby;

    let shelbys = vec![&arthur, &ada];

    for shelby in shelbys {
        greet_shelby(shelby);
    }
}
```

Rust doesn't allow us to store different Shelby members in a single collection. We get the `mismatched types` error for the vector.

So, we need to create a vector that holds references to a single trait, which is called a trait object. This allows us to use dynamic dispatch, where the method to call is determined at runtime.

```react
fn main() {
    let arthur = ArthurShelby;
    let ada = AdaShelby;

    let shelbys: Vec<&dyn Shelby> = vec![&arthur, &ada];

    for shelby in shelbys {
        greet_shelby(shelby);
    }
}
```

In this updated version, we use `&dyn Shelby` to create a trait object. This allows us to store references to different types that implement the `Shelby` trait in a single `Vec`.

We also need to update the `greet_shelby` function so that it accepts a trait object instead of a generic parameter with trait bound:

```react
fn greet_shelby(shelby: &dyn Shelby) {
    println!("Stranger: Nice to meet you!");
    print!("Shelby: ");
    shelby.speak();
}
```

The `greet_shelby` function now accepts a reference to a trait object (`&dyn Shelby`).

Even more idiomatic code would be without the references. But we cannot use `dyn Shelby` without the reference in this context, because the size for values of our trait object cannot be known at compilation time.

We can go around it by using the smart pointer `Box<>`:

```react
fn main() {
    let arthur = ArthurShelby;
    let ada = AdaShelby;

    let shelbys: Vec<Box<dyn Shelby>> = vec![Box::new(arthur), Box::new(ada)];

    for shelby in shelbys {
        greet_shelby(shelby);
    }
}

fn greet_shelby(shelby: Box<dyn Shelby>) {
    println!("Stranger: Nice to meet you!");
    print!("Shelby: ");
    shelby.speak();
}
```

Notice that we have replaced `&dyn Shelby` with `Box<dyn Shelby>` and instead of storing references in the vector, we use `Box::new()`. What's the difference?

`Box<>` is a smart pointer that points to a heap-allocated data and owns that data. This means when you use `Box<dyn Shelby>`, you're storing owned trait objects (instead of references) in the vector. This is useful when the trait objects need to live as long as the vector without the need to manage lifetimes explicitly. On the other hand, `&dyn Shelby` refers to a borrowed reference, which means the lifetime of the objects it points to must be managed outside the vector to ensure they live as long as the vector uses them.

This was a rather complex chapter, but I wanted to cover all relevant aspects of trait objects in one place so that you have a good overview.

To sum up, with trait objects, we achieve the flexibility to work with different types implementing the same trait. You will find this useful when you need to store a collection of heterogeneous types that share common behavior.

### 8\. Associated Types in Traits

Within traits, we can define a placeholder type that each implementation of that trait can specify.

Let's say each family member has a trait `Role`. Different roles have different types of duties. So we add a generic placeholder type `Duty` to the `Role` trait.

```react
trait Role {
    type Duty;

    fn perform_duty(&self, duty: Self::Duty);
}

struct ArthurShelby;

impl Role for ArthurShelby {
    type Duty = String; // Arthur's duty expressed as a task in String format

    fn perform_duty(&self, duty: Self::Duty) {
        println!("Arthur's duty: {}", duty);
    }
}

struct FinnShelby;

impl Role for FinnShelby {
    type Duty = i32; // Finn's duties are typically quantifiable, like the number of shipments

    fn perform_duty(&self, duty: Self::Duty) {
        println!("Finn's number of shipments to manage: {}", duty);
    }
}

fn main() {
    let arthur = ArthurShelby;
    let finn = FinnShelby;

    arthur.perform_duty(String::from("Negotiate with the New York gangs"));
    finn.perform_duty(15);
}
```

In this example, the `Role` trait includes an associated type `Duty`. Each implementation of `Role` specifies what `Duty` is---Arthur's duties are represented as strings (perhaps tasks), while Finn's are numerical (like counting shipments). This allows the `perform_duty` method to accept different types of data for each Shelby family member's role, enhancing flexibility and type safety in the family's operations.

### 9\. Using Supertraits to Build on Existing Traits

We can build new traits based on existing ones using supertraits.

The traits of each family member build on one another, each adding layers to their character. This is like saying, "To be a Shelby Leader, one must first be a Shelby."

```react
trait Shelby {
    fn identity(&self) -> String;
}

trait ShelbyLeader: Shelby {
    fn make_decision(&self);
}

struct TommyShelby;

impl Shelby for TommyShelby {
    fn identity(&self) -> String {
        String::from("Tommy Shelby")
    }
}

impl ShelbyLeader for TommyShelby {
    fn make_decision(&self) {
        println!("{} decides to expand to America.", self.identity());
    }
}

fn main() {
    let tommy = TommyShelby;
    println!("I am {}", tommy.identity());
    tommy.make_decision();
}
```

In this example, `ShelbyLeader` is a supertrait of `Shelby`, meaning that before we can implement `ShelbyLeader`, we must also implement `Shelby`.

### 10\. Newtype Pattern for External Types

We sometimes face a situation where we need to implement a trait for a type that we didn't create. The Rust's orphan rule forbids that.

The Newtype pattern provides a handy workaround by allowing us to wrap the external type in a new struct, creating a 'new type' local to our crate. We can then implement any trait on this new type.

Imagine if the Shelbys had to adapt an external business tool for their operations, without being able to modify the original tool:

```react
// Assume `ExternalTool` is a type from a crate we do not own
struct ExternalTool;

// We create a NewType wrapping `ExternalTool`
struct ShelbyTool(ExternalTool);

// We can now implement traits on `ShelbyTool`, even if they are external
trait Adapt {
    fn adapt(&self);
}

impl Adapt for ShelbyTool {
    fn adapt(&self) {
        println!("Adapting the tool for Shelby's use...");
    }
}

fn main() {
    let tool = ShelbyTool(ExternalTool);
    tool.adapt();
}
```

We created `ShelbyTool` as a new type that wraps `ExternalTool`, allowing the Shelby family to adapt and use the tool according to their needs without directly altering the original external type.

### That's it!

I hope you enjoyed these fun comparisons and got a good overview of Traits and Generics in Rust. You will find the important code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium) in case you want to practice them.

If you liked this guide, please hit the clap 👏 button.

Rust on!
