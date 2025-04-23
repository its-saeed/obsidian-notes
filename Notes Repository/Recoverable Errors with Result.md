- Most errors aren’t serious enough to require the program to stop entirely.
- Sometimes when a function fails it’s for a reason that you can easily interpret and respond to.
	- If file doesn't exist, create a new one instead of panic!
### What's `Result`
```rust
enum Result<T, E> { // T and E are generic type parameters
    Ok(T),
    Err(E),
}
```

- `T` represents the type of the value that will be returned in a success case within the `Ok` variant.
- `E` represents the type of the error that will be returned in a failure case within the `Err` variant.

#### Example
```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

- If `open` succeeds, type of success value is `std::fs::File`
- If `open` fails, type of the failure value is `std::io::Error`
- `Result` means `open` may succeed or fail
- Because `Result` is an enum we need to handle all of the cases:
```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```
- Exactly like `Option`, `Result` and all of its variants are loaded by prelude

### Matching on Different Errors
The previous code will `panic!` no matter why `File::open` failed.
However, we want to:
- create the file and return the handle to the new file, if file doesn't exist
- `panic!` otherwise

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {e:?}"),
            },
            other_error => {
                panic!("Problem opening the file: {other_error:?}");
            }
        },
    };
}
```

#### Shortcuts for Panic on Error: `unwrap` and `expect`
**unwrap**
- If the `Result` value is the `Ok` variant, `unwrap` will return the value inside the `Ok`.
- If the `Result` is the `Err` variant, `unwrap` will call the `panic!` macro for us.

**expect**
- Same as `unwrap` with some error message included
- Using `expect` instead of `unwrap` and providing good error messages can convey your intent and make tracking down the source of a panic easier.

### Propagating Errors
Handling the error in a parent function which called the current function instead of handling it within the function itself.

**Why**
Because it gives more control to the calling code, where there might be **more information** or **logic** that dictates how the error should be handled than what you have available in the context of your code.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```
- The code that calls this code will then handle getting either an `Ok` value that contains a username or an `Err` value that contains an `io::Error`.
- It’s up to the calling code to decide what to do with those values.
	- It can panic! if `Err` is returned
	- or it can use a default username

**🦀This pattern of propagating errors is so common in Rust that Rust provides the question mark operator `?` to make this easier.**

#### A Shortcut for Propagating Errors: the ? Operator
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```
- If the value of the `Result` is an `Ok`:
	- the value inside the `Ok` will get returned from this expression, 
	- and the program will continue.
- If the value is an `Err`:
	- the `Err` will be returned from the whole function as if we had used the `return` keyword so the error value gets propagated to the calling code.

⚠️There is a difference between what the `match` expression does and what the `?` operator does:
Error values that have the `?` operator called on them go through the `from` function, defined in the `From` trait in the standard library.

**From Trait**
Is used to convert values from one type into another.

When the `?` operator calls the `from` function, the error type received is converted into the error type defined in the return type of the current function.
- This is useful when a function returns one error type to represent all the ways a function might fail, even if parts might fail for many different reasons.

❓What should we do if we want to change `read_username_from_file` to return a custom error type like `OurError`

We could even shorten this code further by chaining method calls immediately after the `?`:
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```

ℹ️Reading a file into a string is a fairly common operation, so the standard library provides the convenient `fs::read_to_string` function that:
- opens the file,
- creates a new `String`,
- reads the contents of the file,
- puts the contents into that `String`, 
- and returns it.

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

#### Where The ? Operator Can Be Used
The `?` operator can only be used in functions whose return type is compatible with the value the `?` is used on.

```rust
use std::fs::File;
// This program won't compile
fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```

```sh
$ cargo run Compiling error-handling v0.1.0 (file:///projects/error-handling) error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`) --> src/main.rs:4:48 | 3 | fn main() { | --------- this function should return `Result` or `Option` to accept `?` 4 | let greeting_file = File::open("hello.txt")?; | ^ cannot use the `?` operator in a function that returns `()` | = help: the trait `FromResidual<Result<Infallible, std::io::Error>>` is not implemented for `()`
```

🦀This error points out that we’re only allowed to use the `?` operator in a function that returns `Result`, `Option`, or another type that implements `FromResidual`.

You can only use `?` on `Option` in a function that returns `Option`:
- If value is `None`, the `None` is returned early from that function
- If the value is `Some`, the value inside the `Some` is the resultant value of the expression, and the function continues.
```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

**Mixing `Result` and `Option`**
It's not possible to use `?` on `Option` in a function that returns `Result` and vise versa.
🦀You can use methods like the `ok` method on `Result` or the `ok_or` method on `Option` to do the conversion explicitly.

**Return `Result` from main**
- Main is the entry point and and exit point of the program, hence the return type can't be anything.
- But it can be `Result<(), E>`
```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```
- The `Box<dyn Error>` type is a _trait object_
	- to mean “any kind of error.”
- When a `main` function returns a `Result<(), E>`, the executable will:
	- exit with a value of `0` if `main` returns `Ok(())`
	- exit with a nonzero value if `main` returns an `Err` value.
- The `main` function may return any types that implement [the `std::process::Termination` trait](https://doc.rust-lang.org/std/process/trait.Termination.html)
	- It contains a `report` function that returns `ExitCode`