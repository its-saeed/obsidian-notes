We use generics to create definitions for items like function signatures or structs, which we can then use with many different concrete data types.

## In Function Definitions
- To parameterize the types in a new single function, we need to name the type parameter, just as we do for the value parameters to a function.
- Rust’s type-naming convention is `UpperCamelCase`, `T` is default choice
- the function `largest` is generic over some type `T`:

```rust
fn largest<T>(list: &[T]) -> &T {
```

Here's the complete definition of the function: (Doesn't compile yet!)
```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {result}");

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {result}");
}
```

```sh
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

```

**What's the problem**
- `T` potentially can be anything. It can be `u32` or `String` or even `File`.
- However, `largest` requires that `T` is something you can compare with `>`
- This means `T` must implement `PartialOrd`
- Some types are comparable, some are not.
- C++ doesn't complain here, but Rust compiler requires you to state the expected capabilities of a generic type.

**What more?**
Unlike languages like Java where all objects have a set of core functions like `toString`, there are no core methods in Rust. Without specifying any restrictions, a generic type `T` how no capabilities:
- It's not printable
- It's can't be cloned
- It's not mutable

## In Struct Definitions

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```
Because the same type `T` is used for both `x` and `y`, the following code won't compile:
```rust
fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

We can fix it by having different types for `x` and `y`: 
```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

**BUT**
 ☠️If you’re finding you need lots of generic types in your code, it could indicate that *your code needs restructuring into smaller pieces.*

## In Enum Definitions
```rust
enum Option<T> { // Enum with one generic type
    Some(T),
    None,
}

enum Result<T, E> { // Enum with two generic types
    Ok(T),
    Err(E),
}
```

## In Method Definitions
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```
- Note that we have to declare `T` just after `impl` so we can use `T` to specify that we’re implementing methods on the type `Point<T>`.
- By declaring `T` as a generic type after `impl`, Rust can identify that the type in the angle brackets in `Point` is a generic type rather than a concrete type.
- Methods written within an `impl` that declares the generic type will be defined on any instance of the type, no matter what concrete type ends up substituting for the generic type.

We can also implement methods only on `Point<f32>` instances rather than on `Point<T>`:
```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```
 ℹ️ Other instances of `Point<T>` where `T` is not of type `f32` will not have this method defined.

 ⚠️You cannot simultaneously implement specific _and_ generic methods of the same name this way. For example, if you implemented a general `distance_from_origin` for all types `T` and a specific `distance_from_origin` for `f32`, then the compiler will reject your program

**Different generic types for struct definition and method definition**
Generic type parameters in a struct definition aren’t always the same as those you use in that same struct’s method signatures:
```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}

```

## Performance of Code Using Generics
The good news is that using generic types won’t make your program run any slower than it would with concrete types. But *how*? Rust accomplishes this by performing monomorphization of the code using generics at compile time.

**Monomorphization**
_Monomorphization_ is the process of turning generic code into specific code by filling in the concrete types that are used when compiled.
the compiler looks at all the places where generic code is called and generates code for the concrete types the generic code is called with.

```rust
let integer = Some(5);
let float = Some(5.0);
```
🦀Compiler identifies two kinds of `Option<T>`: one is `i32` and the other is `f64`. As such, it expands the generic definition of `Option<T>` into two definitions specialized to `i32` and `f64`. 
**It replaces the generic definition with specific ones**.

The monomorphized version of the code looks similar to the following:
```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

🦀Because of monomorphization we don't pay runtime cost.

## Question
#### Question 1

Imagine using a third-party function whose implementation you don't know, but whose type signature is this:

```rust
fn mystery<T>(x: T) -> T {
  // ????
}
```

Then you call `mystery` like this:
```rust
let y = mystery(3);
```

Assuming `mystery` uses no unsafe code, then the value of `y` must be?