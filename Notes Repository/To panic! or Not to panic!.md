- It's possible to `panic!` in all situations.
- But when you `panic!` you’re making the decision that a situation is unrecoverable on behalf of the calling code.
- When you choose to return a `Result` value, you give the calling code options.
	- So `Result` is a good default choice when you're defining a function that might fail
### Examples, Prototype Code, and Tests
- In examples, it’s understood that a call to a method like `unwrap` that could panic is meant as a placeholder for the way you’d want your application to handle errors.
- Similarly, the `unwrap` and `expect` methods are very handy when prototyping:
	- They leave clear markers in your code for when you’re ready to make your program more robust.
- If a method call fails in a test, you’d want the whole test to fail, even if that method isn’t the functionality under test.
### Cases in Which You Have More Information Than the Compiler
It would also be appropriate to call `unwrap` or `expect` when you have some other logic that ensures the `Result` will have an `Ok` value, *but the logic isn’t something the compiler understands.*

```rust
    use std::net::IpAddr;

    let home: IpAddr = "127.0.0.1"
        .parse()
        .expect("Hardcoded IP address should be valid");
```

### Guidelines for Error Handling
- It’s advisable to have your code panic when it’s possible that your code could end up in a bad state.
	- _bad state_ is when some assumption, guarantee, contract, or invariant has been broken
	- *bad state* is something that's unexpected, as opposed to something that will likely happen.
	- Your code after this point needs to rely on not being in this bad state
- `panic!` is often appropriate if you’re calling external code that is out of your control and it returns an invalid state that you have no way of fixing.
- when failure is expected, it’s more appropriate to return a `Result`
	- a parser being given malformed data
	- an HTTP request returning 404
- When your code performs an operation that could put a user at risk if it’s called using invalid values:
	- your code should verify the values are valid first
	- panic if the values aren’t valid.
	- This is mostly for safety reasons: attempting to operate on invalid data can expose your code to vulnerabilities.
	- This is the main reason the standard library will call `panic!` if you attempt an out-of-bounds memory access
- Panicking when the function contract is violated makes sense because a contract violation always indicates a *caller-side bug*
- It's recommended to use Rust type system to prevent some of these error at compile time by compiler
	- If you expect to have something, don't use `Option` because then you must handle `None` which doesn't make sense to you.
	- Use `u32` if you expect to have an unsigned number
### Creating Custom Types for Validation
```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```
Instead of checking if `value` is in `1..100` range every time you want to use it, just create a type around it and do the verification once. 