---
transition: fade-out
layout: section
---

# 🦀Using Structs to Structure Related Data

---

# Defining and Instantiating Structs
- Structs are similar to tuples
- Both hold multiple related values.
- Like tuples, the pieces of a struct can be different types.
- Unlike tuples, structs have names for each piece of data.
- Adding these names means that structs are more flexible than tuples.
- you don’t have to rely on the order of the data to specify or access the values of an instance.

---
layout: two-cols
---

## Defining a Struct
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```
<br>

## Instantiating a Struct

```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
}
```

::right::

<img src="/img/5-using-structs/1-defining-and-instantiating-structs/stack-1.png" alt="Ownership Figure 4.1" width="400px" style="margin: 250px auto; display: block;" />

---

## Accessing Struct Fields
- To get a specific value from a struct, we use dot notation.
-  If the instance is mutable, we can change a value by using the dot notation and assigning into a particular field.

```rust{all|9}
fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

ℹ️Note that the entire instance must be mutable; Rust doesn’t allow us to mark only certain fields as mutable.

---

## Using the Field Init Shorthand
If a field in the struct has the same name as a variable, we can use the shorthand syntax to create an instance of the struct.

```rust{all|1,4-5}
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

---

# Struct Update Syntax
It’s often useful to create a new instance of a struct that includes most of the values from another instance, but changes some. 

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

- It creates an instance in `user2` that has a different value for `email` but has the same values for the `username`, `active`, and `sign_in_count` fields from `user1`
- `..user1` must come last.
- After creating `user2`, `user1` is partially invalidated because the String in the `username` field of `user1` was moved into `user2`. 
- If we had only used `activate` and `sign_in_count`, then `user1` would still be fully valid after the `user2` creation.

---

# Tuple Structs
- Rust also supports structs that look similar to tuples, called tuple structs.
- Tuple structs have the added meaning the <span color="orange">struct name</span> provides.
- But **don’t have** names associated with <span color="orange">their fields</span>.
- Useful when you want to give the whole tuple a name and make the tuple a different type from other tuples
- Or, when naming each field as in a regular struct would be verbose or redundant.

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

- `black` and `origin` values are different types because they’re instances of different tuple structs. 
- A function that takes a parameter of type `Color` cannot take a `Point` as an argument.
- You can destructure them into their individual pieces.

---

### Unit-Like Structs
- Unit-like structs are structs that don’t have any fields.
- These are called unit-like structs because they behave similarly to `()`
- Useful when you need to implement a trait for a type but don’t have any data to associate with that type.
- Defined with the `struct` keyword, followed by the name of the struct and a semicolon.

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```
More practical example:
```rust
struct Meter;
struct Second;

fn calculate_speed(distance: (f64, Meter), time: (f64, Second)) -> f64 {
    distance.0 / time.0
}

let distance = (10.0, Meter);
let time = (5.0, Second);
let speed = calculate_speed(distance, time); // Correct!
// calculate_speed(time, distance); // Compiler error!

```

---

# Borrowing Fields of a Struct
Rust’s borrow checker will track ownership permissions at both the struct-level and field-level. 

For example, if we borrow a field `x` of a `Point` structure, then both `p` and `p.x` temporarily lose their permissions (but not `p.y`):

<img src="/img/5-using-structs/1-defining-and-instantiating-structs/code-1.png" alt="Ownership Figure 4.1" width="400px" style="margin: 0 auto; display: block;" />