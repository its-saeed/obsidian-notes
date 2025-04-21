#hash #map

**Definition**
The type `HashMap<K, V>` stores a mapping of keys of type `K` to values of type `V` using a _hashing function_

**Hashing function**
determines how it places these keys and values into memory. [[Hash Functions]]

**Other names**
_hash_, _map_, _object_, _hash table_, _dictionary_, or _associative array

**When to use**
Hash maps are useful when you want to look up data not by using an index, as you can with vectors, but by using a key that can be of any type.

### Creating a New Hash Map
- It's not already in the prelude, we need to import it.
- Data stored on the heap. [[Heap vs. Stack]]
- Like vectors, hash maps are homogeneous. 
	- All of the keys most have the same type.
	- All of the values must have the same type.
```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

	println!("{scores:?}");
}
```

### Accessing Values in a Hash Map
##### Using `get`
```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
```
- The `get` method returns an `Option<&V>`; 
	- if there’s no value for that key in the hash map, `get` will return `None`.
- This program handles the `Option` by calling `copied` to get an `Option<i32>` rather than an `Option<&i32>`, 
- then `unwrap_or` to set `score` to zero if `scores` doesn’t have an entry for the key.

#### Iterating using `for`

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
```

### Hash Maps and Ownership
* For types that implement the `Copy` trait, like `i32`, the values are copied into the hash map.
* For owned values like `String`, the values will be moved and the **hash map** will be the owner of those values.

```rust
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
```

🦀If we insert references to values into the hash map, the values won’t be moved into the hash map. The values that the references point to **must be valid for at least as long as the hash map is valid.** [[Validating References with Lifetimes]]
### Updating a Hash Map
- The number of key and value pairs is growable.
- each unique key can only have one value associated with it at a time
- we can have different keys with the same value.

**When you want to change the data in a hash map, you have 3 options if the key already have a value assigned to**:
1. replace the old value with the new value, completely disregarding the old value.
2. keep the old value and ignore the new value, only adding the new value if the key _doesn’t_ already have a value.
3. combine the old value and the new value

#### Overwriting a Value
If we insert a key and a value into a hash map and then insert that same key with a different value, the value associated with that key will be replaced.

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{scores:?}");
}
```

#### Adding a Key and Value Only If a Key Isn’t Present
It’s common to check whether a particular key already exists in the hash map with a value and then to take the following actions:
- if the key does exist in the hash map, the existing value should remain the way it is;
- if the key doesn’t exist, insert it and a value for it.

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{scores:?}");
}
```
- Hash maps have a special API for this called `entry` that takes the key you want to check as a parameter.
- The return value of the `entry` method is an enum called `Entry` that represents a value that might or might not exist.
- The `or_insert` method on `Entry` is defined to:
	- return a mutable reference to the value for the corresponding `Entry` key if that key exists,
	- and if not, it inserts the parameter as the new value for this key and returns a mutable reference to the new value.

#### Updating a Value Based on the Old Value

```rust
use std::collections::HashMap;
fn main() {
    let text = "hello world wonderful world";
    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{map:?}");
}
```

### Hashing Functions
[[Hash Functions]]
- `HashMap` uses a hashing function called _SipHash_ by default. 
- We can use other hasher
- hasher is a type that implements `BuildHasher` trait

### Questions
1. Will this program pass the compiler?
```rust
use std::collections::HashMap;
fn main() {
  let mut h = HashMap::new();
  h.insert("k1", 0);
  let v1 = &h["k1"];
  h.insert("k2", 1);
  let v2 = &h["k2"];
  println!("{} {}", v1, v2);
}
```