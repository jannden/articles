## Rust Control Flow: A Fun Guide with Examples

_Originally published on Feb 19, 2024 [here](https://medium.com/rustaceans/rust-conditionals-a-fun-guide-with-examples-85e831c86501)._

---

Welcome to the spicy world of Rust. I will explain all about Control Flow mechanisms (Conditionals, Loops, Pattern Matching), drawing parallels with the world of Dune to make the journey fun and practical at the same time.

You can practice the code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium).

### 1\. Conditional Statements

In the world of Dune, making decisions is crucial for survival, much like using conditional statements in Rust.

Imagine Paul Atreides standing at a crossroads in the desert.

```react
fn main() {
  let water_still = true;
  let sandworm_nearby = false;

  // Go on if there's water and no sandworm
  if water_still && !sandworm_nearby {
    println!("Paul can refill his water.");
  } else if sandworm_nearby {
    println!("Danger! A sandworm is near.");
  } else {
    println!("No water source found.");
  }
}
```

In this snippet, `if` and `else if` statements help Paul decide his course of action based on the conditions he encounters.

### 2\. While Loops

The ongoing battle for control over Arrakis leads to the endless repetition found in loops.

For instance, perhaps Paul must oversee the harvesting of the spice --- a process that would simply repeat until a quota was filled:

```react
fn main() {
  let mut spice_harvested = 0;
  let spice_quota = 100;

  // Using a while loop to continue harvesting until the quota is met
  while spice_harvested < spice_quota {
    spice_harvested += 10;  // Harvest 10 units of spice
    println!("Harvested {} units of spice.", spice_harvested);
  }

  println!("Spice quota met!");
}
```

Here, the `while` loop allows Paul to continue the spice harvest until the desired amount is achieved.

### 3\. For Loops and Iterators

Like the Fremen traversing predetermined paths across the desert, `for` loops in Rust iterate over a range or collection.

Paul decides to train a group of Fremen warriors.

```react
fn main() {
  let fremen_warriors = ["Stilgar", "Chani", "Duncan Idaho"];

  // Using a for loop to train each warrior
  for warrior in fremen_warriors.iter() {
    println!("{} is training.", warrior);
  }

  println!("All Fremen warriors have completed their training!");
}
```

Notice that here the `for` loop iterates over the array of Fremen warriors. Each iteration means there is a training session for a warrior.

### 4\. Nested Loops

Nested loops are like the layered strategies in the politics of Dune.

Think of Paul's political situation in Arrakeen.

```react
fn main() {
  let houses = ["Atreides", "Harkonnen", "Corrino"];
  let agendas = ["Trade", "War", "Espionage"];

  // Using nested loops to understand each House's agenda
  for house in houses.iter() {
    for agenda in agendas.iter() {
      println!("House {} and their stance on {}.", house, agenda);
    }
  }
}
```

Here, the nested `for` loops help Paul systematically assess each House's stance on different agendas.

### 5\. Controlling Loops with `Continue`

Just as Paul must sometimes make abrupt decisions to either continue his journey or retreat, loop controls in Rust alter the flow of loops.

Paul is searching for a hidden water cache in the desert:

```react
fn main() {
  let mut distance_traveled = 0;

  while distance_traveled < 10 {
    distance_traveled += 1;

    if distance_traveled % 2 == 0 {
      println!("Continuing search...");
      continue; // Skipping the rest of this iteration
    }

    println!("Searching at distance: {}", distance_traveled);
  }

  println!("Water cache found!");
}
```

In this snippet, `continue` skips to the next iteration under certain conditions.

### 6\. Infinite Loops and Escape Conditions with Break

Like the enduring struggle for Arrakis, infinite loops run indefinitely unless an escape condition is met.

Paul sets up an infinite surveillance loop to guard against sandworm attacks:

