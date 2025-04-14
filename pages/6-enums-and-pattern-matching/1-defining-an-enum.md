---
transition: fade-out
layout: section
---

# 🦀Defining an Enum

---

## Structs vs. Enums

- Structs give you a way of grouping together related fields and data, like a `Rectangle`
- Enums give you a way of saying a value is one of a possible set of values.

```rust{1-4|7-9|11,12|14-16}
enum IpAddrKind {
    V4,
    V6,
}
//...

// We can create instances of each of the two variants of IpAddrKind like this
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;

// We can define a function that takes an IpAddrKind as a parameter
fn route(ip_kind: IpAddrKind) {}

// We can call the function with either of the two variants:
    route(IpAddrKind::V4);
    route(IpAddrKind::V6);

```

---

## Enums with Data
Using enums has even more advantages. 

We want to store the IP address as well. You might be tempted to tackle this problem with structs:

```rust
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

---

- However, representing the same concept using just an enum is more concise. 
- We can put data directly into each enum variant.

```rust{1-4|6-9|all}
enum IpAddr {
    V4(String),
    V6(String),
}

// The name of each enum variant that we define also becomes a function that constructs an instance of the enum.
let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```
ℹ️There’s another advantage to using an enum rather than a struct.
 Each variant can have different types and amounts of associated data.

```rust{2|all}
 enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```
---

# IP Address In Standard Library
Let’s look at how the standard library defines IpAddr:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```
This code illustrates that you can put any kind of data inside an enum variant: 

- strings
- numeric types
- structs
- other enums

<!-- Note that even though the standard library contains a definition for IpAddr, we can still create and use our own definition without conflict because we haven’t brought the standard library’s definition into our scope. -->

---

# Another Example
```rust{all|1-6|7-11}
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
// We can create instances of each of the four variants of Message like this
let m = Message::Write(String::from("hello"));
let m = Message::ChangeColor(255, 0, 0);
let m = Message::Move { x: 10, y: 20 };
let m = Message::Quit;
```
There is one more similarity between enums and structs: 

- Just as we’re able to define methods on structs using `impl`, we’re also able to define methods on enums.

```rust
impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

---

# Option Enum
- The `Option` enum is a powerful and flexible way to express the idea of an optional value.
- Encodes the very common scenario in which a value could be something or it could be nothing.
    - If you request the first item in a non-empty list, you would get a value.
    - If you request the first item in an empty list, you would get nothing. 
- Rust doesn’t have the null feature that many other languages have.

🦀Expressing this concept in terms of the type system means the <span color="orange">compiler</span> can check whether you’ve handled <span color="yellow">all the cases</span> you should be handling.

☠️In his 2009 presentation “Null References: The Billion Dollar Mistake,” Tony Hoare, the inventor of null, has this to say:
> I call it my billion-dollar mistake. At that time, I was designing the first comprehensive type system for references in an object-oriented language. My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn’t resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

---

- The concept that null is trying to express is still a useful one:
    - a null is a value that is currently invalid or absent for some reason.
- The problem isn’t really with the concept but with the particular implementation.
- 🦀 Rust solution is, `Option<T>` enum.
- The `Option<T>` enum is so useful that it’s even included in the prelude.
- Its variants are also included in the prelude: 
    - you can use `Some` and `None` directly without the `Option::` prefix.

```rust{1-4|6-10|all}
enum Option<T> {
    None,
    Some(T),
}

// Example of using Option<T> enum:
let some_number = Some(5);  // Type is Option<i32>
let some_char = Some('e');  // Type is Option<char>

let absent_number: Option<i32> = None; // Type is Option<i32>

```
<!-- 
The <T> syntax is a feature of Rust we haven’t talked about yet. It’s a generic type parameter, and we’ll cover generics in more detail in Chapter 10.  -->

---

##  So is `Option<T>` better

`Option<T>` and `T` (where T can be any type) are different types, the compiler won’t let us use an `Option<T>` value as if it were definitely a valid value.

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

```sh
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
```

- This error message means that Rust doesn’t understand how to add an i8 and an `Option<i8>`, because they’re different types.
- In other words, you have to convert an `Option<T>` to a `T` before you can perform `T` operations with it.

☠️ Generally, this helps catch one of the most common issues with null: <span color="orange">assuming that something isn’t null when it actually is.</span>

--- 

## How to Use `Option<T>`
In general, in order to use an `Option<T>` value, you want to have code that will handle each variant.
- You want some code that will run only when you have a `Some(T)` value
    - This code is allowed to use the inner `T`.
- You want some other code to run only if you have a `None` value
    - That code doesn’t have a `T` value available.

🦀 This is where match expressions come in.

<!-- The Option<T> enum has a large number of methods that are useful in a variety of situations; you can check them out in its documentation. Becoming familiar with the methods on Option<T> will be extremely useful in your journey with Rust. -->