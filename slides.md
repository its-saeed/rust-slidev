---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev
lineNumbers: true
---

# Rust Programming

By Saeed Dadkhah

---
transition: fade-out
layout: section
---

# 🦀 Understanding Ownership

---
layout: statement
---

# <span color="green">Ownership</span> is a discipline for ensuring the <span color="green">safety</span> of Rust programs.

<br>

<v-click>

## <span color="green">Safety</span> is the Absence of <span color="orange">Undefined Behavior☠️</span>

</v-click>

<style>
  h1 {
    font-size: 2rem;
    font-weight: bold;
  }
</style>

---

# What is Undefined Behavior?
Undefined Behavior (UB) is a term used in programming languages to describe a situation where the behavior of a program is unpredictable or not well-defined. This can occur when the program violates the language's rules or specifications, leading to unexpected results or crashes.
It can happen due to various reasons, such as:

<v-clicks>

- Accessing memory that has already been freed
- Dereferencing a null or invalid pointer
- Using uninitialized variables
- Buffer overflows

</v-clicks>

<v-click>

## What to do?
😀In a safe programming language, errors are trapped as they happen.

☹️In an unsafe programming language, errors are not trapped. Rather, after executing an erroneous operation the program keeps going, but in a silently faulty way that may have observable consequences later on.
</v-click>


---
layout: statement
---


# If any step in a program’s execution has undefined behavior, then the <span color="red">entire execution</span> is without meaning.☠️

<br>

<v-click>

## ⚠️Also, it’s not that the execution is meaningful up to the point where undefined behavior happens: the bad effects can actually <span color="orange">precede</span> the undefined operation.

</v-click>

<!-- 
If any step in a program’s execution has undefined behavior, then the entire execution is without meaning. This is important: it’s not that evaluating (1<<32) has an unpredictable result, but rather that the entire execution of a program that evaluates this expression is meaningless. Also, it’s not that the execution is meaningful up to the point where undefined behavior happens: the bad effects can actually precede the undefined operation. 

In summary: There’s nothing inherently bad about running with a ball in your hands and also there’s nothing inherently bad about shifting a 32-bit number by 33 bit positions. But one is against the rules of basketball and the other is against the rules of C and C++. In both cases, the people designing the game have created arbitrary rules and we either have to play by them or else find a game we like better.

-->

---

# Why Is Undefined Behavior Good?

It simplifies the compiler’s job, making it possible to generate very efficient code in certain situations. For example:

<v-clicks>

* High-performance array code doesn’t need to perform bounds checks.
* When compiling a loop that increments a signed integer, the C compiler does not need to worry about the case where the variable overflows and becomes negative

</v-clicks>

<br>

<v-click>

# Why Is Undefined Behavior Bad?
When programmers cannot be trusted to reliably avoid undefined behavior, we end up with programs that silently misbehave.

</v-click>

<!-- certain tight loops speed up by 30%-50% when the compiler is permitted to take advantage of the undefined nature of signed overflow. -->

---
layout: statement
---

# The key insight behind designing a programming language with undefined behavior is that the compiler is only <span color="orange">obligated</span> to consider cases where the behavior is <span color="green">defined</span>.

---

# Undefined Behavior in C

Consider the following function. The behavior is defined for some inputs and undefined for others.

```c
int32_t unsafe_div_int32_t (int32_t a, int32_t b) {
  return a / b;
}
```

<v-click>

## Why is this function unsafe?

</v-click>

<v-click>
This function has a precondition; it <strong>should only be called</strong> with arguments that satisfy this predicate:

```c
(b != 0) && (!((a == INT32_MIN) && (b == -1)))
```
</v-click>

<div v-click>
The compiler performs a case analysis:
</div>

<v-clicks>

- Case 1: `(b != 0) && (!((a == INT32_MIN) && (b == -1)))` Behavior of `/` operator is defined, so Compiler is obligated to emit code computing `a / b`
- Case 2: `(b == 0) || ((a == INT32_MIN) && (b == -1))` Behavior of `/` operator is undefined, so Compiler has no particular obligations

