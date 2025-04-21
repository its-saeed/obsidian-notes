#error #panic

Sometimes bad things happen in your code, and there’s nothing you can do about it. In these cases, Rust has the `panic!` macro.

**There are two ways to cause a panic in practice:**
1. by taking an action that causes our code to panic (such as accessing an array past the end) 
2. by explicitly calling the `panic!` macro.

**After Panic**
By default, these panics will:
1. print a failure message,
2. unwind, 
3. clean up the stack, 
4. and quit.

### Unwinding the Stack or Aborting in Response to a Panic
By default, when a panic occurs the program starts _unwinding_:
- Rust walks back up the stack and cleans up the data from each function it encounters.
- But it's a lot of work

Rust, therefore, allows you to choose the alternative of immediately _aborting_:
- end the program without cleaning up
- the binary would be smaller
- put `panic = 'abort'` in Cargo.toml file

```toml
[profile.release]
panic='abort'
```

### Let's panic!

```rust
fn main() {
    panic!("crash and burn");
}
```

```console
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`
thread 'main' panicked at src/main.rs:2:5:
crash and burn
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
ℹ️ We can use the backtrace of the functions the `panic!` call came from to figure out the part of our code that is causing the problem.

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```
We're reading out of the vector so Rust panics here.

```console
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
Based on the output we can set `RUST_BACKTRACE` environment variable to get a backtrace of exactly what happened to cause the error.

**What is backtrace**
A _backtrace_ is a list of all the functions that have been called to get to this point.

**How to read a backtrace**
start from the top and read until you see files you wrote. That’s the spot where the problem originated.
- The lines above that spot are code that your code has called;
- the lines below are code that called your code.
```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/std/src/panicking.rs:645:5
   1: core::panicking::panic_fmt
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:72:14
   2: core::panicking::panic_bounds_check
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:208:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/slice/index.rs:255:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/alloc/src/vec/mod.rs:2770:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```
