---
transition: fade-out
layout: section
---

# 🦀The match Control Flow Construct

---

# Match
- extremely powerful control flow.
- Allows you to compare a value against a series of patterns,
- and then execute code based on which pattern matches. 
- Patterns can be made up of 
    - literal values
    - variable names
    - wildcards
    - and many other things
- The power of match comes from the expressiveness of the patterns,
- and the fact that the compiler confirms that all possible cases are handled.
- Think of a match expression as being like a coin-sorting machine:
    - Coins slide down a track with variously sized holes along it,
    - Each coin falls through the first hole it encounters that it fits into.

---

## Example
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

- First we list the match keyword followed by an expression, which in this case is the value coin.
- Next are the match arms. An arm has two parts: 
    - A pattern
    - Some code
- Each arm is separated from the next with a comma.

---

## Match is an expression
- The code associated with each arm is an expression
- The resultant value of the expression in the matching arm is the value that gets returned for the entire match expression.

## Some more details
- We don’t typically use curly brackets if the match arm code is short.
- If you want to run multiple lines of code in a match arm, you must use curly brackets, and the comma following the arm is then optional.

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```
---

## Patterns That Bind to Values
Another useful feature of match arms is that they can bind to the parts of the values that match the pattern.
```rust
enum UsState {
    Alabama,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {state:?}!");
            25
        }
    }
}
```

---

## Matching with `Option<T>`

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```

---

## Matches Are Exhaustive
🦀The arms’ patterns must cover all possibilities.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }
```
```sh
$ cargo run
error[E0004]: non-exhaustive patterns: `None` not covered
```
<br>

☠️Especially in the case of `Option<T>`, when Rust prevents us from forgetting to explicitly handle the None case, it protects us from assuming that we have a value when we might have null, thus making the billion-dollar mistake discussed earlier impossible.

---

## Catch-all Patterns and the _ Placeholder
Using enums, we can also take special actions for a few particular values, but for all other values take one default action.
```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    other => move_player(other),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
```
🦀This catch-all pattern meets the requirement that match must be exhaustive.
⚠️We have to put the catch-all arm last because the patterns are evaluated in order.
```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),  // _ is a special pattern that matches any value and does not bind to that value.
    }
```

---

## How Matches Interact with Ownership
If an enum contains non-copyable data like a String, then you should be careful with whether a match will move or borrow that data.

```rust
let opt: Option<String> = 
    Some(String::from("Hello world"));

match opt {
    Some(_) => println!("Some!"),   // Will not move the value
    None => println!("None!")
};

println!("{:?}", opt);
```
The following code will NOT compile:
```rust
let opt: Option<String> = 
    Some(String::from("Hello world"));
match opt {
    Some(s) => println!("{:?}", s),  // Will move the value
    None => println!("None!")
};
println!("{:?}", opt); // Error: value moved
```

---

If we want to peek into opt without moving its contents, the idiomatic solution is to match on a reference

```rust
fn main() {
let opt: Option<String> = 
    Some(String::from("Hello world"));

// opt became &opt
match &opt {
    Some(s) => println!("Some: {}", s),
    None => println!("None!")
};

println!("{:?}", opt);
}
```

🦀Rust will “push down” the reference from the outer enum, `&Option<String>`, to the inner field, `&String`. 

---
layout: section
---
# 🦀Concise Control Flow with `if let`
---

## if let
The if let syntax lets you combine if and let into a less verbose way to handle values that match one pattern while ignoring the rest.

```rust
let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {max}"),
        _ => (),
    }
```
We don’t want to do anything with the None value. To satisfy the match expression, we have to add _ => () after processing just one variant.

**Instead**, we can use `if let`:
```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {max}");
}
```
🦀Using if let means less typing, less indentation, and less boilerplate code.
<!-- 
The syntax if let takes a pattern and an expression separated by an equal sign. It works the same way as a match, where the expression is given to the match and the pattern is its first arm. In this case, the pattern is Some(max), and the max binds to the value inside the Some. We can then use max in the body of the if let block in the same way we used max in the corresponding match arm. The code in the if let block isn’t run if the value doesn’t match the pattern. -->

---

## `if let` with `else`
- You can also add an else block to the if let expression.
- The else block is run if the value doesn’t match the pattern.
```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {state:?}!"),
    _ => count += 1,
}
```

We can use `if let` and `else` instead:

```rust
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {state:?}!");
    } else {
        count += 1;
    }
```