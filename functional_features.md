[#](#.md) Functional Features of Rust -> Iterators & Closures

- programming in functional style often includes passing functions as values 
    - as arguments,
    - as return values
    - as variables for later use

- Closures -> function-like construct you can store in a variable
- Iterators -> way of processing series of elements


## Closures

- Unlike functions, Closures can capture values from the scope in which they are defined
- allow code reuse and behaviour customization

```rust
let expensive_closure = |num| {
	// do stuff
	thread::Sleep(Duration::from_secs(2));
	num
}
```

- Inside the `||` we define the parameters to the Closures
- This `let` statement means `expensive_closure` contains the definition of an anon function
    - and not the resulting value of the function
    - this allows us to store the code and call it at a later point

### Closure type inference and Annotation

- we can annotate variables just like how we do in functions if we want to increase explicitness and clarity

```rust
// fn
fn  add_one_v1   (x: u32) -> u32 { x + 1 }

// closure
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5); // This errors cos we set param type and return type effectively in prev line
```

### Storing closures using generic parameters and fn traits

- we can create a struct that will hold the closure and the resulting value
- The struct will exec the closure only when needed and cache the ret value thus making it faster to reuse
    - This pattern -> `Memoization` or `Lazy loading`

- Structs require explicit type definition even for closures
    - each closure tho identical in terms of fn signature has its own unique anonymous type
    - to define closures in structs, enums or func params we use ***Generics*** and ***trait bounds***

```rust
struct Cacher<T> 
	where T: Fn(u32) -> u32
{
	calculation: T,
	value: Option<u32>,
}

impl Cacher<T> where T: Fn(u32) -> u32 {
	fn new(calculation: T) -> Cacher {
		Cacher {
			calculation,
			value: None,
		}
	}

	fn value(&mut self, arg: u32) -> u32 {
		match self.value {
			Some(v) => v,
			None => {
				let v = calculation(arg);
				self.value = Some(v);
				v
			}
		}
	}
}

fn foo() {
	let mut expensive_operation = Cacher::new(|num| {
		// do stuff
		thread::Sleep(Duration::from_secs(2));
		num
	})

	expensive_operation.value(some_u32); // Exec the closure function and stores value in Cacher
	expensive_operation.value(some_u32); // Fetches from Cacher
}
```

### Capturing environment with closures

- closures can capture the environment they are defined in access vars that are in same scope

```rust
fn main() {
    let x = 4;
    let equal_to_x = |z| z == x;
    let y = 4;
    assert!(equal_to_x(y));
}
```

- Closures can capture values from env in 3 ways encoded in following `Fn` traits
	1. `FnOnce`
		- consumes vars it captures from its enclosing scope(environment)
		- Consumption is done by taking ownership of the vars and moving them into where the closure is defined
        - `Once` part represents closure cant take ownership of same vars more than once 
        - All Closures implement `FnOnce`
    2. `FnMut`
        - change the env cos it mutably borrows	values
    3. `Fn`
        - borrows vals from env immutably

- If we want to force a closure to take ownership we need to use `move` keyword before param list

```rust
fn main() {
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x; // closure takes ownership of x
    // println!("can't use x here: {:?}", x); // This err since main doesnt have ownership of x
    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```

## Iterators

```rust
pub trait Iterator {
	type item;

	fn next(&mut self) -> Option<Self::Item>; // must be impl if we wish to use Iterator trait
	
	// other methods with default impl
}
```

- `iter` method produces immutable references to the elements
- If we want to take ownership of the elements we can use `into_iter` method
- and in case of mutable references we can use `iter_mut` method

```rust
let v = vec![1,2,3,4];
let mut v_itrer = v.iter(); // rqeuired to be mut cos internal state changes

let sum = v_itrer.sum();

let some_val = v_itrer.next(); // sum() consumes iterator by taking ownership of it
```

- Producing Iterators: _Iterator adaptors_
    - allows change of iterators into different kind of iterators
    - unless and until we consume the iterator changes dont happen cos Iterators are Lazy

```rust
let v = vec![1,2,3,4];

let v_modified: Vec<u32> = v.iter().map(|x| x * 10).collect(); // vec![10, 20, 30, 40]
```

```rust
fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
	shoes.into_iter()
		.filter(|s| s.size == shoe_size)
		.collect()
}
```

### Creating own iterators with _Iterator_ trait

```rust
struct Counter {
	count: i32,
}

impl Iterator for Counter {
	type Item = i32;

	fn next(&mut self) -> Option<Self::Item> {
		self.count += 1;
		if self.count < 6 {
			Some(self.count);
		} else {
			None
		}
	}
}
```

- Rust perform an optimization technique called: `Unrolling`
    - removes overhead of loop controlling code and generates repetitive code for each loop iteration
    - all values get stored in registers so they can be accessed faster
    - this makes the zero cost abstraction of Iterators faster and more efficient
