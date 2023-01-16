title: Rust

# **Rust**

[**The Rust Programming Language book**](https://rust-book.cs.brown.edu/experiment-intro.html)

[**Rust API Guidelines**](https://rust-lang.github.io/api-guidelines/about.html)

[**The Rustonomicon**](https://doc.rust-lang.org/nomicon/)


---------------------------------------------------------------------------------------------------------------------------

## **C++ differences**

Blocks are expressions.

Move or copy by default.

Const by default.

Mutable references vs const references have the same relationship as read-write lock:
only a single mutable reference is allowed, or multiple const references.

Move is a real move: it is not allowed to use a moved-from variable.


Self is explicit, like in Python, but unlike Python, it is useful --- we specify type modifiers, such as const/mut, ref or move.

Enums can have associated with its fields data/types.





---------------------------------------------------------------------------------------------------------------------------

## **Pattern matching**

**if let**

```rust
} else if let Ok(age) = age {
    if age > 30 {
        println!("Using purple as the background color");
    } else {
        println!("Using orange as the background color");
    }
```


**while let**

```rust
let mut stack = Vec::new();
stack.push(1);
stack.push(2);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```


**Function parameters**

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```


**or**

```rust
match x {
    1 | 2 => println!("one or two"),
```


**Ranges**

```rust
let x = 'c';

match x {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
```

**Structural decomposition**

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    //
    // struct
    //
    let p = Point { x: 0, y: 7 };
    let Point { x, y } = p;
    let Point { x: a, y: b } = p; // <--- if names of variables are different

    let p = Point { x: 0, y: 7 };
    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }

    //
    // Enums:
    //
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
    }
    let msg = Message::Move(0, 160);
    match msg {
        Message::Quit => { ... }
        Message::Move { x, y } => { ... }

    //
    // nested enums
    //
    match msg {
      Message::ChangeColor(Color::Rgb(r, g, b)) => println!(

    //
    // nested structs
    //
    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });

```


**Ignoring unused values**

```rust
// Function argument:
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

// Variable:
let _x = 5;
let y = 10;
```


**Ignoring a part of a value**:

```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {first}, {third}, {fifth}")
    }
}
```


**Ignoring many parts of a value**:

```rust
let numbers = (2, 4, 8, 16, 32);
match numbers {
    (first, .., last) => {
        println!("Some numbers: {first}, {last}");
    }
}

```

**arms can be used with `if`**:

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {:?}", x),
    }
```

**storing matched value in a variable**:

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
        //  ^^^^^^^^^^^^^
        // will be assigned specific value
    } => println!("Found an id in range: {}", id_variable),
```



---------------------------------------------------------------------------------------------------------------------------

## **OOP**

There is no inheritance.





---------------------------------------------------------------------------------------------------------------------------

## **Traits**

Traits are a mix of C++ concepts and CRTP and pure abstract interfaces:

* In the context of templates (aka generics), a trait defines an interface (== concept) and optionally some,
  default implementation (== CRTP).
    * Concepts are compulsory: you will not be able to do much with a template type, unless its traits are specified
      (even if the type has the methods you are trying to call).
    * Traits can have typedefs: `type Item;` in `Iterator`.
* In the context of virtual functions, their interface is similar to pure virtual functions.

Special functions are introduced as traits. For example if you need your type:

* to have a non-trivial destructor, you implement `Drop` trait for your type.
    * `std::mem::drop` can drop (call a destructor) earlier than it would happen otherwise.
* to implement `operator*` (in terms of C++), you implement `Deref` trait for your type.
* to implement `operator+`, use `use std::ops::Add;`.


Traits can "inherit" from other traits. In this case the derived trait says that type implementing it, should also
implement the base trait (which is called super-trait):

```rust
trait OutlinePrint: fmt::Display {      // Any type implementing OutlinePrint, should also implement fmt::Display
    fn outline_print(&self) {
        let output = self.to_string();  // <--- because we are using its functions here
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
    }
}
struct Point {
    x: i32,
    y: i32,
}
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result { write!(f, "({}, {})", self.x, self.y) }
}
impl OutlinePrint for Point {}
```


Rust can choose an appropriate trait implementation based on self type. In the example below, map calls `to_string`
function of `ToString` trait and finds the correct implementation of the trait (`i32`):

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> =
    list_of_numbers.iter().map(ToString::to_string).collect();
```


Similar (explicit) syntax of calling a trait is also useful in case of collision of names of functions:

* Explicit specification if we have an object (`self`) which can be used to determine which function we wanted:

    ```rust
    fn main() {
        let person = Human;
        Pilot::fly(&person);
        Wizard::fly(&person);
        person.fly();
    }
    ```

* Explicit specification if we do not have an object (if the functions are static):

    ```rust
    trait Animal {
        fn baby_name() -> String;
    }
    struct Dog;
    impl Dog {
        fn baby_name() -> String { String::from("Spot") }
    }
    impl Animal for Dog {
        fn baby_name() -> String { String::from("puppy") }
    }
    fn main() {
        println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
        //                                     ^^^^^^^^^^^^^
    }
    ```


---------------------------------------------------------------------------------------------------------------------------

## **Project organisation & Structure & Cargo**

[Cargo documentation](https://doc.rust-lang.org/cargo/index.html).

From smaller units to larger:

1. **Module**: corresponds to a single `.rs` or namespace (it is not necessarily required to create a file. A module can
   be created "in-place").

2. **Crate**: an executable or the (main and only!) library of a package.

    There is a way to maintain two different structures of the code: one better suited for developers and the other for
    external consumers of the library. The idea is to re-export publicly symbols from internal/nested modules in the crate's
    module. [More info](https://rust-book.cs.brown.edu/ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use).

3. **Package**: can contain multiple binary crates and *at most* one library-crate
   ([1](https://doc.rust-lang.org/cargo/reference/cargo-targets.html),
   [2](https://internals.rust-lang.org/t/how-about-changing-lib-to-lib-to-allow-multiple-library-in-a-crate/2022)):

     > The filename defaults to `src/lib.rs`, and the name of the library defaults to the name of the package. A package can 
       have only one library

     So, from the point of view of dependencies, package == library.

     Each package is handled by its `Cargo.toml` and can be published to crates.io.

4. **Workspace**: analogous to a project and consists of multiple (presumably related) packages/libraries.
   [How-to.](https://rust-book.cs.brown.edu/ch14-03-cargo-workspaces.html)


## **References & (smart) pointers**

Unlike C++, references, pointers and smart-pointers are much more similar, and have same usage:

* References has to be dereferenced: `#!rust let y = &x; assert_eq!(5, *y);`
* It allows for interoperability with smart pointers: if we change `y` to be a `Box` the `assert` will still compile fine.





---------------------------------------------------------------------------------------------------------------------------

## **Other**

### Logical const-ness {>>aka mutable<<}

Sometimes there is a field that should be changed on a const object (for example a counter of calls).

In C++, `mutable` or `const_cast<>()` are used. In Rust `RefCell<T>` is the solution: it allows us to get mutable as well as
immutable references, while enforcing borrow-checker invariants in run-time (the app will crash if more than one mutable
reference has been requested).



### ::clone vs .clone vs `operator=`

The idea is that `.clone()` perform a deep-copy. It allows us to quickly find places in the code that can be slow.

Everything else is supposed and designed to be fast (for example, assigning is a copy if it is an small type, such
an integer, or a move otherwise). This is also why Rc pointer is supposed to be copied by

??? info "`::clone`, not `.clone()`"

    > We could have called a.clone() rather than Rc::clone(&a), but Rust’s convention is to use Rc::clone in this case.
    The implementation of Rc::clone doesn’t make a deep copy of all the data like most types’ implementations of clone do.
    The call to Rc::clone only increments the reference count, which doesn’t take much time. By using Rc::clone for
    reference counting, we can visually distinguish between the deep-copy kinds of clones and the kinds of clones that
    increase the reference count.

    > When looking for performance problems in the code, we only need to consider the deep-copy clones and can disregard
    calls to Rc::clone

### Multithreading

There are both: message-passing as well as mutexes.

* Mutex wraps and owns a data-structure.



### Using & Typedefs

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
// Same as:
// template <class T>
// using Result = std::result::Result<T, std::io::Error>;
```







---------------------------------------------------------------------------------------------------------------------------

## **Questions**

* `dyn`.
* enum operations (aka `magic_enum`): get list of values, print to string, etc...
* Range syntax: `(0u32..20)`.

