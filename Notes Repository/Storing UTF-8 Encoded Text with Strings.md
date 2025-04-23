#rust #string #str #literals
## What is String
Rust has only one string type in the core language, which is the string slice `str` that is usually seen in its borrowed form `&str`.

The `String` type, which is provided by Rust’s standard library rather than coded into the core language, is a:
1. growable
2. mutable
3. owned
4. UTF-8 encoded string type

Not that both `String` and string slices are UTF-8 encoded. 
[[Unicode and Character Sets]]
[[What every programmer absolutely, positively needs to know about encodings and character sets to work with text]]
### Creating a New String
* Many of the same operations available with `Vec<T>` are available with `String` as well
* `String` is actually implemented as a wrapper around a vector of bytes with some extra guarantees, restrictions, and capabilities.

1. To Create a new empty string:
```rust
fn main() {
	let mut s = String::new();
	println!("String content is: {s}");
}
```

2. Using `to_string`
```rust
fn main() {
	let s = "initial content".to_string();
}
```

>`to_string` method is available on any type that implements the `Display` trait.

3. Using `String::from`
We can also use the function `String::from` to create a `String` from a string literal:
```rust
fn main() {
	let s = String::from("Initial content");
	println!("{s}");
}
```

#### Strings are UTF-8 encoded
Remember that strings are UTF-8 encoded, so we can include any properly encoded data in them:
```rust
let hello = String::from("سلام چوری");
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
```

### Updating a String
A `String` can grow in *size* and its contents can *change*, just like the contents of a `Vec<T>`, if you push more data into it.
#### Appending to a String with `push_str` and `push`

```rust
fn main() {
	let mut s = String::from("Foo");
	s.push_str("bar");
	println!("{s}");
}
```

The `push_str` method takes a string slice because we don’t necessarily want to take ownership of the parameter.
```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {s2}");
```
We can use `s2` in line 4 because the ownership is not taken by `s1`

**push** can be used to push a character to an string:
```rust
    let mut s = String::from("lo");
    s.push('l');
```

#### Concatenation with the + Operator or the format! Macro
```rust
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```
- `s1` is no longer valid after the addition.
- `&s2` is used to borrow it
why?
:LiInfo: The `+` operator uses the `add` method, whose signature looks something like this:
```rust
fn add(self, s: &str) -> String {
```

* We can only add a `&str` to a `String`
* We can’t add two `String` values together.
* The type of `&s2` is `&String`, not `&str`, as specified in the second parameter to `add`. So why does previous code compile?
	The reason we’re able to use `&s2` in the call to `add` is that the compiler can _coerce_ the `&String` argument into a `&str`. When we call the `add` method, Rust uses a _deref coercion_, which here turns `&s2` into `&s2[..]`. 

* `s2` will still be a valid `String` after this operation.
* `s1` will be MOVED into `add`, will no longer be valid.

Although `let s3 = s1 + &s2;` looks like it will copy both strings and create a new one, this statement instead does the following:
1. `add` takes ownership of `s1`,
2. it appends a copy of the contents of `s2` to `s1`,
3. and then it returns back ownership of `s1`.

❓What is the maximum number of times a heap allocation could occur in this program?
```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
```

For complex string concatenation it's better to use `format` function:
```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}");
```

### Indexing into Strings
If you try to access parts of a `String` using indexing syntax in Rust:
```rust
let s1 = String::from("Hello");
let h = s1[0];
```
you'll get the following error:
```sh
cargo run Compiling collections v0.1.0 (file:///projects/collections) error[E0277]: the type `str` cannot be indexed by `{integer}`
```

🦀The error and the note tell the story: **Rust strings don’t support indexing.**
Why? We need to see how strings are stored in Rust.

#### Internal Representation
A `String` is a wrapper over a `Vec<u8>`.
```rust
let hello = String::from("Hola");
```
`len` will be 4, each letters takes one byte when encoded in UTF-8.

But this string is different:
```rust
let hello = String::from("Здравствуйте");
```
⚠️If you were asked how long the string is, you might say 12. In fact, Rust’s answer is 24.
Each Unicode scalar value in this string takes 2 bytes.

⛔Therefore, an index into the string’s bytes will not always correlate to a valid Unicode scalar value.

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```
The `answer` will not be `З` because when encoded in UTF-8 the first byte of `3` is `208`and the second one is `151`. Returning `208` is not what user wants. So this code doesn't get compiled in rust.

#### Bytes and Scalar Values and Grapheme Clusters
Another point about UTF-8 is that there are 3 ways to look at strings from Rust's perspective:
1. as bytes
2. as scalar
3. as grapheme clusters(Closest thing to what we would call letters)

If we look at the Hindi word “नमस्ते” written in the Devanagari script, it is stored as a vector of `u8` values that looks like this:

`[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]`

That’s 18 bytes and is how computers ultimately store this data. If we look at them as Unicode scalar values, which are what Rust’s `char` type is, those bytes look like this:

`['न', 'म', 'स', '्', 'त', 'े']`

There are six `char` values here, but the fourth and sixth are not letters: they’re diacritics that don’t make sense on their own.
Finally, if we look at them as grapheme clusters, we’d get what a person would call the four letters that make up the Hindi word:

`["न", "म", "स्", "ते"]`

**The final reason why Rust doesn't allow us to index into string is:** Indexing operation is expected to always take constant time (O(1)). But it's not possible to guarantee this performance with a `String` why?

🦀Because Rust would have to walk through the contents from the beginning to the index to determine how many valid characters there were.

### Slicing Strings
⚠️Indexing into a string is often a bad idea because it’s not clear what the return type of the string-indexing operation should be: a byte value, a character, a grapheme cluster, or a string slice.

Rather than indexing using `[]` with a single number, you can use `[]` with a range to create a string slice containing particular bytes:

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];
```
Here `s` will be a `&str` containing the first four bytes of the string.

🦀If we were to try to slice only part of a character’s bytes with something like `&hello[0..1]`, Rust would panic at runtime in the same way as if an invalid index were accessed in a vector:

```sh
cargo run
Compiling collections v0.1.0 (file:///projects/collections) Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s Running `target/debug/collections`
thread 'main' panicked at src/main.rs:4:19:
byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`
```

### Methods for Iterating Over Strings
The best way to operate on pieces of strings is to be explicit about whether you want characters or bytes.

- To get characters out of a string:
```rust
fn main() {
	for c in "سلام چطوریُ".chars() {
	    println!("{c}");
	}
}
```

- To get bytes of a string:
```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

⚠️Remember that valid Unicode scalar values may be made up of more than one byte.
🦀Getting grapheme clusters from strings, as with the Devanagari script, is complex, so this functionality is not provided by the standard library.
