## Rust Ownership: A Fun Guide with Examples

_Originally published on Apr 3, 2024 [here](https://medium.com/rustaceans/rust-ownership-a-fun-guide-with-examples-be54d4b5d0bf)._

---

Welcome to the clever world of Rust. I will explain all about Ownership, drawing parallels with the world of Danny Ocean to make the journey fun and practical at the same time.

### 1\. No Garbage Collector

A garbage collector automatically cleans up unused data in a program, freeing memory space so the program can run smoothly. It helps avoid memory waste without needing manual cleanup by the programmer.

Rust uses a unique system called ownership and borrowing instead of a garbage collector. This system manages memory by enforcing rules on how data is accessed and when it's deleted.

### 2\. The Rules of Ownership

There are three core rules of ownership in Rust:

1.  Each value in Rust has a variable that's called its owner.
2.  There can only be one owner at a time.
3.  When the owner goes out of scope, the value will be dropped.

```react
// Rule 1: Each value in Rust has a variable that's called its owner
let owner1 = "Some value";

// Rule 2: There can only be one owner at a time
let owner2 = owner1; // Now owner2 is the owner, and owner1 no longer has access

// Rule 3: When the owner goes out of scope, the value will be dropped
{
    let owner3 = "Another value";
    // owner3 is in scope
}
// owner3 goes out of scope here, and "Another value" is dropped
```

Let's explore the rules deeper.

### 3\. Ownership through Variable Binding

When you bind a value to a variable, that variable becomes the owner of the value. This ownership allows Rust to track the lifetime of the value and ensure its proper cleanup when it's no longer needed.

Imagine the Ocean's Twelve crew planning their heist. Each member has a specific role and responsibility.

```react
fn main() {
    let rusty = "Rusty Ryan";
    let danny = "Danny Ocean";
    let linus = "Linus Caldwell";
}
```

We bind the names of three crew members to the variables `rusty`, `danny`, and `linus`. From that point on, each variable owns its string value. At the end of the function, the variables go out of scope, and Rust automatically cleans up the owned values.

### 4\. Moving Ownership

When you assign a value from one variable to another, the ownership is transferred. This is known as a *move*.

Let's say Rusty needs to pass a secret message to Danny.

```react
fn main() {
    let secret_message = String::from("The Bellagio vault is the target.");
    let rusty_message = secret_message;

    // The next line would error out: secret_message is moved
    // println!("Secret message: {}", secret_message);

    // But this is fine as rusty_message is the new owner:
    println!("Rusty's message: {}", rusty_message);
}
```

First, `secret_message` is initially the owner of the string value. When we assign `secret_message` to `rusty_message`, the ownership is moved to `rusty_message`. After the move, `secret_message` is no longer valid, and attempting to use it will result in a compilation error.

### 5\. Ownership Scope

We touched on this earlier, but to demonstrate the ownership scope, let's consider a scenario where Linus is tasked with a specific job within the heist.

```react
fn main() {
    {
        let full_name = "Linus Caldwell";
    }

    // The next line would error out: linus is not available here
    // println!("Linus' name: {}", full_name);
}
```

The variable `full_name` is defined within an inner scope. Once that scope ends (marked by the closing curly brace), `full_name` goes out of scope, and its ownership is released. Attempting to use `full_name` outside its scope will result in a compilation error.

### 6\. Borrowing

Borrowing allows multiple parts of your code to access data without taking ownership. It's like team members sharing the heist plan document without putting it into their pockets and without changing it.

Immutable borrowing is made with the `&` symbol.

Rusty lets other members have a look at the heist plan:

```react
fn main() {
    let rustys_plan = String::from("The Heist Plan");

    let linus_looking = &rustys_plan; // Linus reads the plan

    println!("Plan details: {}", linus_looking);
}
```

Here, `linus_looking` is a reference to `rustys_plan`. Linus borrows the plan to read it but doesn't own it nor can he change it.

Let's make Linus more professional and have him explore the plan in a function, again without taking ownership:

```react
fn analyze_plan(plan: &str) {
    println!("Analyzing borrowed plan: {}", plan);
}

fn main() {
    let heist_plan = String::from("The Heist Plan");

    // Borrow heist_plan temporarily:
    analyze_plan(&heist_plan);

    // The original heist_plan wasn't consumed by analyze_plan and can still be used:
    println!("Original plan: {:?}", heist_plan);
}
```

In this example, the `analyze_plan` function takes a shared reference `&str` as a parameter. By passing `&heist_plan` to the function, we borrow the value of `heist_plan` without transferring ownership. After the function call, we can still use `heist_plan` because we retained its ownership.

You can practice the important code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium).

### 7\. Mutable Borrowing

Mutable borrowing allows the original to be modified without giving away the ownership of it.

Important: When a mutable borrow is active, there cannot be any other borrow at the same time (whether mutable or immutable).

Mutable borrowing is made with the `&mut` symbol.

For example, the team decides to make a last-minute change to the plan. Only one person can take the pen and adjust the document at a time and nobody can observe while the change is being made. Basher takes the initiative:

```react
fn main() {
    let mut heist_plan = String::from("The Heist Plan");

    let basher_will_modify = &mut heist_plan; // Basher will modify the plan

    basher_will_modify.push_str(" with a Twist");

    println!("Updated Original: {:?}", heist_plan);
}
```

