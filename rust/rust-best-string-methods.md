## Rust: The Best String Methods

_Originally published on Mar 25, 2025 [here](https://medium.com/@jannden/rust-the-most-useful-string-methods-d23c5f1c04c8)._

---

Just before we start, I want to clarify two topics related to strings.

### A) Strings VS String Slices

- Strings in Rust (the `String` type) are growable, heap-allocated pieces of text that are ownable and modifiable.
- String slices (the `&str` type) are references to a part of a string (often a literal or a slice of a `String`) and are not ownable nor modifiable.

When you have a `String`, you can change its contents, but a `&str` only lets you read the text it points to.

Use `String` when you need ownership and mutability, and use `&str` when you just need to reference text without taking ownership. Keep that in mind, as you will see that many methods which modify strings in-place (like `push_str`, `insert_str`, and `replace_range`) only exist on `String`, while read-only operations (like `contains`, `find`, and `trim`) typically exist on `&str`.

### B) UTF-8 Nuances

Rust `String` type is UTF-8 encoded. Each character (especially if it's non-ASCII) can be made up of multiple bytes, so you can't directly index a `String` by character without risking slicing in the middle of a multi-byte sequence. Instead, you can safely iterate over characters with methods like `.chars()`.

### C) Patterns

Throughout this article, where "pattern" is mentioned, Rust often accepts various pattern types beyond simple strings, like string literals, characters, character ranges, or closures.

With these clarifications out of the way, let's start with the essential Rust string methods.

### .contains("...")

Checks if a `String` or `&str` contains a specified substring.

```react
fn main() {
    let note = "Call the plumber, we have a leak!";
    if note.contains("Pizza") {
        println!("Charlie is interested.");
    } else {
        println!("Charlie ignores the note.");
    }
}
```

When to Use: Use `.contains` when you need to confirm the presence of a specific substring or character in your string.

### .find("...") and .rfind("...")

Search a string for a pattern, returning an `Option<usize>` indicating the start index of the first (`.find`) or last (`.rfind`) match.

```react
fn main() {
    let voicemail = "We should go to the beach. No, wait, Vegas! Vegas it is.";
    if let Some(position) = voicemail.find("Vegas") {
        println!("Charlie first mentioned Vegas at position {}", position);
    } else {
        println!("No mention of Vegas found.");
    }
}
```

When to Use: Use `.find` or `.rfind` when you need to locate a specific substring's position, possibly to slice or modify the string further.

### .trim(), .trim_start(), .trim_end()

Remove whitespace from the start and/or end of a string slice.

- `.trim()` removes whitespace at both ends.
- `.trim_start()` removes whitespace at the beginning.
- `.trim_end()` removes whitespace at the end.

```react
fn main() {
    let dirty_note = "    Find my phone please!    ";
    let clean_note = dirty_note.trim();
    println!("Charlie reads: '{}'", clean_note);
}
```

When to Use: Use these methods to clean up extra spaces when parsing user input or working with data containing stray whitespace.

### .trim_matches("pattern")

Removes characters matching a pattern from the start and end of a string slice.

```react
fn main() {
    let story = "***Amazing***Incredible***";
    let core_story = story.trim_matches('*');
    println!("What actually happened: {}", core_story);
    // -> What actually happened: Amazing***Incredible
}
```

When to Use: Use `.trim_matches` to strip away specific leading and trailing characters (not just whitespace) without touching the middle.

### .to_uppercase() and .to_lowercase()

Convert your string to all uppercase or all lowercase letters, respectively.

```react
fn main() {
    let calm_note = "Please clean your room";
    let emphasized_note = calm_note.to_uppercase();
    println!("Alan: '{}'", emphasized_note);

    let dramatic_note = "This is UNACCEPTABLE!";
    let cool_note = dramatic_note.to_lowercase();
    println!("Charlie: '{}'", cool_note);
}
```

When to Use: Use these methods to simply change the capitalization of letters.

### .split("pattern") and .split_whitespace()

Break up a string into an iterator of substrings based on a delimiter.

- `.split("pattern")` uses a specified pattern (like a comma, space, or other character).
- `.split_whitespace()` uses any whitespace as a delimiter.

```react
fn main() {
    let chores = "dishes laundry vacuum";
    for chore in chores.split_whitespace() {
        println!("Jake needs to: {}", chore);
    }
}
```

When to Use: Use `.split` or `.split_whitespace` to separate text into smaller pieces---for example, parsing CSV lines or user input commands.

### .join("separator")

Combines an iterator of string slices into a single `String`, inserting a specified separator between elements.

```react
fn main() {
    let activities = ["movies", "board games", "pizza"];
    let plan = activities.join(" & ");
    println!("The Harper family plans: {}", plan);
}
```

When to Use: Use `.join` when you need to merge multiple string slices into one cohesive string with a chosen delimiter.

### .starts_with("...") and .ends_with("...")

Check whether a string begins or ends with a specified pattern.

```react
fn main() {
    let song = "Cry Me a River";
    if song.starts_with("Cry") {
        println!("Charlie is in one of his moods.");
    } else if song.ends_with("Waffles") {
        println!("Charlie's just enjoying his classics.");
    }
}
```

When to Use: Use `.starts_with` or `.ends_with` for pattern matching in routing, validation, or simply tailoring logic based on prefixes/suffixes.

### .chars().rev()

Reverses the sequence of characters in a string by first converting it to an iterator of chars, then calling `.rev()`, and finally collecting back into a new `String`.

```react
fn main() {
    let to_do_list = "groceries laundry dishes";
    let reversed_list: String = to_do_list.chars().rev().collect();
    println!("Charlie's reversed to-do list: {}", reversed_list);
}
```

When to Use: Use this pattern when you need to invert the order of characters, keeping in mind Rust's UTF-8 intricacies.

### .chars().nth(index)

Retrieves the character at a specific index (0-based) in a string, returning an `Option<char>`.

```react
fn main() {
    let emojies = "🐨🐍🦀";
    if let Some(nth_char) = emojies.chars().nth(2) {
        println!("Jake uses the {} emoji.", nth_char);
    } else {
        println!("Jake can't use an emoji.");
    }
}
```

When to Use: Use `.chars().nth` to safely access individual characters, acknowledging that multi-byte characters. Note that it's not very efficient (especially for repeated access), so you might consider for example loading the characters into a vector.

### .chars().count()

Returns the number of Unicode scalar values (characters) in the string.

```react
fn main() {
    let story = "And then, in the dark, something touched my foot!";
    let length = story.chars().count();
    println!("Charlie's story was {} characters long", length);
}
```

When to Use: Use `.chars().count()` when you need the character count for display, validation, or analysis---especially mindful of multi-byte characters.

### .replace("pattern", "replacement")

Creates a new `String` by replacing all occurrences of a given pattern with another substring.

```react
fn main() {
    let message = "Meet me at the fancy restaurant.";
    let updated_message = message.replace("fancy restaurant", "beach");
    println!("Charlie's new message: {}", updated_message);
}
```

When to Use: Use `.replace` when you want to substitute text in a string without mutating the original.

### .replace_range(range, "replacement")

Modifies a `String` in-place by replacing the specified character range with a new substring.

```react
fn main() {
    let mut grocery_list = String::from("Apples, Bread, Chicken, Salad");
    grocery_list.replace_range(15..=21, "Fish");
    println!("Updated grocery list: {}", grocery_list);
}
```

When to Use: Use `.replace_range` for precise, in-place modifications when you know exactly which slice of characters you need to replace.

### .push('x')

Adds a single `char` to the end of a `String`. *Note: only a character can be pushed this way, not a string slice. A character is enclosed in single quotes.*

```react
fn main() {
    let mut song = String::from("This is the song that never ends");
    song.push('!');
    println!("Jake's song: {}", song);
}
```

When to Use: Use `.push` when you only need to add a single character to your string.

### .push_str("...")

Appends a string slice onto the end of an existing `String`. *This is for more than a single character --- compare with the *`*.push('x')*`_ above._

```react
fn main() {
    let mut sandwich = String::from("Ham and Cheese");
    sandwich.push_str(" with Avocado");
    println!("Jake's improved sandwich: {}", sandwich);
}
```

When to Use: Use `.push_str` when you need to add extra text to an existing `String` without creating a new one.

### .insert_str(position, "substring")

Inserts a substring at a given byte index in a `String`.

```react
fn main() {
    let mut email = String::from("I will come on Friday.");
    email.insert_str(21, " at 3 PM");
    println!("Alan's updated email: {}", email);
}
```

When to Use: Use `.insert_str` when you need to add content at a precise location in your string, and you know the correct byte offset.

### .to_owned() and .to_string()

Convert other types into an owned `String`.

- .to_owned() creates an owned `String` from a borrowed type---like a `&str`---by leveraging the `ToOwned` trait.
- .to_string() uses the `ToString` trait, which for `&str` also yields a `String` with the same contents.

```react
fn main() {
    let hour_str: &str = "12";
    let owned_hour = hour_str.to_owned();
    println!("Using `.to_owned()`: {:?}", owned_hour);

    let hour_num: u8 = 12;
    let owned_hour = hour_num.to_string();
    println!("Using `.to_string()`: {:?}", owned_hour);
}
```

When to Use: Use either `.to_owned()` or `.to_string()` when you need a standalone `String`.

_Note: While both methods produce the same result for string slices, Clippy prefers _`*.to_owned()*`_ when converting _`*&str*`_ to _`*String*`_._

### .escape_debug()

Escapes special characters in a string, often used for debugging output.

```react
fn main() {
    let error_message = "File not found: 'homework.rs'";
    let escaped_message = error_message.escape_debug();
    println!("Jake's error message: {}", escaped_message);
}
```

When to Use: Use `.escape_debug()` to safely display non-printable or special characters, making logs or error reports more readable.

### format!("... {}", var)

Constructs a new string using interpolation, similar to `println!`, but returns a `String` instead of printing to stdout.

```react
fn main() {
    let her_name = "Linda";
    let my_excuse = "unexpected visit from my brother";
    let apology = format!(
        "Hi {}, I'm really sorry about last night, there was an {}.",
        her_name, my_excuse
    );
    println!("Charlie's apology: {}", apology);
}
```

When to Use: Use `format!` to build strings with variable data. It's type-safe, flexible, and avoids concatenation woes.

### .as_bytes()

Returns a byte slice (`&[u8]`) of the entire string, reflecting its UTF-8 representation.

```react
fn main() {
    let coded_message = "Come by.";
    let message_bytes: Vec<u8> = coded_message.as_bytes().to_vec();
    println!("Rose's message in bytes: {:?}", message_bytes);
}
```

When to Use: Use `.as_bytes()` when you need the raw byte representation---for example, for networking or manual parsing.

### That's it!

These were the most useful string methods in Rust.

Don't learn them by heart, just know they are at your disposal. Refer to this article anytime you need.

If you find it helpful, please click the clap 👏 button.

Rust on!
