---
transition: fade-out
layout: section
---

# 🦀 References and Borrowing
---

## References and Borrowing
Ownership, boxes, and moves provide a foundation for safely programming with the heap.

⚠️However, move-only APIs can be inconvenient to use.

```rust
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");
    greet(m1, m2);
    let s = format!("{} {}", m1, m2); // Error: m1 and m2 are moved
}

fn greet(g1: String, g2: String) {
    println!("{} {}!", g1, g2);
}
```
---

First solution is:
```rust
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");
    let (m1_again, m2_again) = greet(m1, m2);
    let s = format!("{} {}", m1_again, m2_again);
}

fn greet(g1: String, g2: String) -> (String, String) {
    println!("{} {}!", g1, g2);
    (g1, g2)
}
```
⚠️This style of program is quite verbose.

🦀Rust provides a concise style of reading and writing without moves through *references.*

---

# References Are Non-Owning Pointers

- A reference is a kind of pointer.
- A reference does not own the data it points to.
- A reference is created with the `&` operator.

```rust
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");
    greet(&m1, &m2); // note the ampersands
    let s = format!("{} {}", m1, m2);
}

fn greet(g1: &String, g2: &String) { // note the ampersands
    println!("{} {}!", g1, g2);
}
```
ℹ️Because `g1` did not own “Hello”, Rust did not deallocate “Hello” on behalf of `g1`. Consistent with Box Deallocation Principle.
<!-- TODO: Add stack diagram -->

---

# Dereferencing a Pointer Accesses Its Data
- The `*` operator dereferences a pointer to access the data it points to.

```rust{all|1|2-3|6-7|9-10}
let mut x: Box<i32> = Box::new(1);
let a: i32 = *x;         // *x reads the heap value, so a = 1
*x += 1;                 // *x on the left-side modifies the heap value,
                         //     so x points to the value 2

let r1: &Box<i32> = &x;  // r1 points to x on the stack
let b: i32 = **r1;       // two dereferences get us to the heap value

let r2: &i32 = &*x;      // r2 points to the heap value directly
let c: i32 = *r2;    // so only one dereference is needed to read it
```

<img src="/img//4-understanding-ownership/2-references-and-borrowing/1.png" alt="Ownership Figure 4.1" width="150px" style="margin: 0 auto; display: block;" />

---

# Dereferencing a Pointer Accesses Its Data

- You probably won’t see the dereference operator very often when you read Rust code. 
- Rust implicitly inserts dereferences and references in certain cases, such as calling a method with the dot operator. Or macros like `println!`.

<v-click at="2">

- This implicit conversion works for multiple layers of pointers.

</v-click>

<v-click at="3">

- This conversion also works the opposite direction. 

</v-click>

```rust{all|1-4|6-9|11-14}
let x: Box<i32> = Box::new(-1);
let x_abs1 = i32::abs(*x); // explicit dereference
let x_abs2 = x.abs();      // implicit dereference
assert_eq!(x_abs1, x_abs2);

let r: &Box<i32> = &x;
let r_abs1 = i32::abs(**r); // explicit dereference (twice)
let r_abs2 = r.abs();       // implicit dereference (twice)
assert_eq!(r_abs1, r_abs2);

let s = String::from("Hello");
let s_len1 = str::len(&s); // explicit reference
let s_len2 = s.len();      // implicit reference
assert_eq!(s_len1, s_len2);
```

---

# Rust Avoids Simultaneous Aliasing and Mutation

<v-clicks depth="2">

- Pointers are a <span color="green">powerful</span> and <span color="orange">dangerous</span> feature because they enable aliasing.
- Aliasing is accessing the same data through different variables. 
- On its own, aliasing is harmless. But combined with mutation, we have a recipe for disaster.
  - By deallocating the aliased data, leaving the other variable to point to deallocated memory.
  - By mutating the aliased data, invalidating runtime properties expected by the other variable.
  - By concurrently mutating the aliased data, causing a data race with nondeterministic behavior for the other variable.

</v-clicks>

---

# An example of Aliasing and Mutation
- Arrays have a fixed length
- Vectors have a variable length by storing their elements in the heap.
- The macro `vec!` creates a vector with the elements between the brackets. 
- The capacity of a vector is the number of elements it can hold without reallocating.

<v-click at="2">

- If the vector is full, it will reallocate to a larger size.

</v-click>

```rust{all|1-2|3|4}
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &i32 = &v[2];
v.push(4);
println!("Third element is {}", *num);

```

<div class="image-row">
  <img v-click="['1', '2']" src="/img/4-understanding-ownership/2-references-and-borrowing/2-1.png" alt="Image 1">
  <img v-click="['2', '3']" src="/img/4-understanding-ownership/2-references-and-borrowing/2-2.png" alt="Image 2">
  <img v-click="3" src="/img/4-understanding-ownership/2-references-and-borrowing/2-3.png" alt="Image 3">
</div>

<v-click at="3">
☠️ <span color="orange">Undefined behavior:</span> pointer used after its pointee is freed.
</v-click>

<br>

<v-click>

⚠️The issue is that the vector `v` is both aliased (by the reference `num`) and mutated (by the operation `v.push(4)`)

</v-click>

<style>

.image-row {
  display: flex;
  justify-content: center;
  align-items: end;
  gap: 50px; /* Space between images */
}

.image-row img {
  width: 150px; /* Adjust the size of images */
  height: auto;
}

</style>

---
layout: statement
---

## 🦀<span color="#787878">Pointer Safety Principle:</span>

<br>

# data should never be aliased and mutated at the same time.

---

# Pointer Safety Principle

<v-clicks>

- Data can be aliased. 
- Data can be mutated. 
- **But data cannot be both aliased and mutated.**
- Rust enforces this principle for boxes (owned pointers) by disallowing aliasing.
- Assigning a box from one variable to another will move ownership, invalidating the previous variable.

</v-clicks>

<br>

<v-click>

### 🦀Owned data can only be accessed through the owner — no aliases.

</v-click>

<br>

<v-clicks>

- Because references are non-owning pointers, they need different rules than boxes to ensure the Pointer Safety Principle.
- By design, references are meant to temporarily create aliases.
- <span color="green">Borrow checker</span> ensures that references are valid and do not violate the Pointer Safety Principle.

</v-clicks>

<!-- 
- We are talking about pointers and pointer safety principle, so we need to talk about the `Box` type. They are owned pointers and don't allow aliasing 

- In the rest of this section, we will explain the basics of how Rust ensures the safety of references through the borrow checker. -->

---

# References Change Permissions on Places

<v-clicks depth="2">

- The core idea behind the borrow checker is that variables have three kinds of permissions on their data:
    - **Read** (<span color="orange">R</span>): data can be copied to another location.
    - **Write** (<span color="blue">W</span>): data can be mutated.
    - **Own** (<span color="red">O</span>): data can be moved or dropped.
- These permissions don’t exist at runtime, only within the compiler.
- They describe how the compiler “thinks” about your program before the program is executed.
- By default, a variable has read/own permissions (RO) on its data.
- If a variable is annotated with `let mut`, then it also has the write permission (W).

</v-clicks>

<br>

<v-click>

### 🦀The key idea is that references can temporarily remove these permissions.

</v-click>