```react
fn main() {
  loop {
    // Check for sandworm activity
    if check_for_sandworms() {
      println!("Sandworm detected! Initiating defense protocols.");
      break; // Break out of the loop if a sandworm is detected
    } else {
      println!("No sandworm activity. Continuing surveillance.");
    }
  }
}

// Dummy function to simulate sandworm detection
fn check_for_sandworms() -> bool {
  // In a real program, this would involve some condition check
  // For illustration, it randomly returns true or false
  rand::random()
}
```

Here, an infinite `loop` runs surveillance until `sandworm_alert` becomes true, at which point the loop is exited using `break`.

### 7\. Pattern Matching with `match` Statements

In the intricate world of Dune, the complex societal structures and allegiances require pretty good adaptability, which is where Rust's `match` statements come in handy.

Imagine Paul analyzing the loyalties of various factions on Arrakis:

```react
fn main() {
    let faction = "Fremen";

    let message = match faction {
        "Fremen" => "Allies in the desert.",
        "Harkonnen" => "Enemies of the Atreides.",
        "Bene Gesserit" => "Unpredictable allies or foes.",
        _ => "Unknown faction.",
    };

    println!("The {} are: {}", faction, message);
}
```

In this snippet, the `match` statement allows for complex decision-making based on the value of `faction`, which gives us the power to handle multiple conditions with precision.

### 8\. Error Handling with `Result` Type

The unforgiving landscape of Arrakis demands that Paul anticipates and manages risks, mirroring the way Rust programmers handle potential errors using `Result` type.

Consider Paul sending spies to gather intelligence, where the mission could fail:

```react
fn main() {
    let spy_report = gather_intelligence();

    match spy_report {
        Ok(info) => println!("Intelligence gathered: {}", info),
        Err(error) => println!("Mission failed: {}", error),
    }
}

// Dummy function to simulate gathering intelligence
fn gather_intelligence() -> Result<String, String> {
    // Simulation of a situation that could either succeed or fail
    if rand::random() {
        Ok("Spice production levels are normal.".to_string())
    } else {
        Err("Spy captured by Harkonnen.".to_string())
    }
}
```

Here, the `Result` type is used to handle the possibility of failure in a controlled way, showing how Rust encourages explicit error handling to build reliable software.

### 9\. Absence of Value with `Option` Type

The quest for the elusive spice melange on Arrakis often leads to situations where outcomes are uncertain, much like how Rust's `Option` type is used to handle the absence of value in a safe and clear manner.

Imagine Paul conducting a search for a hidden spice cache, where the existence of the cache is not guaranteed:

```react
fn main() {
    let spice_cache_location = find_spice_cache();

    match spice_cache_location {
        Some(location) => println!("Spice cache found at: {}", location),
        None => println!("No spice cache found."),
    }
}

// Dummy function to simulate searching for a spice cache
fn find_spice_cache() -> Option<String> {
    // Simulation of a search that might or might not find a cache
    if rand::random() {
        Some("eastern dune sea".to_string())
    } else {
        None
    }
}
```

In this snippet, `Option` is used to represent the outcome of searching for the spice cache. The `Some` variant indicates the cache was found and contains its location, while `None` represents the failure to find the cache.

### 10\. Using `if let` for Concise Control Flow

In the complex and often ambiguous world of Dune, quick and decisive action based on partial information is vital for survival, much like how `if let` can be used in Rust for a more concise control flow when dealing with enums.

Imagine Paul trying to quickly assess the allegiance of a group encountered in the desert:

```react
enum Allegiance {
    Fremen,
    Harkonnen,
    Unknown,
}

fn main() {
    let encountered_group = Allegiance::Fremen;
    if let Allegiance::Fremen = encountered_group {
        println!("The encountered group are allies.");
    } else {
        println!("Proceed with caution.");
    }
}
```

This snippet shows how `if let` can simplify working with enums by focusing on only one pattern and handling it, making the code cleaner and more readable when you're interested in only one of the possible cases.

### That's it!

Hope you enjoyed these fun comparisons and got a good overview of Control Flow mechanisms in Rust. I added all code snippets to [Rust Drills](https://www.rustdrills.com/?utm_source=medium) in case you want to practice them.

If you liked this guide, please hit the clap 👏 button.

Rust on! 🦀
