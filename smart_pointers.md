# Smart Pointers

- Smart pointers are data structures that not only act as pointers
    - but also have additional metadata and capabilities

- provide functionality beyond references
- Smart pointers own the data they point to
- 

- usually implemented as structs 
    - the characteristic that sep smart pointer from ordinary struct -> they implement `Derf` and `Drop` traits
 
- `Derf` trait allows an instance of smart pointer to behave like a reference
    - so we can write code that works with either smart pointers or references

- `Drop` trait alows customizing the code that is run when smart pointer goes out of scope

## Using `Box<T>` to point to data in heap

- Boxes allow you to store data on heap rather than stack
- doesnt have performance overhead other than storing data on heap
- usecases:
    - when we have a type whose size cant be known at compile time
		- and we want to use a value of that type in contenxt requires that exact size
	- when we have large amt of data and we want to transfer ownership
		- but we wanna ensure data wont be copied when we do so
    - when we want to open a value and we care about the type
        - such that it implements a particular trait rather than being of a specified trait

### Storing data in heap

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

- just like other allocations once `b` goes out of scope it will be deallocated

#### Recursive types with Boxes

- Rust doesnt know the size of a recursive type at compile time as it could go on indefinitely
- however boxes have a fixed size so by inserting a box in recursive type definition we can have recursive types

**Cons List**
- DS from Lisp
- aka construct function -> constructs a pair from from its two arguments (usually a single value and another pair) these pairs containing pairs form a list

- cons list has made its way into general functional programming jargon: `to cons x onto y`
    - constructs a new container instance by putting the element `x` at the start of this new container followed by container `y`

- each item in a cons list contains 2 elements: value of current item and next item
- last item in the list contains its next item as `nil` (in case of rust: `null`)

```rust
enum List {
    Cons(i32, List),
    Nil,
}

fn main() {
    let list = List::Cons(1, List::Cons(2, List::Cons(3, List::Nil)));
}
```

- Since we define `Cons(i32, List)` a type of `List` making it recursive -> it holds another value of itself directly

- Using `Box<T>` to get a recursive type with known size
- we have to use `Indirection` -> instead of storing a value directly we have modify the DS to store the value indirectly by storing a pointer to value instead
- Since `Box<T>` is a pointer, Rust compiler knows the space required by it 

## Treating Smart pointers like regular references with `Deref` trait

- `Derf` trait allows customization of dereference operator `*` behaviour

```rust
use std::ops::Deref;

// Tuple struct
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0 // returns reference to value we want [to](to.md) access (first element in tuple)
    }
}
```

- Behind the scenes rust ran: `*(y.deref())` to access the value inside the box


### Implicit Deref Coercions with Functions and Methods

- `Deref coercion` is a convenience that Rust performs on arguments to functions and methods
- converts a reference to a type that implements `Deref` into a reference to a type that `Deref` can convert the original type into
- this prevents most explicit declaration of references(&) and pointers(*) when passing vals to fn

```rust
fn hello(name: &str) {
	println!("Hi {}", name);
}

fn main() {
	let x = Box::new(String::from("Bobby"));
	hello(&x); // Rust calls deref when accessing &x -> converts to &MyBox<T> to &String which gets converted to &str by String's deref
	// If we didnt have deref coercions -> we would have to access this like: hello(&(*x)[..]);
}
```

- `*x` -> pointer to MyBox instance -> we get a String
- `&(*x)[..]` -> we reference the String and use `[..]` to create a slice effectively creating `&str`

#### Deref coercions and mutability

- `DerefMut` trait allows overriding behaviour of `*` over mutable references
- different cases:
	- `&T` to `&U` when `T: Deref<Target=U>`
    - `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
    - `&mut T` to `&U` when `T: Deref<Target=U>`

- In third case Rust forces a mutable reference into an immutable one but this isnt possible the other way around
    - Converting immutable reference into mutable reference requires only 1 immutable reference to that data which cannot be guarenteed


## Running cleanup code with `Drop` trait

> `Drop` trait lets us customize what should happen when a value goes out of scope

- Example usecase:
    - `Box<T>` uses `Drop` to free up the heap space occupied by the value Box points to 

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!(
            "CustomSmartPointer instance with value: {} is dying.... ",
            self.data
        );
    }
}

fn main() {
    let csp = CustomSmartPointer {
        data: String::from("Test data"),
    };

    println!("Current val: {}", csp.data);
}

// Prints:
// Current val: ....
// CustomSmartPointer instance with value: {} is dying.... 
```

- `drop()` need not be called explicitly

### Dropping values early using: `std::mem::drop`

- when to use: when we want to clean up a value early
    - eg: using smart pointers that manage locks -> force the `drop()` so that lock is released and other code can run

- `foo.drop()` call is not allowed so we have to do the same via `std::mem::drop`
    - why -> at end of the fn drop will be called by Rust again -> causes **Double free** error (cleaning up value twice)

```rust
std::mem::drop(foo);
```
