## Rust Structs: A Fun Guide with Examples

_Originally published on Feb 26, 2024 [here](https://medium.com/rustaceans/rust-structs-a-fun-guide-with-examples-c2851dfb96c6)._

---

Welcome to the royal world of Rust. I will explain all about Structs, drawing parallels with Game of Thrones to make the journey fun while keeping it practical at the same time.

You can practice the code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium).

### 1\. The Basics of Structs

When we want to group related values, structs come really handy.

For example, we can define each great house in the *Game of Thrones* with some specific attributes, such as name and region.

```react
struct House {
  name: String,
  region: String,
}

fn main() {
  let house_stark = House {
    name: String::from("Stark"),
    region: String::from("The North"),
  };

  println!("House {} of {}", house_stark.name, house_stark.region);
}
```

We defined a noble house in Westeros as a `House` struct with fields for `name` and `region`.

### 2\. Methods in Structs

Each house has a specific way it behaves in certain situations. Let's say that each house can rally its bannermen for aid.

We will implement this action as a method of the `House` struct while adding the number of bannerman as another field in the `House` struct.

```react
struct House {
  name: String,
  region: String,
  bannermen: u32,
}

impl House {
  fn rally_bannermen(&mut self, additional: u32) {
    self.bannermen += additional;
    println!("House {} now has {} bannermen.", self.name, self.bannermen);
  }
}

fn main() {
  let mut house_arryn = House {
    name: String::from("Arryn"),
    region: String::from("The Vale"),
    bannermen: 500,
  };

  house_arryn.rally_bannermen(300);
}
```

In the example above, the `House Arryn` can use the `rally_bannermen` method to increase the number of its bannermen.

### 3\. Associated Functions

Associated functions represent the cold face of a house shown to strangers. They are related to a struct but don't use information of the struct and neither update the struct.

Creating a new house in Westeros is a significant event and can be represented as an associated function. Notice that the `found_new_house` function has neither the `&self` nor `&mut self` parameter --- so it's not a method but just an associated function.

```react
struct House {
  name: String,
  region: String,
  bannermen: u32,
}

impl House {
  fn found_new_house(name: String, region: String) -> House {
    House {
      name,
      region,
      bannermen: 100, // Default number of bannermen for a new house
    }
  }
}

fn main() {
  let house_tarly = House::found_new_house(
    String::from("Tarly"),
    String::from("The Reach"),
  );
  println!("House {} of {} has been founded.", house_tarly.name, house_tarly.region);
}
```

The function `found_new_house` is associated with the `House` struct now. It creates a new house with a default bannermen number, but it neither reads any info from a `House` and neither updates an existing `House`.

Core differences between struct methods and associated functions:

- Associated functions don't receive the `&self` / `&mut self` argument, so they don't have access to the struct's properties, but struct methods do.
- Associated functions are called with the `::` notation as opposed to struct methods which are called with the `.` notation.

### 4\. Structs with Enums

Just as houses have complex structures, so too can Rust structs incorporate enums for added flexibility. Enums in Rust allow us to define a type by enumerating its possible variants. This will be useful when dealing with properties that can have one out of a set of possible values.

Let's introduce an enum to represent the current state of a house in Westeros, such as at peace or war. This enum, named `HouseState`, can then be incorporated into our `House` struct.

```react
#[derive(Debug)]
enum HouseState {
  AtPeace,
  AtWar,
}

struct House {
  name: String,
  region: String,
  bannermen: u32,
  state: HouseState,
}

impl House {
  fn change_state(&mut self, new_state: HouseState) {
      self.state = new_state;
  }
}

fn main() {
  let mut house_lannister = House {
      name: String::from("Lannister"),
      region: String::from("The Westerlands"),
      bannermen: 1000,
      state: HouseState::AtPeace,
  };

  house_lannister.change_state(HouseState::AtWar);

  println!("House {} is now {:?}", house_lannister.name, house_lannister.state);
}
```

The first line, `#[derive(Debug)]`, is there just so that Rust knows how to print the enum value with `println!` at the end of the `main` function.

The important thing here is that we defined the `HouseState` enum with two variants. Each `House` struct now also includes a `state` field of type `HouseState`. The method `change_state` allows a house to change its state, reflecting dynamic political scenarios in Westeros.

_Read more about Enums in _[_my previous article_](https://medium.com/rustaceans/rust-enums-a-fun-guide-with-examples-f3bb0f590d93)_._

### 5\. Structs with Traits

Traits in Rust are similar to interfaces in other languages. They define a set of methods that can be implemented by structs.

So whats the difference between struct methods and traits, you might ask. Well:

- struct methods are specific to a particular struct,
- traits define a set of methods that can be implemented by multiple types (including structs).

Let's define a trait `HouseTraits` that represents common actions of a noble house, such as hosting a feast or defending its lands. This trait can then be implemented by our `House` struct as well as other structs or even other types.

```react
#[derive(Debug)]
enum HouseState {
  AtPeace,
  AtWar,
}

struct House {
  name: String,
  region: String,
  bannermen: u32,
  state: HouseState,
}

trait HouseTraits  {
  fn host_feast(&self);
  fn defend_lands(&mut self, additional_bannermen: u32);
}

impl HouseTraits for House {
  fn host_feast(&self) {
      println!("House {} is hosting a grand feast!", self.name);
  }

  fn defend_lands(&mut self, additional_bannermen: u32) {
      self.bannermen += additional_bannermen;
      println!("House {} defends its lands with {} bannermen.", self.name, self.bannermen);
  }
}

fn main() {
  let mut house_baratheon = House {
      name: String::from("Baratheon"),
      region: String::from("The Stormlands"),
      bannermen: 800,
      state: HouseState::AtWar,
  };

  house_baratheon.host_feast();
  house_baratheon.defend_lands(200);
}
```

In this example, the `HouseTraits` trait includes two methods: `host_feast`, which is a read-only action, and `defend_lands`, which modifies the state of the struct. The `House` struct then implements these methods, providing specific behaviors.

### That's it!

Hope you enjoyed these fun comparisons and got a good overview of Structs in Rust. I added all code snippets to [Rust Drills](https://www.rustdrills.com/?utm_source=medium) in case you want to practice them.

If you liked this guide, please hit the clap 👏 button.

Rust on!
