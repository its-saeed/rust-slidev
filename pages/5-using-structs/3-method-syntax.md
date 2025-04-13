---
transition: fade-out
layout: section
---

# 🦀Struct Method Syntax

---

## Method Syntax
Methods are similar to functions:
- Declared with `fn` keyword
- Can have parameters and a return value

But, unlike functions:
- Methods are defined within the context of a struct (or enum or trait).
- The first parameter is always `self`, which represents the instance of the struct.

---

## Defining a Method
```rust{all|2-5|7-11|14-17|19-22}
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

- In the signature for `area`, we use `&self` instead of `rectangle: &Rectangle`.
- The `&self` is actually short for `self: &Self`.
- Within an `impl` block, the type `Self` is an alias for the type that the `impl` block is for. 
- Methods must have a parameter named self of type `Self` for their first parameter
    - So Rust lets you abbreviate this with only the name self in the first parameter spot.
    - we still need to use the `&` in front of the `self` shorthand to indicate that this method borrows the Self

---

- Methods can:
    - Borrow self **immutably**, `&self`
    - borrow self **mutably**. `&mut self`
    - Take **ownership** of self. `self`
- Having a method that takes ownership of `self` is rare
    - Usually used when the method transforms `self` into something else
    - Preventing the caller from using the original instance after the transformation.
- The main reason for using methods instead of functions is for organization and readability.
    - Users of our code do not need to search for capabilities of `Rectangle` in various places in the library we provide.
- we can choose to give a method the same name as one of the struct’s fields.

```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}
```
---

## Methods with More Parameters
Methods can take multiple parameters that we add to the signature after the self parameter

```rust{all|6-8|21}
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
}
```

---

# Associated Functions
- All functions defined within an `impl` block are called associated functions because they’re associated with the type named after the `impl`.
- We can define associated functions as functions that don’t have `self` as their first parameter.
    - So they are not methods.
- We’ve already used `String::from`
- Associated functions are often used for constructors that will create an instance of the struct.

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}
```

- To call this associated function, we use the `::` syntax with the struct name;

---

## Multiple impl Blocks
Each struct is allowed to have multiple impl blocks. 

```rust 
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

---

## Method Calls are Syntactic Sugar for Function Calls
```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn set_width(&mut self, width: u32) {
        self.width = width;
    }
}
```
```rust{all|5-6|9-10}
    let mut r = Rectangle { 
        width: 1,
        height: 2
    };
    let area1 = r.area();
    let area2 = Rectangle::area(&r);
    assert_eq!(area1, area2);

    r.set_width(2);
    Rectangle::set_width(&mut r, 2);
```

---

## Methods and Ownership
Suppose we have these methods implemented for `Rectangle`:

```rust
impl Rectangle {    
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn set_width(&mut self, width: u32) {
        self.width = width;
    }

    fn max(self, other: Rectangle) -> Rectangle {
        Rectangle { 
            width: self.width.max(other.width),
            height: self.height.max(other.height),
        }
    }
}
```
---

## Reads and Writes with &self and &mut self
- If we make an owned rectangle with `let rect = Rectangle { ... }`, then rect has <span color="orange">R</span> and <span color="red">O</span> permissions. 
- With those permissions, it is permissible to call the `area` and `max` methods.
<img src="/img/5-using-structs/3-method-syntax/code-1.png" alt="Ownership Figure 4.1" width="500px" style="margin: 0 auto; display: block;" />
- But not possible to call `set_width` because it requires <span color="blue">W</span> permission.
- Same for calling `set_width` on an immutable reference to a `Rectangle`, even if the underlying rectangle is mutable.

---

## Moves with self
- Calling a method that expects `self` will move the input struct.
- Unless the struct implements `Copy`.
- We can't use a `Rectangle` after passing it to `max`:
<img src="/img/5-using-structs/3-method-syntax/code-2.png" alt="Ownership Figure 4.1" width="400px" style="margin: 0 auto; display: block;" />
- ⚠️A similar situation arises if we try to call a self method on a reference. 
<img src="/img/5-using-structs/3-method-syntax/code-3.png" alt="Ownership Figure 4.1" width="500px" style="margin: 0 auto; display: block;" />
- `self` is missing <span color="red">O</span> permissions in the operation self.max(..)
- ⛔error[E0507]: cannot move out of `*self` which is behind a mutable reference


---

## Good Moves and Bad Moves
why does it matter if we move out of *self?

 - In fact, for the case of Rectangle, it actually is safe to move out of `*self`, even though Rust doesn’t let you do it.
 - Because Rectangle does not own any heap data.
 - We can get Rust to compile `set_to_max` by simply adding `#[derive(Copy, Clone)]`

 ❓Why doesn’t Rust automatically derive `Copy` for Rectangle?

 Rust does not auto-derive Copy for stability across API changes. Imagine that the author of the Rectangle type decided to add a name: String field. Then all client code that relies on Rectangle being Copy would suddenly get rejected by the compiler. To avoid that issue, API authors must explicitly add #[derive(Copy)] to indicate that they expect their struct to always be Copy.

---
layout: two-cols
---

## Good Moves and Bad Moves
```rust
struct Rectangle {
    width: u32,
    height: u32,
    name: String,
}

impl Rectangle {    
  fn max(self, other: Self) -> Self {
    let w = self.width.max(other.width);
    let h = self.height.max(other.height);
    Rectangle { 
      width: w,
      height: h,
      name: String::from("max")
    }
  }
    fn set_to_max(&mut self, other: Rectangle) {
        let max = self.max(other);
        drop(*self); // This is usually implicit,
                         // but added here for clarity.
        *self = max;
    }
}
```

::right::

<br>
<br>


```rust 
fn main() {
    let mut r1 = Rectangle { 
        width: 9, 
        height: 9, 
        name: String::from("r1") 
    };
    let r2 = Rectangle {
        width: 16,
        height: 16,
        name: String::from("r2")
    };
    r1.set_to_max(r2);
}
```

☠️When we do `*self = max`, we encounter undefined behavior. 