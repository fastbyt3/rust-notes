# Vectors

## creating vecs

```rust
let mut v: Vec<i32> = Vec::new();

let x = vec![1,2,3,4,5];
```

## accessing vecs elements
- vecs are indexed from 0

```rust
let v = vec![1,2,3,4,5];
println!("First element is {}", v[0]);

let second: &i32 = &[1];

let third = v.get(2);
```

- In the above case when we use `.get(i: i32)` what we get is an `Option<&i32>`
- `get()` is preferred when we want to handle the case where element at that position doesnt exist
- and `[i]` is better if we want rust to panic if we access a non-existent element

- If we hold a reference to an element which was got using `get` or `[]` we can NOT update the vec(using `push`..)
    - this is cos push takes a mutable self reference which cant co-exist with a immutable ref


## iterating

```rust
let v = vec![1,2,3,4];

for i in &mut v {
    *i += 10; // use dereference operator to update the reference
}
```

```rust
let nums = vec![1, 2, 3];

for i in 0..nums.len() {
    println!("{}", nums[i]);
}
```

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", num); // In this case even tho num is of type &i32 println handles dereferencing
}
```

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```
