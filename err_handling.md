# Error handling

- `Recoverable` and `Unrecoverable` errors
    - recoverable -> file not found -> allow retry
    - unrecoverable -> access location beyond end of arr

- For recvorable => `Resut<T, E>`
- In Unrecoverable => `panic!()` macro

## Understanding what happens when `panic!`

- program starts **Unwinding**
- walks back up stack and cleans up data from each fn
- if we abort immediately -> role of OS to clean up the used mem
- to abort immediately: 

    ```toml
    [profile.release]
    panic = 'abort'
    ```

## Recoverable errors with Result

```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

- 2 variants
    - `Ok(T)` -> type of value in case of success
    - `Err(E)` -> type of err in case of err

- if we try to open a file with `std::fs::File`

```rust
fn main() {
	let f = File::open("test.txt");

	let f = match f {
		Ok(file) => file,
		Err(error) => panic!("Error encountered in opening file: {}", error)
	};
}
```

- matching on different errors:

```rust
fn main() {
	let f = File::open("test.txt");

	let f = match f {
		Ok(file) => file,
		Err(error) => match error.kind() {
			ErrorKind::NotFound => match File::create("test.txt") {
				Ok(created_file) => created_file,
				Err(e) => panic!("Couldnt create file: {}", e),
			},
			other_err => panic!("Error encountered in opening file: {}", error),
		},
	};
}
```

```rust
// THE RUST WAY
fn main() {
	let f = File::open("test.txt").unwrap_or_else(|error| {
		if error.kind() == ErrorKind::NotFound {
			File::Create("test.txt").unwrap_or_else(|error| {
				panic!("Error creating file, {}", error);
			})
		} else {
			panic!("Error opening file: {}", error);
		}
	});
}
```

### Useful methods to utilize instead of `match`

#### `unwrap`

- If Result value is `Ok<T>` variant then unwrap will return the value inside `Ok`
- else if the value of Result is `Err` then unwrap will call `panic!`

#### `expect`

- `expect` is meant to be used in same manner as `unwrap`
- but its focus is to provide helpful error information which is passed on to `panic!`

```rust
let f = File::open("test.txt").unwrap();

let f = File::open("test.txt").expect("failed to open test.txt");
```

## Propagating errors

```rust
fn read_from_file() -> ResultString<String, io::Error> {
	let f = File::open("test.txt");

	let mut f = match f {
		Ok(file) => file,
		Err(e) => return Err(e),
	};

	let mut s = String::new();

	// Return string / err
	match f.read_to_string(&mut s) {
		Ok(_) => Ok(s),
		Err(e) => Err(e),
	}
}
```

### Shortcut to propagating errors: using `?` operator

- `?` placed after a `Result` value is defined to work in _almost_ the same way as `match` expr
- If value is `Ok` variant then its value is returned
- Else if the value is `Err` the `Err` will be returned

- Main difference between `?` and `match`:
    - error values go through `from` function (`From` trait)
    - convert one Error types into another

```rust
fn read_from_file() -> ResultString<String, io::Error> {
	let mut s = String::new();

	File::open("test.txt")?.read_to_string(&mut s)?;

	Ok(s)
}
```

> `?` can be used only in Functions that return `Result<T, E>`

- In case of `main` function:
    - there are restrictions on the return type
    - one valid type is `()`
    - another is `Result<T, E>`

	```rust
	// Box<dyn Error> -> trait obj (any kind of err)
	fn main() -> Result<(), Box<dyn Error>> {
		let f = File::open("text.txt")>;

		Ok(())
	}
	```

## When to panic and when not to

- Some scenarios that are appropirate for panics are _prototype code_, _tests_ and _examples_
- In other cases where humans have more info than compiler doesnt warrent need for panic

## Guidelines for err validation

### Creating custom types for validation

- In case of guessing game problem if we have to check if the user input is always in the range of 1-100
- it becomes tedious and might impact performace too
- So we can make a new type that has the validation (num in range 1 to 100) within a function of that type

```rust
pub struct Guess {
	value: i32,
}

impl Guess {
	pub fn new(n: u32) -> Guess {
		if (n < 1 || n > 100) {
			panic!("Value must be between 1 and 100");
		}

		Guess { n }
	}
	
	// This pub method is necessary since value of Guess is not public
	pub fn value(&self) -> i32 {
		self.value
	}
}
```

## Using `map_err`

- Maps a `Result<T, E>` to `Result<T, F>` by applying a fallback function to transform `E` to `F`
- leaves `Ok` value untouched

```rust
fn stringify(x: u32) -> String { format!("error code: {x}") }

fn main() {
	let x: Result<u32, u32> = Ok(2);
	assert_eq!(x.map_err(stringify), Ok(2));

	let y: Result<u32, u32> = Err(2);
	assert_eq!(y.map_err(stringify), Err("error code: 2").to_string());
}
```
