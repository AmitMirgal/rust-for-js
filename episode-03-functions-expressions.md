# Episode 03. Functions, return values, and expressions

> **Series:** Rust for JS Devs &nbsp;|&nbsp; **Prerequisite:** [Episode 02. Variables, types, and mutability](./episode-02-variables-types-mutability.md)

*A Rust function returns the last expression in its body. A semicolon drops that value.*

This episode follows [The Rust Book, chapter 3.3, Functions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html) and [Rust By Example, Functions](https://doc.rust-lang.org/rust-by-example/fn.html). Names match those pages, so you can read them beside this file. `another_function`, `five`, and `plus_one` are the Book's examples. `is_divisible_by` is the Rust By Example example.

---

## 01 · `fn` is a compile-time item

[Functions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html) start with `fn`, a name, parentheses, and a block. The Book's style for those names is snake case. `rustc` warns on camel case by default, with `non_snake_case`.

**JavaScript.** A `function` declaration can run before its line in the source.

```js
anotherFunction();

function anotherFunction() {
  console.log("Another function.");
}
```

A `const` arrow cannot. The call below throws a `ReferenceError` at run time.

```js
another();
const another = () => console.log("no");
```

**Rust.** A `fn` item at module scope can be called from anywhere in that module, including from an earlier line. The Book's point is that the compiler does not care whether the item sits above or below the call. The call still has to name an item in a scope the caller can see.

```rust
fn main() {
    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

**False friend.** JavaScript hoisting is a run-time rule for `function` declarations. A Rust `fn` item is compiled with the crate. It has no temporal dead zone, and it is not a value you assign with `=`. A nested `fn` item also cannot read a local from the enclosing function. A nested JavaScript function can.

**JavaScript.**

```js
function main() {
  const x = 1;
  function inner() {
    console.log(x);
  }
  inner();
}
```

**Rust.** The next Rust block does not compile. `rustc` reports `E0434`, "can't capture dynamic environment in a fn item", and the help text tells you to write a closure instead.

```rust
fn main() {
    let x = 1;
    fn inner() {
        println!("{x}");
    }
    inner();
}
```

Closures are a later episode. The Book's chapter is [Closures](https://doc.rust-lang.org/book/ch13-01-closures.html). The matching Rust By Example page is [Closures](https://doc.rust-lang.org/rust-by-example/fn/closures.html).

---

## 02 · Every parameter has a type

[Parameters](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#parameters) are the names in the signature. Arguments are the values at the call. The Book requires a type on every parameter so the compiler can check the body without guessing.

**JavaScript and TypeScript.**

```ts
function plusOne(x: number): number {
  return x + 1;
}
```

JavaScript runs the same function with the annotations removed.

```js
function plusOneJs(x) {
  return x + 1;
}
```

**Rust.**

```rust
fn plus_one(x: i32) -> i32 {
    x + 1
}

fn add(left: i32, right: i32) -> i32 {
    left + right
}

fn main() {
    assert_eq!(plus_one(5), 6);
    assert_eq!(add(2, 3), 5);
}
```

The next Rust block does not compile. A parameter with no type is a parse error.

```rust
fn plus_one(x) -> i32 {
    x + 1
}
```

**False friend.** TypeScript will infer a callback parameter from the surrounding call, as in `nums.map(n => n + 1)`. A Rust `fn` parameter never works that way. Closures can infer types. A `fn` item cannot.

**False friend.** Rust has no default-parameter syntax and no overload set.

**JavaScript.**

```js
function greet(name = "world") {
  return "hello " + name;
}

function len(value) {
  return value.length;
}
```

**Rust.** The next Rust block does not compile. There is no syntax for a default value on a parameter.

```rust
fn greet(name: i32 = 1) {}
```

The next Rust block does not compile. A second `len` in the same module is `E0428`, "the name `len` is defined multiple times".

```rust
fn len(x: i32) -> i32 {
    x
}

fn len(x: i64) -> i64 {
    x
}
```

Pass the value at the call, or write a second function with its own name. Generics show up later. They are not overloads.

Parameters follow the same binding rule as episode 02. The name is immutable unless you write `mut`. `mut` lets the function reassign its own binding. An `i32` argument is a separate value from the caller's binding.

**JavaScript.** Reassigning the parameter does not change the caller's number.

```js
function inc(n) {
  n += 1;
  return n;
}

const original = 1;
const next = inc(original);
```

**Rust.** The next block does not compile. Assignment to an immutable argument is `E0384`.

```rust
fn inc(n: i32) -> i32 {
    n += 1;
    n
}
```

This one compiles. `original` stays `1`.

```rust
fn inc(mut n: i32) -> i32 {
    n += 1;
    n
}

fn main() {
    let original = 1;
    let next = inc(original);
    assert_eq!(original, 1);
    assert_eq!(next, 2);
}
```

What changes when the argument is a `String`, or another type that is not a plain integer, is the next episode. Do not treat this `i32` sample as the rule for every type.

---

## 03 · Statements and expressions

The Book draws the line in [Statements and Expressions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#statements-and-expressions).

- A statement performs an action and does not produce a value you can bind.
- An expression evaluates to a value.

`let y = 6;` is a statement. The `6` on the right is an expression. A function definition is a statement. A function call is an expression.

**JavaScript.** Assignment is an expression, so one line can set two bindings to `6`.

```js
let y;
const x = (y = 6);
```

**Rust.** `let` does not produce a value. The next block does not compile. The error is "expected expression, found `let` statement".

```rust
fn main() {
    let x = (let y = 6);
}
```

Assignment is an expression, and its type is the unit type `()`. It does not evaluate to the assigned number. `y` becomes `6`. `x` becomes `()`.

```rust
fn main() {
    let mut y: i32;
    let x: () = y = 6;
    assert_eq!(y, 6);
    let _: () = x;
}
```

**False friend.** A JavaScript block does not yield its last line. You wrap the block in an arrow, or in a function you call immediately, and you write `return`. A Rust block is already an expression. The value is the last line. A semicolon on that line drops the value.

**JavaScript.**

```js
const price = (() => {
  const base = 100;
  const discount = 15;
  return base - discount;
})();
```

**Rust.**

```rust
fn main() {
    let price = {
        let base = 100;
        let discount = 15;
        base - discount
    };
    assert_eq!(price, 85);
}
```

The next Rust block does not compile. The semicolon turns the tail into a statement, so the block has type `()` and cannot land in an `i32`.

```rust
fn main() {
    let price: i32 = {
        let base = 100;
        base - 15;
    };
}
```

---

## 04 · The tail is the return value

[Functions with Return Values](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#functions-with-return-values) and [Rust By Example, Functions](https://doc.rust-lang.org/rust-by-example/fn.html) use the same rule. Write the type after `->`. The last expression is the value. `return` leaves early.

**JavaScript.** An arrow with an expression body returns that expression. An arrow with a block returns `undefined` unless you write `return`.

```js
const five = () => 5;

const plusOne = (x) => {
  x + 1;
};
```

`plusOne(5)` is `undefined`.

**Rust.** The block still returns the tail. `five` is the Book's example of a body that is only a value.

```rust
fn five() -> i32 {
    5
}

fn plus_one(x: i32) -> i32 {
    x + 1
}

fn main() {
    assert_eq!(five(), 5);
    assert_eq!(plus_one(5), 6);
}
```

The next Rust block does not compile. `rustc` says it expected `i32` and found `()`, and that the body "implicitly returns `()`" because the tail is gone. The help text says to remove the semicolon. That is `E0308`.

```rust
fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

**False friend.** In JavaScript the missing `return` is a silent `undefined`. In Rust the same habit is a compile error when the signature promised a real type.

Early exit still uses `return`. [Rust By Example](https://doc.rust-lang.org/rust-by-example/fn.html) does that in `is_divisible_by`, then lets the comparison be the tail. The `if` condition is a `bool`, with no parentheses around it. See [if/else](https://doc.rust-lang.org/rust-by-example/flow_control/if_else.html).

**JavaScript.**

```js
function isDivisibleBy(lhs, rhs) {
  if (rhs === 0) {
    return false;
  }
  return lhs % rhs === 0;
}
```

**Rust.**

```rust
fn is_divisible_by(lhs: u32, rhs: u32) -> bool {
    if rhs == 0 {
        return false;
    }
    lhs % rhs == 0
}

fn main() {
    assert!(is_divisible_by(10, 2));
    assert!(!is_divisible_by(10, 0));
}
```

**False friend.** JavaScript treats a non-zero number as true. Rust requires a `bool`. The Book states that rule in [if Expressions](https://doc.rust-lang.org/book/ch03-05-control-flow.html#if-expressions).

```js
const number = 3;
if (number) {
  console.log("truthy");
}
```

The next Rust block does not compile. The condition is `E0308`, expected `bool`, found integer.

```rust
fn main() {
    let number = 3;
    if number {
        println!("truthy");
    }
}
```

Compare that with an explicit `bool`.

```rust
fn main() {
    let number = 3;
    if number != 0 {
        println!("not zero");
    }
}
```

Because `if` is an expression, a function tail can be the `if` itself. Both arms have to share one type. The Book's listing assigns `if condition { 5 } else { 6 }` and then shows that `{ 5 }` and `{ "six" }` cannot share a binding.

**JavaScript and TypeScript.** The two branches may have different types.

```ts
function label(ok: boolean): number | string {
  if (ok) {
    return 5;
  }
  return "six";
}
```

**Rust.**

```rust
fn score_bonus(streak: i32) -> i32 {
    if streak > 2 {
        10
    } else {
        0
    }
}

fn main() {
    assert_eq!(score_bonus(3), 10);
    assert_eq!(score_bonus(1), 0);
}
```

The next Rust block does not compile. The arms have incompatible types, integer and `&str`.

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { "six" };
    println!("{number}");
}
```

`loop`, `while`, and `for` are the rest of [Control Flow](https://doc.rust-lang.org/book/ch03-05-control-flow.html). They are not this episode. The series index goes to ownership next.

---

## 05 · No value is still a type

A JavaScript function with no `return` gives back `undefined`. Rust By Example says a function that "doesn't" return a value actually returns the unit type `()`. You can write `-> ()`, or you can leave the return type off. Both mean the same signature.

**JavaScript.**

```js
function logScore(score) {
  console.log(score);
}

logScore(7);
```

The call's result is `undefined`.

**Rust.**

```rust
fn log_score(score: i32) {
    println!("score={score}");
}

fn log_score_explicit(score: i32) -> () {
    println!("score={score}");
}

fn main() {
    let result: () = log_score(7);
    let _: () = result;
    log_score_explicit(7);
}
```

The next Rust block does not compile. A tail of type `i32` does not match an omitted return type. The compiler's help text is to add `-> i32`.

```rust
fn plus_one(x: i32) {
    x + 1
}
```

**False friend.** `()` is one real value. It is not `null`, and it is not `undefined`. Episode 02 already pointed at `Option` for a missing value. Do not return `()` and then check it the way you would check `undefined`.

**False friend.** JavaScript `throw` leaves the function through an exception.

```js
function never() {
  throw new Error("no");
}
```

Rust By Example calls a function that never returns a diverging function, and writes its return type as `!`. [Diverging functions](https://doc.rust-lang.org/rust-by-example/fn/diverging.html) is that page. `()` means the function did return, with no information in the value. `!` means control did not come back. This episode does not use `!`.

---

## 06 · JavaScript beside Rust

| Idea | JavaScript or TypeScript | Rust |
| --- | --- | --- |
| Declare a function | `function f() {}` or `const f = () => {}` | `fn f() {}` |
| Name style | camel case | snake case, warned by `non_snake_case` |
| Call before the declaration line | `function` yes, `const` arrow no | `fn` item yes, in the same scope |
| Nested function reads a local | yes, it closes over it | `fn` item no, that is a closure |
| Parameter types | optional in JavaScript | required on every `fn` parameter |
| Default parameter | `function f(n = 1)` | no syntax for it |
| Overload | TypeScript overload signatures | one name, one signature, in that module |
| Reassign a parameter | allowed | write `mut` on that binding |
| Last line of a block | not a return | the value, when the semicolon is absent |
| Return with no `return` keyword | arrow expression body only | any tail expression, including a block |
| Forgot the value | `undefined` at run time | `()` if the signature allows it, otherwise a compile error |
| Missing value | `null` or `undefined` | not `()`, see `Option` in a later episode |
| `if (3)` | runs the branch | does not compile, the condition must be `bool` |

---

## 07 · Quick reference

```rust
fn another_function() {
    println!("Another function.");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}

fn inc(mut n: i32) -> i32 {
    n += 1;
    n
}

fn is_divisible_by(lhs: u32, rhs: u32) -> bool {
    if rhs == 0 {
        return false;
    }
    lhs % rhs == 0
}

fn score_bonus(streak: i32) -> i32 {
    if streak > 2 {
        10
    } else {
        0
    }
}

fn log_score(score: i32) -> () {
    println!("score={score}");
}

fn main() {
    another_function();

    let price = {
        let base = 100;
        base - 15
    };

    let mut y: i32;
    let assigned: () = y = 6;

    assert_eq!(plus_one(5), 6);
    assert_eq!(inc(1), 2);
    assert_eq!(price, 85);
    assert_eq!(y, 6);
    assert_eq!(is_divisible_by(10, 5), true);
    assert_eq!(score_bonus(3), 10);
    let _: () = assigned;
    let _: () = log_score(7);
}
```

---

## Summary

- A `fn` item is visible through its scope, and it does not capture locals. A nested JavaScript function does.
- Every parameter has a type. There is no default syntax and no second signature under the same name.
- `let` is a statement. A block's tail is an expression. A semicolon replaces that tail with `()`.
- Assignment's value is `()`, not the number you stored.
- The signature's `->` type is the tail expression. `return` is the early exit.
- No useful return value is `()`, which is not `undefined`.
- An `if` used as a value needs `bool`, and both arms share one type. Loops stay outside this episode.

---

## Up next. Episode 04

**Ownership.**

Passing an `i32` into `inc` left the caller's binding unchanged. Ownership is the rule for values that do not work that way, including `String`.