Now, `basher_will_modify` is a mutable reference to `heist_plan`, allowing Basher to modify it. The original must also be mutable, so notice we defined it with `let mut heist_plan`.

No one else can borrow `heist_plan` while it's being changed because of the Rules of Borrowing.

### 8\. The Rules of Borrowing

Borrowing must follow two main rules so that memory stays safe.

The first rule states that you can have *any number of immutable* references (`&T`) to a resource, or you can have *one mutable* reference (`&mut T`) at any given time, but not both simultaneously.

```react
fn main() {
    let mut heist_plan = String::from("The Heist Plan");

    let plan_for_discussion = &heist_plan; // Immutable reference
    let second_discussion = &heist_plan; // Another immutable reference is fine

    // The next line would error out: can't borrow `heist_plan` as mutable because it's also borrowed as immutable
    // let plan_for_modification = &mut heist_plan;

    println!("Plan for discussion: {}", plan_for_discussion);
}
```

The second rule states that references must always be valid.

Just as Danny Ocean would never rely on a team member who isn't dependable, Rust ensures that a reference never outlives the data it points to. This prevents dangling references, which could lead to security vulnerabilities.

In our analogy, if a team member leaves the crew, they can no longer participate in discussions or decisions about the heist.

```react
fn main() {
    let plan_for_modification;

    { // An inner scope starts
        let heist_plan = String::from("The Heist Plan");

        // Let's point plan_for_modification to heist_plan:
        plan_for_modification = &heist_plan;
    } // heist_plan goes out of scope and is dropped here

    // The next line would error out: plan_for_modification contains a reference to a dropped value
    // println!("Modified Plan: {}", plan_for_modification);
}
```

By following these rules, Rust's borrowing system ensures that memory management is efficient and safe.

### 9\. Lifetimes

Lifetimes in Rust are like the timeline of each member's involvement in the heist. They ensure that all references are valid for as long as they are needed, and no longer.

Let's consider the next scenario where we need to compare two plans:

```react
fn main() {
    let rusty_plan = String::from("Hack the security system");
    let linus_plan = String::from("Pickpocket the key card");

    let best_plan = compare_plans(&rusty_plan, &linus_plan);

    println!("The best plan is: {}", best_plan);
}
```

Now we will implement the `compare_plans` function without lifetimes:

```react
// Without lifetimes - won't work
fn compare_plans(plan1: &str, plan2: &str) -> &str {
    if plan1.len() > plan2.len() {
        plan1
    } else {
        plan2
    }
}
```

When we run the program, we will get an error. The compiler doesn't know how long the function's return value should live. Why is that a problem?

The return of the function (`&str`) is not a value, but a reference to a value. To comply with the rules of borrowing, the compiler needs to know, how long that reference should live.

In our function, the lifetimes of the references `plan1` and `plan2` are not explicitly connected to the lifetime of the return value. The compiler must be sure that the returned reference does not outlive the borrowed data.

So we need to tell the compiler how long should the return reference live with respect to the parameters. We will do so with lifetimes. Lifetimes are denoted by an apostrophe followed by a name, such as `'a`, `'b`, and so on. The function signature will change to this:

```react
// With lifetime 'a
fn compare_plans<'a>(plan1: &'a str, plan2: &'a str) -> &'a str {
    if plan1.len() > plan2.len() {
        plan1
    } else {
        plan2
    }
}
```

The `'a` annotation is called a lifetime specifier. It's not a specific named lifetime but rather a placeholder that the compiler uses to ensure the references have a consistent lifetime. Notice that it is repeated 4 times in the function signature:

- First at the end of the function name within square brackets. Those square brackets are the space where we specify all lifetime specifiers used in that function. It's just the one named *a*, so it looks like this: `<'a>`. But if there were more lifetime specifiers, all would be listed there, such as `<'a, 'b, 'c>`, and so on.
- Twice for our two parameters. The type of the parameters `plan1` and `plan2` would originally be `&str` but they changed to `&'a str` to indicate that the parameter should live for at least a particular amount of time, which we codenamed *a*.
- And lastly, the value we reference in the function's return should also live at least that long, so `-> &str` changes to `-> &'a str`.

### 10\. Lifetime Elision

In many cases, Rust can infer lifetimes automatically through a process called lifetime elision. This allows us to omit explicit lifetime annotations in certain common scenarios --- that is whenever any of these three rules are fulfilled:

1.  Each parameter, that is a reference, gets its own lifetime specifier. For example, the compiler interprets a function `fn foo(x: &str)` really as `fn foo<'a>(x: &'a str)`.
2.  If there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters. For example, `fn foo(x: &str) -> &str` would be interpreted as `fn foo<'a>(x: &'a str) -> &'a str`.
3.  If there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` (as in method definitions), the lifetime of `self` is assigned to all output lifetime parameters. For example, `impl MyType { fn method(&self) -> &str }` would be interpreted as `impl MyType { fn method<'a>(&'a self) -> &'a str }`.

We don't have to write explicit lifetime specifiers in any of those three cases. That's a convenient shortcut.

### That's it!

I hope you enjoyed these fun comparisons and got a good overview of Ownership in Rust. You will find the important code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium) in case you want to practice them.

If you liked this guide, please hit the clap 👏 button.

Rust on!
