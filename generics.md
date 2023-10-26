# Generics

## Generic data types

### Function definitions

```rust
fn largest<T>(list: &[T]) -> T {
//	....
}
```

- when we compare directly using generics we get error that operation cannot be applied
    - `std::cmp::PartialOrd` is a trait that must be used for this


### Struct definitions

```rust
struct Point<T> {
	x: T,
	y: T,
}

fn main() {
	let int_point = Point { x: 5, y: 0 };
	let float_pt = Point { x: 5.5, y: 2.0 };
	let mixed_pt = Point { x: 5.5, y: 1 }; // this wont work
}
```


### Enum definitions

```rust
enum Option<T> {
	Some(T),
	None
}

enum Result<T, E> {
	Ok(T),
	Err(E)
}
```

### Method definitions

```rust
struct Point<T> {
	x: T,
	y: T,
}

impl<T> Point<T> {
	fn x(&self) -> &T {
		&self.x
	}
}

fn main() {
	let p = Point { x: 1, y: 2 };
	println!("{}", p.x());
}
```

## Performace of code using generics

- `monomorphization` is the process of turning generic code into specific code
    - by filling in the concrete types that are used when compiled

- thus no runtime cost for using generics

# Traits -> Defining shared behaviour

- trait informs Rust about functionality a particular type has and can share with other types
- used to define shared behaviour in abstract way
- **trait bounds** can be used to specify that a generic type can be any type that has certain behaviour

## Defining a trait

```rust
pub trait Summary {
	fn summarize(&self) -> String;
}
```

- `trait` keyword -> declares trait
- within `{}` we define the method _signatures_ that describes the behaviour of types that implement this trait
- each type implementing this trait must provide its own custom behaviour for the method body
- a trait can have multiple methods


## Implementing a trait on a type

```rust
pub struct NewsArticle {
	pub headline: String,
	pub location: String,
	pub author: String,
	pub content: String,
}

impl Summary for NewsArticle {
	fn summarize(&self) -> String {
		format!("{} by {} -> ", self.headline, self.author)
	}
}

pub struct Tweet {
 pub username: String,
 pub tweet: String,
 pub reply: bool,
 pub retweet: bool,
}

impl Summary for Tweet {
	fn summarize(&self) -> String {
		format!("{} tweeted {}", self.username, self.tweet)
	}
}
```

- If we want to use a trait written inside a crate ( say ` aggregator` in `lib.rs` )
    - first things we need to bring the trait into scope like: `aggregator::summarize`
    - for this we need the trait as `pub`

- One *restriction with traits*: we can implement a trait on a type only if
	- either the trait or the type is local to our crate
 - This restriction is part of the property called: **Coherence**
     - aka _orphan rule_ -> parent type is not present

## Default implementations

- instead of just method signature we can implement a default behaviour

```rust
pub trait Summary {
	fn summarize(&self) -> String {
		String::from("Read more....")
	}
}
```

- to use this default implementation in a struct say `NewsArticle`

```rust
impl Summary for NewsArticle {}

fn main() {
	println!("Article summary: {}", some_article.summarize());
	// Prints: "Article summary: Read more...."
}
```

- It isnt possible to call the default implementation from an overriding implementation of same method

## Traits as parameters

- using traits to define functions that accept many types

```rust
pub fn notify(item: impl Summary) {
	println!("Holy crap did you see this -> {}", item.summarize());
}
```

- Instead of concrete types, we specify that the param can be any type that implements the specified trait
	- In the above case we can see that item can either be `NewsArticle` or `Tweet`

### Trait bound syntax

```rust
pub fn notify<T: Summary>(item: T) {
	println!("Holy crap did you see this -> {}", item.summarize());
}
```

- this code snipped is equivalent to using traits as params but is verbose & more concise in simple cases
- useful when multiple params are of same type:

```rust
pub fn notify(item1: impl Summary, item2: impl Summary) {
	println!("Holy crap did you see this -> {}", item.summarize());
}

// More consise and readable :)
// constraint that both params must be of same generic type T
pub fn notify<T: Summary>(item1: T, item2: T) {
	println!("Holy crap did you see this -> {}", item.summarize());
}
```

### Specifying multiple trait bounds

```rust
pub notify<T: Summary + Display>(item: T) {
	// ....
}

pub notify(item: impl Summary + Display) {
	// ....
}
```

### Cleared trait bounds with `where` clauses

```rust
fn foo<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
	// ....
}

// can be replaced with
fn foo<T, U>(t: T, u: U) -> i32
	where T: Display + Clone,
          U: Clone + Debug
{
	// .....
}
```

## Returning types that impl traits

- like how we replaced concrete types with types that implement a trait we can do the same for return types

```rust
fn returns_summarizable() -> impl Summary {
	Tweet {
		// ....
	}
}
```

> This concept is important in terms of Closures and Iterators

- Important note: we can use this `impl trait` only if we are returning a single type
    - if `foo()` returns either `NewsArticle` or `Tweet` then compiler would throw err


