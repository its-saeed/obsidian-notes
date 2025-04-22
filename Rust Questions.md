1. Write a list data structure in rust that can hold items with different types like `string`, `u32`, `f32`
2. Write a program to print all Persian characters
3. How many bits do we need to encode a language with 8 characters? What about n characters?
4. Implement `give_me_a_salam` function in the code bellow:

```rust
fn give_me_a_salam() -> Vec<u8> {
    todo!()
}

fn main() {
    let salam = String::from_utf8(give_me_a_salam()).unwrap();
    assert_eq!(salam, "سلام".to_string());
}
```
You may need to use bitwise operations like `&, |, >>, <<`.
This table may help you:

| Character | Code Point  |
| --------- | ----------- |
| س         | 11000110011 |
| ل         | 11001000100 |
| ا         | 11000100111 |
| م         | 11001000101 |
