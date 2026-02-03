## Rust Strings: A Fun Guide with Examples

_Originally published on Mar 2, 2024 [here](https://medium.com/rustaceans/rust-strings-a-fun-guide-with-examples-e412ff963465)._

---

Welcome to the bewitched world of Rust. I will explain all about Strings, drawing parallels with iconic characters from The Witcher to make the journey fun and practical at the same time.

You can practice the code snippets from this article on [Rust Drills](https://www.rustdrills.com/?utm_source=medium).

### 1\. Creating Strings

In Rust, strings are a bit like the diverse languages spoken across the Continent in The Witcher. They come in various forms, each with unique characteristics.

Imagine creating a lexicon for Geralt, containing various terms and phrases he encounters in his journey.

```react
fn main() {
  // Create an empty string
  let mut lexicon = String::new();

  // The string is empty
  println!("Opening an empty lexicon {}", lexicon);

  // Convert a string literal to a String
  let phrase = "Witcher";
  lexicon = phrase.to_string();

  // Print the String
  println!("Geralt's profession: {}", lexicon);

  // Print the length of the String
  println!("Number of characters: {}", lexicon.len());

  // Define a String using `from` method
  let bestiary = String::from("Griffin");

  // Print the String
  println!("Monster in the Bestiary: {}", bestiary);

  // Print the length of the String
  println!("Bestiary length: {}", bestiary.len());
}
```

As you can see, we used three ways to create a `String`:

- `String::new()` to create an empty `String`,
- `.to_string()` to convert a string slice to a `String`,
- `String::from()` to create a `String` from an un-named string slice (string literal).

### 2\. String Types

There are three crucial terms we have used in the previous section: `String`, `string slice`, and `string literal`. That might be a bit confusing in the beginning. Let's explore their differences.

### String

In Rust, when we say string, we usually mean the type `String`. It is like a story told around a campfire in The Witcher's world. It's dynamic, can change over time, and grow as more details are added. Just like how Geralt can add more tales to his adventures, we can add more text to a `String`.

```react
fn main() {
    let mut story = String::from("Geralt rode through the dark forest");

    // Geralt's story is growing as he encounters a Leshen
    story.push_str(", facing the eerie Leshen");
    println!("Campfire story: {}", story); // Prints the growing story
}
```

In this example, `story` is a mutable `String` type, which means it can be modified after it has been created. We start with Geralt riding through a forest and then add another sentence about an encounter, making the story longer.

### String Slice

A string slice in Rust is of the type `&str` and it is like a part of the campfire story. It's a reference to a section of the story, not the whole thing. Imagine if you only want to talk about the part where Geralt faces the Leshen, not the entire journey.

```react
fn main() {
    let full_story = String::from(
        "Geralt rode through the dark forest, facing the eerie Leshen",
    );

    // Slicing the story from the 38th character to the end
    let part_of_story = &full_story[37..];

    // Prints the slice of the story
    println!("Part of the story: {}", part_of_story);
}
```

Here, `part_of_story` is a slice of `full_story`. It doesn't contain the whole story but just a fragment, specifically the encounter with the Leshen. Unlike the whole story (`String`), this is just a reference to a part of the original --- and as such cannot be changed.

### String Literal

Finally, a string literal in Rust is also of the type `&str`, but it is more like a story that has been written down into a chronicle in The Witcher's world. It is fixed and cannot be changed --- immutable and eternal.

```react
fn main() {
    // This story is set in stone
    let written_story = "Geralt's adventures are known across the lands.";

    // Prints the string literal
    println!("Written story: {}", written_story);
}
```

This `written_story` is a string literal, which means it's a fixed-size string and lives for the entire duration of the program. It's like a tale that has been engraved into history, unchanging and always accessible.

### 3\. String Methods

So we have established that only the type `String` can be changed. Let's now look at the most important options we have for changing it.

Imagine that Geralt wants to update his lexicon with more details. Notice which String methods he might use:

```react
fn main() {
  let mut lexicon = String::from("Witcher, ");

  // Check if the lexicon contains a specific word
  if lexicon.contains("Griffin") {
    println!("Griffin is already in the lexicon.");
  } else {
    lexicon.push_str("the Griffin slayer, ");
  }

  // Replace a word in the lexicon
  lexicon = lexicon.replace("Witcher", "Geralt of Rivia");

  // Trim any extra spaces
  let trimmed_lexicon = lexicon.trim();

  // Check if the lexicon starts with a specific word
  if trimmed_lexicon.starts_with("Geralt") {
    println!("The lexicon starts with Geralt's name.");
  }

  // Adding the details of Geralt's adventures
  lexicon.push_str("is now resting in an inn.");

  // Print the updated lexicon
  println!("Updated Lexicon: {}", lexicon);
}
```

We can update the lexicon using methods like `contains`, `push_str`, `replace`, and `trim`. Each method is a tool in your arsenal, much like swords and potions for Geralt.

### 4\. Methods for updating Strings

Like Geralt upgrades his gear and potions, you can add, modify, or remove characters from strings.

Using those options, Geralt decides to update his potion recipe.

```react
fn main() {
    let mut recipe = String::from("Swallow: ");

    // Adding ingredients to the potion
    recipe.push_str("Celandine, ");
    recipe.push_str("Drowner Brain, ");
    recipe.push('W'); // Adding a single character, notice the single quotes

    // Removing the last ingredient (mistakenly added)
    recipe.pop();

    // Combining with another potion recipe
    let enhanced_recipe = format!("{} and Thunderbolt", recipe.trim());

    // Print the final potion recipe
    println!("Updated Potion Recipe: {}", enhanced_recipe);
}
```

Geralt uses `push_str`, `push`, and `pop` to craft and update his potion recipe. The `format!` macro then combines two recipes into one, like mixing ingredients to enhance a potion's effect.

### 5\. Iterating Over Strings

Iterating over strings in Rust can be compared to Geralt exploring different regions of the Continent.

Imagine Geralt coming across a mysterious inscription. He needs to analyze it character by character. This is how we might approach it in Rust using a couple of handy methods:

```react
fn main() {
  let inscription = "Kaer Morhen: Witcher School";

  // Splitting the inscription into words
  for word in inscription.split_whitespace() {
    println!("Word: {}", word);
  }

  // Splitting based on a specific character
  for part in inscription.split(':') {
    println!("Part: {}", part.trim());
  }

  // Iterating over each character
  for ch in inscription.chars() {
    println!("Character: {}", ch);
  }
}
```

Here, `split_whitespace`, `split`, and `chars` methods help Geralt break down the inscription into understandable parts. These methods work on both types of strings (`String` and `&str`).

### 6\. Converting Between Strings and Other Types

Just as Geralt converts resources found in the Continent into useful items and potions, Rust allows for the conversion between strings and other types.

For instance, Geralt might find an ancient scroll with numbers that need to be used as text or vice versa.

```react
fn main() {
    let numerical_string = String::from("2024");

    // Converting string to i32
    let year: i32 = numerical_string.parse().expect("Not a number!");
    println!("The year in the scroll: {}", year);

    // Converting a number back to string
    let new_string = year.to_string();
    println!("The year as a string: {}", new_string);

    // Converting a boolean to a string
    let is_witcher = true;
    let status = is_witcher.to_string();
    println!("Is Geralt a Witcher? {}", status);
}
```

Rust can convert different data types with the helpful `parse` method and still keep flexible error handling.

### 7\. Advanced String Manipulation

Advanced string manipulation in Rust can be compared to Geralt's use of signs, tactics, and strategies in combat.

Suppose Geralt is crafting a coded message that needs to be encrypted or formatted in a specific way.

```react
fn main() {
    let secret_message = String::from("Witcher of Rivia");

    // Reversing a string to encode a secret message
    let reversed_message: String = secret_message.chars().rev().collect();
    println!("Reversed Message: {}", reversed_message);

    // Capitalizing each word for a code
    let mut capitalized_words = secret_message
        .split_whitespace()
        .map(|word| {
            let mut chars = word.chars();
            match chars.next() {
                None => String::new(),
                Some(first) => first.to_uppercase().collect::<String>() + chars.as_str(),
            }
        })
        .collect::<Vec<String>>()
        .join(" ");
    println!("Capitalized Message: {}", capitalized_words);

    // Inserting a character at a specific index
    capitalized_words.insert(7, ':'); // Splitting 'Witcher' and 'of'
    println!("Modified Message: {}", capitalized_words);
}
```

In this last example, you can get a small taste of the depth of string handling in Rust using iterators and closures. We manipulate the `secret_message` through reversing, capitalizing, and tampering.

### That's it!

Hope you enjoyed these fun comparisons and got a good overview of Strings in Rust. I added all code snippets to [Rust Drills](https://www.rustdrills.com/?utm_source=medium) in case you want to practice them.

If you liked this guide, please hit the clap 👏 button.

Rust on!

_PS. \*\*You can find more fun Rust guides _[_on this list_](https://medium.com/@jannden/list/fun-rust-guides-b69e0e65a0b0)_._
