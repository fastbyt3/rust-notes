# Hashmap

## Creating

- standard way of creating using collections

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
```

- using `collect` to construct a hashmap from "stream"

```rust
let teams = vec![String:from("blue"), String::from("red")];
let scores = vec![6, 9];

let team_scores: HashMap<_, _> = teams.iter().zip(scores.iter()).collect();
```

> Types that impl `Copy` trait will have their values copied to hashmap
> Owned values (like `String`) will be **moved** and hashmap will be the owner

## Inserting

```rust
map.insert(String::from("black"), 420); // <- (k, v)
```

## Accessing values

```rust
for (k, v) in &scores {
    println!("{}: {}", k, v);
}

scores.get(k); // -> Option(&v) : Some(&v)/None
```

## Updating

- Overwriting:

```rust
scores.insert(String::from("blue"), 10);
scores.insert(String::from("blue"), 69);

scores.get(String::from("blue")); // returns Option of reference to 69
```

- insert if key is not present
    - HMs have a spl API: `entry` -> takes key as param and returns enum called `Entry`
    - has methods like `or_insert` and many more
    - [reference to entry](https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html)

```rust
let mut map: HashMap<String, i32> = HashMap::new();

map.insert(String::from("Red"), 10);

map.entry(String::from("Red")).or_insert(69); // wont insert cos Red key exists
map.entry(String::from("Blue")).or_insert(9999); // will insert cos Blue key doesnt exist

// map = {Red: 10, Blue: 9999}
```

- `or_insert` returns a mutable reference to value of the key

- Update based on old value

```rust
// Problem: count len of each word's occurance in string

let mut map = HashMap::new();
let s = String::from("Hello John hope you are doing well");

for word in s.split_whitespace() {
    let count = map.entry(word).or_insert(0); // Insert 0 if word came across first time
    *count += 1; // dereference and add 1
}

println("{:?}", map);
```