</v-clicks>

<!-- Now the compiler writers ask themselves the question: What is the most efficient implementation of these two cases? Since Case 2 incurs no obligations, the simplest thing is to simply not consider it. The compiler can emit code only for Case 1.

A Java compiler, in contrast, has obligations in Case 2 and must deal with it (though in this particular case, it is likely that there won’t be runtime overhead since processors can usually provide trapping behavior for integer divide by zero). -->

---

# More examples of Undefined Behavior in C

```c
int stupid (int a) {
  return (a+1) > a;
}
```

<v-click>

The precondition for avoiding undefined behavior is:

```c
(a != INT_MAX)
```
</v-click>

<v-click>

Here the case analysis done by an optimizing C or C++ compiler is:

</v-click>

<v-clicks>

- Case 1: `a != INT_MAX`. Behavior of `+` is defined: Computer is obligated to return `1`
- Case 2: `a == INT_MAX`. Behavior of `+` is undefined: Compiler has no particular obligations

</v-clicks>

<v-click>

⚠️Again, Case 2 is degenerate and disappears from the compiler’s reasoning. Case 1 is all that matters. Thus, a good x86-64 compiler will emit:

```asm
stupid:
  movl $1, %eax
  ret
```
</v-click>

---

# A Fun Case Analysis
```c{all|4|6|all}
static void __devexit agnx_pci_remove (struct pci_dev *pdev)
{
  struct ieee80211_hw *dev = pci_get_drvdata(pdev);
  struct agnx_priv *priv = dev->priv; 

  if (!dev) return;
  ... do stuff using dev ...
}
```
The idiom here is to get a pointer to a device struct, test it for null, and then use it. 

<div v-click="1">
<span class="error-text">☠️But there’s a problem!</span> In this function, the pointer is dereferenced before the null check.
</div>

<v-clicks at="2">

* Case 1: `dev == NULL`
“dev->priv” has undefined behavior: Compiler has no particular obligations
* Case 2: `dev != NULL`
Null pointer check won’t fail: Null pointer check is dead code and may be deleted

</v-clicks>

<v-click>

As we can now easily see, neither case necessitates a null pointer check. *The check is removed*, potentially creating an exploitable security vulnerability.

Read more about [Undefined Behavior.](https://blog.regehr.org/archives/213)

</v-click>

<style>
.warning-text {
  color: #ff9900; /* Bright orange text */
  font-size: 18px;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2); /* Subtle shadow */
}

.error-text {
  color: #e60000; /* Bold red text */
  font-size: 18px;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(255, 142, 142, 0.2); /* Subtle shadow */
}

</style>

<!-- About a year ago, the Linux kernel started using a special GCC flag to tell the compiler to avoid optimizing away useless null-pointer checks. The code that caused developers to add this flag looks like this  -->

---
layout: statement
---

# A foundational goal of Rust🦀 is to ensure that your programs never have <br> <span color="orange">undefined behavior☠️</span>

<br>

<v-click>
About 70% of reported security vulnerabilities in low-level systems are caused by memory corruption, which is one form of undefined behavior.
</v-click>

---

# A secondary goal of Rust🦀 is to prevent undefined behavior at <span color="green">compile-time</span> instead of <span color="yellow">run-time</span>.

<v-clicks>

* Avoiding the bugs in production, improving the <span color="green">reliability</span> of your software.
* Fewer runtime checks for those bugs, improving the <span color="green">performance</span> of your software.

</v-clicks>

<v-click>

## We need to understand ownership in terms of the undefined behaviors it prevents.
Why?

</v-click>

<v-click>

# Safety == Absence of Undefined Behavior☠️

</v-click>

---
layout: section
---

# Rust🦀 Memory Model

---

# Variables Live in the Stack

```rust{all|2|3|7-9|8|all}
fn main() {
    let n = 5;
    let y = plus_one(n);
    println!("The value of y is: {y}");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```