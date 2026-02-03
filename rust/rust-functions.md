## Rust Functions: A Fun Guide with Examples

_Originally published on Mar 13, 2024 [here](https://medium.com/rustaceans/rust-functions-a-fun-guide-with-examples-e9b43833a39b)._

---

Welcome to the galaxy of Rust programming. I will explain all about Functions, drawing parallels with Star Wars to make the journey fun and practical at the same time.

You can practice the code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium).

### 1\. How to Create a Function

In Rust, functions are like the starships in Star Wars --- essential tools to navigate the vastness of coding space.

Consider a scenario where you're preparing a starship for launch.

```react
// Defining a function to launch a starship
fn launch_starship() {
  println!("Starship launched!");
}

fn main() {
  // Calling the launch_starship function
  launch_starship();
}
```

You can see the `launch_starship` function that focuses on a specific task. That function is called inside of another function called `main`.

The `main` function is like a wrapper to everything --- as space is a wrapper to all spaceships. It's the entry point of every program you make in Rust.

### 2\. Parameters and Arguments

Starships can be customized for different missions. Similarly, functions in Rust can take parameters to perform varied tasks.

Let's load specific cargo into the Millennium Falcon using a function with parameters.

```react
// Function with parameters to load cargo
fn load_cargo(cargo: &str, quantity: u32) {
  println!("Loading {} units of {}", quantity, cargo);
}

fn main() {
  // Calling the function with arguments
  load_cargo("Coaxium", 100);
}
```

Here, `cargo` of type `&str` and `quantity` of type `u32` are parameters that allow the `load_cargo` function to be versatile and reusable for different types of cargo.

The specific values we use as parameters such as `"Coaxium"` and `100` are called arguments.

### 3\. Return Values

Just as starships return with valuable data or cargo, functions in Rust can return values after execution.

Imagine sending a reconnaissance mission to find a new base for the Rebel Alliance.

```react
// Function returning the location of a new base
fn scout_for_base() -> String {
  "Yavin IV".to_string()
}

fn main() {
  // Function returning a value
  let base_location = scout_for_base();
  println!("New Rebel base located at: {}", base_location);
}
```

The function `scout_for_base` returns a `String` containing the location of a new base, similar to a successful scouting mission in "Star Wars."

_If you want a refresher on the types _`*String*`_ and _`*&str*`_, read my _[_article about strings_](https://medium.com/rustaceans/rust-strings-a-fun-guide-with-examples-e412ff963465?sk=f7c6a09b38b4f3b2de76945cd16ae40a)_._

### 4\. Function Signatures

Function signatures in Rust are like the mission briefings in Star Wars. They outline what a function will do and what it needs, but we wouldn't care about the internals of the function.

If we want to plan a mission to Endor, we need to define what types of resources are needed (parameters of the function) and what the mission will accomplish (return values of the function).

```react
fn plan_mission_to_endor(troops: u32, starships: u32) -> String {
  format!("Sending {} troops and {} starships to Endor.", troops, starships)
}

fn main() {
  let mission_plan = plan_mission_to_endor(1000, 5);
  println!("{}", mission_plan);
}
```

A function signature is the name of the function, its parameters, and return values. In our case, the function signature is:
`fn plan_mission_to_endor(troops: u32, starships: u32) -> String`

### 5\. Methods and Associated Functions

Methods and associated functions can be likened to the specialized capabilities of droids in Star Wars.

Let's use R2-D2's methods to hack into the Death Star's mainframe.

```react
struct Droid {
  name: String,
}

impl Droid {
  // Associated function to create a new Droid
  fn new(name: &str) -> Droid {
    Droid {
      name: name.to_string(),
    }
  }

  // Method for a Droid to hack systems
  fn hack_system(&self) {
    println!("{} is hacking into the system...", self.name);
  }
}

fn main() {
  // Create a new Droid using the associated function
  let bb8 = Droid::new("BB-8");

  // Let the droid use a method
  bb8.hack_system();
}
```

So what's the difference between an associated function and a method? Notice the `&self` part (or sometimes `&mut self`). I am simplifying a little here, but a method has access to the Struct itself and can either read it or modify it. An associated function cannot.

There is also a difference in how the functions are called. Methods are called on an instance of the struct, while associated functions are called on the struct itself.

_Read more about methods and associated functions in my _[_detailed article about structs_](https://medium.com/rustaceans/rust-structs-a-fun-guide-with-examples-c2851dfb96c6?sk=a1493ed9de65e20c38aa7960d4312588)_._

### 6\. Error Handling

Handling errors in functions is like piloting a starship through an asteroid field. You should be prepared for and respond to unexpected situations.

Imagine piloting the Millennium Falcon while evading TIE fighters, using error handling to navigate challenges.

```react
fn navigate_asteroid_field(speed: u32) -> Result<(), String> {
  if speed < 20 {
    Err("Speed too slow! Risk of asteroid collision.".to_string())
  } else {
    Ok(())
  }
}

fn main() {
  match navigate_asteroid_field(15) {
    Ok(()) => println!("Safe passage through the asteroid field."),
    Err(e) => println!("Error: {}", e),
  }
}
```

The function `navigate_asteroid_field` returns a `Result` type, indicating either successful navigation (`Ok`) or an error (`Err`). This is similar to responding to the ever-changing conditions of space travel.

_Read more about match statements and flow control in my _[_dedicated article_](https://medium.com/rustaceans/rust-conditionals-a-fun-guide-with-examples-85e831c86501?sk=94edf6b826f7e5fe350f8d5f082ca6bd)_._

### 7\. Closures

Closures in Rust are like using the Force in Star Wars. They are powerful, flexible, and can capture their environment.

Although closures are similar to functions, there are differences in their syntax. Compare a closure to a function:

```react
fn  my_function   (x: u32) -> u32 { x + 1 }
let my_closure  = |x: u32| -> u32 { x + 1 };
```

We can omit the types when they can be inferred from their usage. Also, we can omit the curly brackets if the body has only one expression. So a simple closure could look like this:

```react
let simple_closure = |x| x+1;
```

If a Jedi was using the Force to move objects, it would be similar to how closures capture variables from their surroundings.

```react
fn main() {
  let force_strength = 10;

  // The force is a special type of function called closure
  let use_force = |mass: u32| mass < force_strength;

  let spaceship_mass = 5;
  if use_force(spaceship_mass) {
    println!("Using the Force to move the object.");
  } else {
    println!("Object is too heavy for the Force.");
  }
}
```

Notice that we don't pass the value of the `force_strength` variable as an argument to the closure. It's just captured from the environment. On the other hand, the closure receives the `mass` value from the parameter.

A closure can capture its environment in three ways: taking ownership, borrowing mutably, or borrowing immutably. It depends on how the closure is used in the code. The default is to borrow immutably, which is what happens in the example with `force_strength`. Rust infers that `force_strength` does not need to be modified by the closure, so it allows the closure to access it through an immutable borrow.

The choice of whether to use capturing variables from the environment or passing them as parameters is about the balance between readability, performance, and the Rust ownership rules. For example, capturing might be the right choice if a closure is only used in a narrow scope and closely tied to that scope's variables. On the other hand, if the closure is part of a public API or designed to be reused in different contexts, making it accept arguments might be more appropriate.

### That's it!

I hope you enjoyed these fun comparisons and got a good overview of Functions in Rust. I added all code snippets to [Rust Drills](https://www.rustdrills.com/?utm_source=medium) in case you want to practice them.

If you liked this guide, please hit the clap 👏 button.

Rust on!
