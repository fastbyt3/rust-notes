# Lifetimes

- Most of the times lifetimes are implicit and inferred
- we must annotate lifetimes when lifetimes of references might be related in few different ways
- To annotate lifetimes we must use **Generic** lifetime parameters

## Preventing dangling references with lifetimes

- lifetime annotation syntax:
    ```rust
	&i32			// reference
	&'a i32			// reference with explicit lifetime
	&'a mut i32		// mutable reference with explicit lifetime
	```

```rust
fn largest<'a>(a: &'a str, b: &'a str) -> &'a str {
	if a.len() > b. len() {
		a
	} else {
		b
	}
}
```

- In the above code we create a generic lifetime annotation: `'a`
    - function signature tells the string slice returned from the function will live as long as `'a` lives

```rust
fn main() {
	let s1 = String::from("test");
	let s2 = "sheeesh";
	let x = largest(&s1, &s2);
	let y;

	let s3 = String::from("hola");

	{
		let s4 = "whoppers";
		y = largest(&s3, &s4);
	}

	println!("{}, {}", x, y);
}
```

### Thinking in terms of lifetime

```rust
fn foo<'a>(x: &'a str, y: &str) -> &'a str {
	y
}
```

- we dont need to specify lifetime for y since that is not gonna be part of whats returned

```rust
// THIS WILL NOT COMPILE
fn bar<'a>(x: &str, y: &str) -> &'a str {
	let result = String:from("test")
	result.as_str()
}
```

- the above function will fail and the reason behind is the lifetime `'a` is not gonna be applied to `result`
    - for cases like this the best option would be to return an _owned data type_

## Lifetime annotations in Struct defintions


```rust
// This means an instance of ImportantExcerpt CANNOT outlive the reference held in `part` field
struct ImportantExcerpt<'a> {
	part: &'a str,
}

fn main() {
    let novel = String::from("bla bla bla bla...... bla");

    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.' character");

    let i = ImportantExcerpt { part: &first_sentence };
}
```

## Lifetime Elision

- lifetime patterns programmed into Rust's analysis of reference is called Lifetime Elision
- this is a set of cases where compiler doesnt require explicit definition of lifetimes if the cases fit the code
- Lifetimes on functions or method parameters are called **input lifetimes**
- Lifetimes on return values are called **Output lifetimes**

- Total 3 rules: 1 for input lifetime and 2 for output lifetime

### First rule

- each parameter that is a reference gets its own lifetime parameter

```rust
fn foo<'a>(x: &'a i32);

fn foo<'a, 'b>(x: &'a i32, y: &'b i32);
```

### Second rule

- If there is exactly one input lifetime then its applied to all output lifetime parameters

```rust
fn foo<'a>(x: &'a i32) -> &'a i32;
```

### Thrid rule

- If there is a `&self` or `&mut self` reference
	- then the lifetime parameter of `self` is assigned to all output lifetime parameters


- If our program doesnt fit in any of the 3 rules then compiler would throw an err
    - and requires explicit defintion of lifetime parameters

```rust
fn foo<'a, 'b>(x: &'a i32, y: &'b i32) -> &i32; // Errrrrrr

// In this case we will get a compiler err
// By first rule each input is assigned a lifetime param
// But 2nd and 3rd rule dont fit thus compiler cant figure out the lifetime param of return value
```

## Static Lifetime

- `'static` denotes the entire duration of program
- All string literals have this lifetime annotation
    - Since they are stored in the memory of the binary

> Requires careful delibration whehter static lifetime is needed most of the times


