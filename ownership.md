# Ownership in Rust

> Understanding ownership you wont need to think about the stack and heap very often :)

## Ownership rules

- each value has a var: **Owner**
- there can only be one owner at a time
- when owner goes out of scope value will be dropped

## ways vars and data interact: 

### MOVE

```rust
let x = 5;
let y = x;
```

- In this case we bind the **value** 5 to x and later say y is equal to x meaning the value of x and y are equal
    - implies a copy of x is created

- Consider the example with Strings instead of i32:

```rust
let x = String::from("hello");
let y = x;
```

- In this case x doesn't hold the value `Hello` directly rather it has a struct that has:
    - `ptr` -> addr pointing to mem where string is stored
    - `len` -> #bytes used
    - `capacity` -> total mem recv from OS

- When we say `y = x` the value (`ptr, len, capacity`) of x are copied over to y
    - In this case y inherently points to same memory containing `hello`

> Data on the heap (string) is not being made a copy of

- In GC when either x, y go out of scope GC attempts to free the heap memory
    - and when they both go out of scope -> **Double free error** a safety bug occurs

- to prevent this Rust considers `x` to no longer be valid
    - therefore when `x` goes out of scope GC doesnt need to free anything
    - Rust shows `value used here after move` error
    - Ownership has entered the chat :)

> Shallow copy and Deep copy are concepts that associate with this
> In rust since we _invalidate_ `x` this operation is called a **Move**

### Clone

- Performing a `deep copy` of heap data we use `Clone` method

```rust
let x = String::from('hello');
let y = x.clone();

println!("{} & {}", x, y); // both can be printed
```

- `x` in this case isnt invalidated

### Copy

```rust
let x = 5;
let y = x;

println!("{} & {}", x, y);
```

- This here is called a **Copy**
- why is this valid: values like these whose sizes are known at compile time are stored entirely in stack
    - making it super easy and efficient to make a copy of

- Rust has a `Copy` trait that allows a older variable (like x) to be usable even if we assign it later
    - This is only if the type doesnt impl `Drop` trait

- Scalar values allow copy
    - Int types: u32, i32, ....
    - Bool: true, false
    - char
    - float-point: f32, f64
    - tuples if they contain types that allow Copy (above mentioned)


## Ownerships and functions

```rust
fn main() {
    let s = String::from("hello");
    take_ownership(s);

    let x = 1;
    makes_copy(x);

    println!("{}", s); // errors
    println!("{}", x); // works fine
}

fn take_ownership(s: String) {
    println!("{}", s);
    // s goes out of scope -> string memory is freed
}

fn makes_copy(some_int: i32) {
    println!("{}", some_int);
    //some_int goes out of scope and is destroyed
}
```

- if a func returns a string -> Gives ownership
- if a func takes an object and returns it takes ownership until func exec and returns it to calling func

```rust
fn main() {
    let s = String::from("Hello");
    println!("{}", s);
    let x = foo(s);
    // println!("{}", s); // Errors cos this is not the owner anymore and its mem got cleared
    println!("{}", x);
}

fn foo(mut some_str: String) -> String {
    some_str.push_str(" world!");

    some_str
}
```

## References & Borrowing -> use a value but dont take ownership

```rust
fn main() {
    let s = String::from("hello");
    let len = calc_len(&s); // pass a reference to s
    // print
}

fn calc_len(some_str: &String) -> u32 {
    s.len()
}
```

- In this case what `some_str` has is a `ptr` to the object `s` in main func
- using `&` allows the func to _refer_ to the object and not take ownership
    - thus when it goes out of scope GC need not dealloc the mem of `s`

> having references as function params is what we call as **Borrowing**

### Mutable references

```rust
fn main() {
    let mut s = String::from("hello");

    println!("{}", s);

    foo(&mut s);

    println!("{}", s);
}

fn foo(x: &mut String) {
    x.push_str(" world!")
}
```

- restriction of mutable reference
    - only one mutable reference to a particular piece of data in particular scope

```rust
let mut s1 = String::from("hellop");

let r1 = &mut s1;
let r2 = &mut s1;

println!("{} {}", r1, r2); // errors -> first mutable borrow shouldnt be used after definining second
```

```rust
{
    let r1 = &mut s1;
    // r1 goes out of scope here
}

let r2 = &mut s1; // this is valid
```

- great adv -> prevent data races at compile time

- Another important fact is **we cannot have a mutable reference when we have an immutable one**
- multiple immutable referneces are fine but mixing aint allowed


### Dangling references

- Rust compiler doesnt allow references to be dangling in any case
- it ensures the data will not go out of scope before the reference to data does => Lifetimes

```rust
fn produce_dangling_ptr() -> &String {
    let s = String::from("hello");

    &s // creates a refernce to s ("hello") and is returned
} // s goes out of scope -> hello is cleared from memory

fn foo() {
    let x = produce_danling_ptr()
    // if we try to access x here we have a dangling ptr
    // cos x doesnt point to the memory holding "hello"
}
```

- correct approach is to return the String without a reference that way the calling func retains ownership & memory

## Slice

- Slice is a datatype that **does NOT** have ownership
- reference a continous sequence of elements in a collection rather than the whole collection

```rust
// DOESNT USE SLICES
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' { return i; } // If space is encountered return the index
    }

    s.len(); // if no space is encountered ret len since thats the end of the whole word
}

// first_word(&String::from("hello world")) -> 5
```

- if we clear the String or update it it causes a data error as the state of s is not connected to actual word
    - the index might not point to the correct index in curr string

### String slices

- string slices references to a part of the string

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

- value of a string slice: `ptr` and `len`

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..] // returns whole string
}
```

- Now if we try to modify/clear the original string in calling func:

```rust
fn main() {
    let mut s = String::from("hello");

    let first = first_word(&s); // this is immutable borrow (we dont pass `mut` here)

    s.clear(); // error -> clear takes a mutable self reference

    // mutable & immutable references cannot exist in same scope

    println!("{}", first);
}
```


#### String literals are Slices

- We define a literal like:

```rust
let s = "Hello world";
```

- here s is of type `&str` as it points directly to the place where string is stored **in binary**.

- and s here is an immutable reference


#### String slices as Function params

```rust
fn first_word(s: &str) -> &str {
    // ....
}

fn main() {
    let x = String::from("hello"); // <- String

    let y = "world"; // <- &str

    let x_first = first_word(&x[..]);
    let y_first = first_word(&y[..]); // or first_word(y);
}
```


#### Other slices

- Like String slices we can use the same concept in arrays too

```rust
let a = [1,2,3,4];

let slice = &a[1..3];
```
