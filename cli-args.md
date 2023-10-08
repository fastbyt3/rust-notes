# Command Line Arguments

- `std::env::args()` -> returns an iterator of args passed to binary
- we can use `collect` to store the values as a `Vector<String>`

```rust
fn main() {
	let args: Vec<String> = std::env::args().collect();
}
```

- When we run the program either directly as binary or with `cargo run` the first argument will be the binary name

> How to pass CLI args when using cargo run
> `cargo run -- arg1 arg2`


