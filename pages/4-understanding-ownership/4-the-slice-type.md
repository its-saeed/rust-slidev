---
transition: fade-out
layout: section
---

# 🦀 The Slice Type
---

# The Slice Type

* Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection.
* A slice is a kind of reference, so it is a non-owning pointer.

<img src="/img//4-understanding-ownership/4-the-slice-type/slice.png" alt="Ownership Figure 4.1" width="400px" style="margin: 0 auto; display: block;" />

---

# Why slices are useful
Write a function that takes a string of words separated by spaces and returns the first word it finds in that string.
If no space is found, return the whole string.

```rust
fn first_word(s: &String) -> ?
```

- The `first_word` function has a `&String` as a parameter. We don’t want ownership of the string, so this is fine. 
- But what should we return? We don’t really have a way to talk about part of a string.


---

# Solution #1

We could return the index of the end of the word, indicated by a space. 

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

⚠️We’re returning a `usize` on its own, but it’s only a meaningful number in the context of the `&String`.

In other words, because it’s a <span color="red">separate</span> value from the `String`, there’s no guarantee that it will still be <span color="red">valid</span> in the future.

☠️If the string gets cleared or goes out of scope, the index will be invalid but the program will still compile.

<!-- 
Situations gets even worse when we try to return the first word as a slice.

We have three unrelated variables floating around that need to be kept in sync.

fn second_word(s: &String) -> (usize, usize) { -->

---

# String Slices
A string slice is a reference to part of a `String`, and it looks like this

```rust
let s = String::from("hello world");
let hello = &s[0..5]; // hello
let world = &s[6..11]; // world
```

- The first number is the starting index, and the second number is one past the end of the slice.
- Slices are special kinds of references because they are <span color="green">“fat”</span> pointers, or pointers with metadata.
    -  The metadata is the **length** of the slice.

<img src="/img/4-understanding-ownership/4-the-slice-type/stack-1.png" alt="Ownership Figure 4.1" width="400px" style="margin: 0 auto; display: block;" />

---

## Slices are references
Because slices are references, they also change the permissions on referenced data.

<img src="/img/4-understanding-ownership/4-the-slice-type/code-1.png" alt="Ownership Figure 4.1" width="400px" style="margin: 0 auto; display: block;" />

---

## Range syntax
```rust{all|3-4|6-8|10-11}
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];

let len = s.len();
let slice = &s[3..len];
let slice = &s[3..];

let slice = &s[0..len];
let slice = &s[..];
```

⚠️String slice range indices must occur at valid UTF-8 character boundaries. If you attempt to create a string slice in the middle of a multibyte character, your program will exit with an error.

 <!-- For the purposes of introducing string slices, we are assuming ASCII only in this section; a more thorough discussion of UTF-8 handling is in the “Storing UTF-8 Encoded Text with Strings” section of Chapter 8. -->

---

# Solution #2

We can return a slice of the string instead of an index.

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

🎉Now when we call `first_word`, we get back a single value that is tied to the underlying data. 

Returning a slice would also work for a `second_word` function:


```rust
fn second_word(s: &String) -> &str {
```

---

# Remember, Slices are references
Everything we learned about references applies to slices as well.

🦀Rust disallows the mutable reference in clear and the immutable reference in word from existing at the same time, and compilation fails.

<br>
<img src="/img/4-understanding-ownership/4-the-slice-type/code-2.png" alt="Ownership Figure 4.1" width="500px" style="margin: 0 auto; display: block;" />

---

# String Literals Are Slices

```rust
let s: &str = "hello world";
```

- String literals are stored in the binary of the program, and they are immutable.
- The type of `s` here is `&str`: it’s a slice pointing to that specific point of the binary.
- This is also why string literals are immutable; `&str` is an immutable reference.

# String Slices as Parameters
Knowing that you can take slices of literals and String values leads us to one more improvement on `first_word`, and that’s its signature:
```rust 
fn first_word(s: &String) -> &str {
```

A more experienced Rustacean would write the signature like this:

```rust
fn first_word(s: &str) -> &str {
```

If we have a string slice, we can pass that directly. If we have a String, we can pass a slice of the String or a reference to the String

---

# Other Slices

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

- Slices can be used with any kind of contiguous sequence, including arrays and vectors.
- The slice type for arrays is `&[T]`, where `T` is the type of the elements in the array.
- Here, this slice has the type `&[i32]`