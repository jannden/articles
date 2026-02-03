## Rust Enums: A Fun Guide with Examples

_Originally published on Feb 21, 2024 [here](https://medium.com/rustaceans/rust-enums-a-fun-guide-with-examples-f3bb0f590d93)._

---

Welcome to the digital world of Rust. I will explain all about Enums, drawing parallels with The Matrix to make the journey fun and practical at the same time.

You can practice the code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium).

### 1\. The Basics of Enums

An Enum is a type that can be any one of several variants. It's similar to how Neo must understand his identity: is he Thomas Anderson, a regular software engineer, or Neo, the chosen one?

Imagine a scenario where Morpheus offers Neo a choice between two pills.

```react
enum Pill {
    Red,
    Blue,
}

fn main() {
  let choice = Pill::Red; // Neo chooses the Red pill
  match choice {
    Pill::Red => println!("Welcome to the real world."),
    Pill::Blue => println!("Stay in the dream."),
  }
}
```

Here, the `Pill` enum has two variants: `Red` and `Blue`. Neo's choice is represented by one of these variants. The `match` statement then acts like Morpheus, interpreting Neo's decision. If you need a refresher on controlling flow with `match` statements, read [my previous article](https://medium.com/rustaceans/rust-conditionals-a-fun-guide-with-examples-85e831c86501).

### 2\. Enums with Data

Enums in Rust can also hold data, much like how characters in The Matrix possess unique abilities and attributes.

We can use an enum to represent different agents with distinct powers.

```react
enum Agent {
  Smith { strength: u32 },
  Brown { speed: u32 },
}

fn main() {
  let agent = Agent::Smith { strength: 100 };

  match agent {
    Agent::Smith { strength } => println!("Agent Smith with strength: {}", strength),
    Agent::Brown { speed } => println!("Agent Brown with speed: {}", speed),
  }
}
```

In this example, each variant of the `Agent` enum carries different data (`strength` for Smith and `speed` for Brown), demonstrating the versatility of enums in Rust.

### 3\. Methods in Enums

Just as Neo learns to control his abilities in The Matrix, methods can be defined on enums to perform actions based on their variants.

Neo's choice of the pill has a significant impact on his journey.

```react
enum Pill {
  Red,
  Blue,
}

impl Pill {
  fn take_action(&self) {
    match self {
      Pill::Red => println!("You've chosen the path of truth."),
      Pill::Blue => println!("You've chosen the path of blissful ignorance."),
    }
  }
}

fn main() {
  let my_choice = Pill::Red;
  my_choice.take_action();
}
```

Here, the `take_action` method is implemented on the `Pill` enum using `impl`. It interprets the choice made, similar to how Neo's decision shapes his destiny.

### 4\. Option handling with enums

The `Option` enum is one of the pre-made enums in Rust that are used frequently and are very useful. It can represent either `Some` value or `None`.

We can compare the `Option`to Neo's potential to have `Some` title (in his case, it would be 'The One') or `None`.

```react
fn main() {
  let neo: Option<&str> = Some("The One");

  match neo {
    Some(title) => println!("Neo is {}", title),
    None => println!("Neo is just another human."),
  }
}
```

This snippet uses `Option` to capture the uncertainty of Neo's true identity, showcasing how Rust's enums can express the presence or absence of a value.

### 5\. Error handling with enums

Just as characters in The Matrix experience glitches, our code is not resistant to problems either. That's why proper error handling is crucial. Fortunately in Rust, we can elegantly represent possible error states with enums.

Consider a scenario where Neo tries to access a secure database but encounters various errors.

```react
enum DatabaseError {
  ConnectionLost,
  AccessDenied,
  NotFound,
}

fn access_database() -> Result<String, DatabaseError> {
  if rand::random() {
      Ok("You are in.".to_string())
    } else {
      // Access might be randomly denied
      Err(DatabaseError::AccessDenied)
    }
}

fn main() {
  match access_database() {
      Ok(data) => println!("Data retrieved: {}", data),
      Err(DatabaseError::ConnectionLost) => println!("Error: Connection lost."),
      Err(DatabaseError::AccessDenied) => println!("Error: Access denied."),
      Err(DatabaseError::NotFound) => println!("Error: Data not found."),
  }
}
```

The `DatabaseError` enum defines different error types that can occur. The `access_database` function, instead of returning data directly, returns a `Result` type, which is a pre-made enum that can be:

- `Ok` containing successful data,
- `Err` containing error data, which in our case would be an enum variant of `DatabaseError`.

The `match` statement in the `main` function handles these potential errors, similar to how Neo and his allies adapt to the challenges they face.

### 6\. Enums with Match Guards

Match guards give us the power to do some sensible decision-making processes within enums.

Let's illustrate this with an example where Neo faces different scenarios based on his health status.

```react
enum Scenario {
  Battle,
  Exploration,
}

fn match_guard(health: u32) -> Scenario {
  match health {
    h if h > 50 => Scenario::Battle,
    _ => Scenario::Exploration,
  }
}

fn main() {
  let neo_health = 30;
  let scenario = match_guard(neo_health);

  match scenario {
    Scenario::Battle => println!("Neo chooses to fight."),
    Scenario::Exploration => println!("Neo chooses to explore."),
  }
}
```

The `choose_scenario` function uses a match guard (`h if h > 50`) to decide the scenario based on Neo's health. If Neo's health is above 50, he chooses to fight. Otherwise, he opts for exploration.

### 7\. Enums with Generics

Generics in Rust provide the flexibility to write code that can operate on different data types. When combined with enums, generics allow us to define more versatile and reusable structures.

Let's create an enum that can hold different data types, reflecting the diverse nature of the Matrix.

```react
enum MatrixEntity<T, U> {
  Human(T),
  Program(U),
}

fn main() {
  let neo: MatrixEntity<&str, &str> = MatrixEntity::Human("The One");
  let smith: MatrixEntity<i32, &str> = MatrixEntity::Program("Agent Smith");

  match neo {
    MatrixEntity::Human(name) => println!("Neo is known as {}", name),
    MatrixEntity::Program(_) => println!("It's a program, not Neo"),
  }

  match smith {
    MatrixEntity::Human(_) => println!("It's a human, not an agent"),
    MatrixEntity::Program(name) => println!("Program {} identified", name),
  }
}
```

Notice that `MatrixEntity` is a generic enum that can represent a human or a program, using different combinations of data types for each variant. `T` and `U` are generic type parameters that make the enum flexible and reusable for various types.

### 8\. Enum Forwarding with Delegation

Enums can delegate responsibilities to their variants, a technique that mirrors the way different characters in The Matrix might take on specific roles or actions. This delegation can be implemented using traits and the `impl` keyword, allowing each variant of an enum to behave differently under the same method call.

Let's illustrate this with an example where different characters in the Matrix have unique responses to a situation.

```react
trait Action {
  fn act(&self);
}

enum Character {
  Neo,
  Morpheus,
  Trinity,
}

impl Action for Character {
  fn act(&self) {
    match self {
      Character::Neo => println!("Neo chooses to fight."),
      Character::Morpheus => println!("Morpheus offers wisdom."),
      Character::Trinity => println!("Trinity hacks the system."),
    }
  }
}

fn main() {
  let character = Character::Trinity;
  character.act();
}
```

The `Action` trait defines an `act` method. Each character variant implements this method differently, signifying their unique response or action.

### 9\. Enums in State Design Pattern

Enums in Rust can be leveraged to implement the State design pattern, mirroring the dynamic changes in the Matrix.

Just as characters in The Matrix transition between different states of awareness and capability, software components can change their behavior based on their state, without altering their type. Enums facilitate this by encapsulating the different possible states and behaviors into variants.

```react
enum MatrixState {
    RealWorld,
    MatrixSimulation,
}

impl MatrixState {
    fn switch(&self) -> MatrixState {
        match self {
            MatrixState::RealWorld => MatrixState::MatrixSimulation,
            MatrixState::MatrixSimulation => MatrixState::RealWorld,
        }
    }
}

fn main() {
    let current_state = MatrixState::RealWorld;
    let new_state = current_state.switch();
    // new_state is now MatrixSimulation
}
```

Neo transitions between the real world and the Matrix simulation, similar to how enums can elegantly handle state transitions in Rust.

### 10\. Recursive Enums

Recursive enums allow for the definition of data structures that can contain themselves. This is particularly useful for creating tree-like structures, such as abstract syntax trees in compilers or various hierarchical models.

```react
enum MatrixComponent {
    Node(String, Vec<MatrixComponent>),
    Leaf(String),
}

fn main() {
    let system = MatrixComponent::Node("Root".to_string(), vec![
        MatrixComponent::Leaf("Leaf 1".to_string()),
        MatrixComponent::Node("Node 1".to_string(), vec![
            MatrixComponent::Leaf("Leaf 2".to_string()),
            MatrixComponent::Leaf("Leaf 3".to_string()),
        ]),
    ]);
    // We now have a recursive enum that contains itself over and over...
}
```

This example represents a hierarchical structure (using a recursive enum) such as the layered reality of the Matrix.

### That's it!

Hope you enjoyed these fun comparisons and got a good overview of Enums in Rust. I added all code snippets to [Rust Drills](https://www.rustdrills.com/?utm_source=medium) in case you want to practice them.

If you liked this guide, please hit the clap 👏 button.

Rust on!

_PS. \*\*You can find more of my fun Rust guides _[_on this list_](https://medium.com/@jannden/list/fun-rust-guides-b69e0e65a0b0)_._
