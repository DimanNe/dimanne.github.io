title: Rust

# **Rust**

Sources & Books:

* [**The Rust Programming Language book**](https://rust-book.cs.brown.edu/experiment-intro.html)

* [**Asynchronous Programming in Rust**](https://rust-lang.github.io/async-book/)

* [**Effective rust**](https://lurklurk.org/effective-rust/cover.html)

* [**The Rustonomicon**](https://doc.rust-lang.org/nomicon/)

* [**std crate**](https://doc.rust-lang.org/std/index.html)

* [**The Unstable Book**](https://doc.rust-lang.org/unstable-book/the-unstable-book.html)

* [**The Rust Reference**](https://doc.rust-lang.org/reference/introduction.html)

* [**Rust API Guidelines**](https://rust-lang.github.io/api-guidelines/about.html)

* [**Rust cookbook**](https://rust-lang-nursery.github.io/rust-cookbook/)

* [**Rust by Example**](https://doc.rust-lang.org/rust-by-example/index.html)

* [**The Little Book of Rust Macros**](https://veykril.github.io/tlborm/)

* [**Fuzz Book**](https://rust-fuzz.github.io/book/)


--------------------------------------------------------------------------------------------------------------

## **C++ differences**

* Blocks are expressions.
* Move or copy by default.
    * Move is a real move: it is not allowed to use a moved-from variable.
* Const by default.
* Mutable references vs const references have the same relationship as read-write lock: only a single mutable
  reference is allowed, or multiple const references.
* `self` is explicit, like in Python, but unlike Python, it is useful --- we specify type modifiers, such as
  `const`/`mut`, `ref` or move.
* Enums can have associated with its fields data/types (so it is a mix of `enum class` and `std::variant`).





--------------------------------------------------------------------------------------------------------------
## **Performance & Profiling**

* **Get Rust fork of perl flamegraph**: `cargo install inferno`

* **Add debug info into release build**:

    ```ini
    [profile.release]
    debug = true
    ```

* **Record perf data**: `perf record -F 1987 --call-graph dwarf your-app-here`


* **Create flame and icicle SVGs**:

    ```fish
    set bn "v2-F1987.1"; set mw 0.01; set suffix "$mw"; perf script | inferno-collapse-perf > $bn.perf.data.folded; \
    cat $bn.perf.data.folded | inferno-flamegraph --minwidth $mw --width 2250 --title "$bn $suffix" --reverse --inverted > ./$bn-$suffix.perf-icicle.svg && \
    cat $bn.perf.data.folded | inferno-flamegraph --minwidth $mw --width 2250 --title "$bn $suffix" > ./$bn-$suffix.perf-flame.svg
    ```

* **Diff**: `inferno-diff-folded folded1 folded2 | inferno-flamegraph > diff2.svg`



--------------------------------------------------------------------------------------------------------------
## **Cargo: Project structure, building, testing, benchmarking & other tools**

* **Get unused dependencies**: `cargo-udeps`
* **Check for security issues, licensing conflicts**: `cargo-deny`
* **Run**: `cargo run --bin digester -- --read-from-file $HOME/file1.txt --log-level trace`
* **Build**: `cargo build`
* **Tests**:
    * Code coverage: `[cargo-tarpaulin`](https://lib.rs/crates/cargo-tarpaulin)
    * Build & run test:
        * **Run specific test with stdout**: `cargo test --release -- --nocapture decoded_reader_zstd_encoder_zstd_decoder_decoded_writer`
        * **Parallel**: `cargo test -- --test-threads=2`
        * **Run ignored**: `cargo test -- --ignored`
        * **Run only non-integration tests**: `cargo test --lib -- --nocapture`
* [Miri](https://github.com/rust-lang/miri)
* Deadlock detection tools: [no_deadlocks](https://docs.rs/no_deadlocks/latest/no_deadlocks/), ThreadSanitizer,
  [parking_lot::deadlock](https://amanieu.github.io/parking_lot/parking_lot/deadlock/index.html)
* Clippy


#### **Create directories / packages**

[Cargo documentation](https://doc.rust-lang.org/cargo/index.html).

From smaller units to larger:

1. **Module**: corresponds to a single `.rs` or namespace (it is not required to create a file. A
   module can be created "in-place").

2. **Crate**: an executable or the (main and the only!) library of a package.

3. **Package**: can contain multiple binary crates and *at most* one library-crate
   ([1](https://doc.rust-lang.org/cargo/reference/cargo-targets.html),
   [2](https://internals.rust-lang.org/t/how-about-changing-lib-to-lib-to-allow-multiple-library-in-a-crate/2022)),
   this corresponds to C++ library and is the main unit of sharing:

     > The filename defaults to `src/lib.rs`, and the name of the library defaults to the name of the package.
       A package can have only one library

     So, from the point of view of dependencies, package == library.

     Each package is handled by its `Cargo.toml` and can be published to crates.io.

4. **Workspace**: analogous to a project and consists of multiple (presumably related) packages/libraries.
   [How-to.](https://rust-book.cs.brown.edu/ch14-03-cargo-workspaces.html)



From the practical point of view, create several libraries (==directories), like this:

```bash
cargo new input  # will create "input" package (and its directory, along with toml file, src, etc...)
cargo new output # same
cargo new query  # same
```

Then you can put them in a single workspace via this:

```toml title="Cargo.toml"
[workspace]

members = [
   "input",
   "output",
   "query",
]
```

#### **Conditional compilation & Features**

```rust
// Build with `RUSTFLAGS` set to:
//   '--cfg myname="a" --cfg myname="b"'
#[cfg(myname = "a")]
println!("cfg(myname = 'a') is set");
#[cfg(myname = "b")]
println!("cfg(myname = 'b') is set");
```

For example, the following chunk of a manifest file includes six features:

```ini
[features]
default = ["featureA"]
featureA = []
featureB = []
# Enabling `featureAB` also enables `featureA` and `featureB`.
featureAB = ["featureA", "featureB"]
schema = []

[dependencies]
rand = { version = "^0.8", optional = true }
hex = "^0.4"
```


The `rand` crate is a dependency that is marked as `optional = true`, and that effectively makes `"rand"`
into the name of a feature. If the crate is compiled with `--features rand`, then that dependency is activated:

```rust
#[cfg(feature = "rand")]
pub fn pick_a_number() -> u8 {
    rand::random::<u8>()
}

#[cfg(not(feature = "rand"))]
pub fn pick_a_number() -> u8 {
    4 // chosen by fair dice roll.
}
```

So you can determine a crate's features by examining `[features]` as well as optional `[dependencies]` in the
crate's `Cargo.toml` file.

Feature unification means that features should be additive; it's a bad idea to have mutually incompatible
features because there's nothing to prevent the incompatible features being simultaneously enabled by
different users.


#### **Show structure**

```
cargo tree

input v0.1.0 (/home/dimanne/devel/scripts/observability/input)
├── anyhow v1.0.75
├── protobuf v3.2.0
│   ├── bytes v1.5.0
│   ├── once_cell v1.18.0
│   ├── protobuf-support v3.2.0
│   │   └── thiserror v1.0.48
│   │       └── thiserror-impl v1.0.48 (proc-macro)
│   │           ├── proc-macro2 v1.0.67
│   │           │   └── unicode-ident v1.0.12
│   │           ├── quote v1.0.33
│   │           │   └── proc-macro2 v1.0.67 (*)
│   │           └── syn v2.0.37
│   │               ├── proc-macro2 v1.0.67 (*)
│   │               ├── quote v1.0.33 (*)
│   │               └── unicode-ident v1.0.12
│   └── thiserror v1.0.48 (*)
└── protobuf-json-mapping v3.2.0
    ├── protobuf v3.2.0 (*)
    ├── protobuf-support v3.2.0 (*)
    └── thiserror v1.0.48 (*)
```


#### **lib.rs, main.rs: mod, pub, use and other**

* **Start from the crate root**: When compiling a crate, the compiler first looks in the crate root file
  (usually `src/lib.rs` for a library crate or `src/main.rs` for a binary crate) for code to compile.
* **Declaring modules**: In the crate root file, you can declare new modules; say, you declare a “garden”
  module with `mod garden;`. The compiler will look for the module’s code in these places:
    * Inline, within curly brackets that replace the semicolon following `mod garden`
    * In the file `src/garden.rs`
    * In the file `src/garden/mod.rs`
* **Declaring *sub*modules**: In any file other than the crate root, you can declare submodules. For example,
  you might declare `mod vegetables;` in `src/garden.rs`. The compiler will look for the submodule’s code
  within the directory named for the parent module in these places:
    * Inline, directly following `mod vegetables`, within curly brackets instead of the semicolon
    * In the file `src/garden/vegetables.rs`
    * In the file `src/garden/vegetables/mod.rs`
* **Paths to code in modules**: Once a module is part of your crate, you can refer to code in that module from
  anywhere else in that same crate (as long as the privacy rules allow), using the path to the code. For
  example, an `Asparagus` type in the garden vegetables module would be found at
  `crate::garden::vegetables::Asparagus`.
* **Private vs public**: Code within a module is private from its parent modules by default. To make a module
  public, declare it with `pub mod` instead of `mod`. To make items within a public module public as well,
  use `pub` before their declarations.
* **The `use` keyword**: Within a scope, the `use` keyword creates shortcuts to items to reduce repetition of
  long paths. In any scope that can refer to `crate::garden::vegetables::Asparagus`, you can create a shortcut
  with `use crate::garden::vegetables::Asparagus;` and from then on you only need to write `Asparagus` to make
  use of that type in the scope.

    aliases are supported with `use`:

    ```rs
    use std::fmt::Result;
    use std::io::Result as IoResult;
    ```

* **Re-exporting different structure**: Re-exporting is useful when the internal structure of your code is
  different from how programmers calling your code would think about the domain. With pub use, we can write
  our code with one structure but expose a different structure. Doing so makes our library well organized for
  programmers working on the library and programmers calling the library:

    ```rs
    mod front_of_house {
        pub mod hosting {
            pub fn add_to_waitlist() {}
        }
    }

    pub use crate::front_of_house::hosting;
    ```



#### **Benchmarking: cargo bench & criterion**

See [criterion crate](https://crates.io/crates/criterion).

The [cargo bench](https://doc.rust-lang.org/cargo/commands/cargo-bench.html) command runs special test cases
that repeatedly perform an operation, and emits average timing information for the operation.

Example of benchmarking `factorial` function:

```rust
#![feature(test)]
extern crate test;
#[bench]
fn bench_factorial(b: &mut test::Bencher) {
    b.iter(|| {
        let result = factorial(std::hint::black_box(15));
        assert_eq!(result, 1_307_674_368_000);
    });
}
```

#### **Documentation**

`cargo doc` build documentation.



#### **Fuzzing**

[cargo fuzz](https://github.com/rust-fuzz/cargo-fuzz)

See [Fuzz Book](https://rust-fuzz.github.io/book/)








--------------------------------------------------------------------------------------------------------------
## **Enums**

Enum "values" as functions:


name of each enum variant that we define also becomes an initializer function. We can use these initializer
functions as function pointers that implement the closure traits, which means we can specify the initializer
functions as arguments for methods that take closures, like so:

```rust
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```



--------------------------------------------------------------------------------------------------------------
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







--------------------------------------------------------------------------------------------------------------

## **Traits**

Traits are a mix of C++ concepts and CRTP and pure abstract interfaces:

* In the context of templates (aka generics), a trait defines an interface (== concept) and optionally some,
  default implementation (== CRTP).
    * Concepts are compulsory: you will not be able to do much with a template type, unless its traits are
      specified (even if the type has the methods you are trying to call).
    * Traits can have typedefs: `type Item;` in `Iterator`.
* In the context of virtual functions (dyn Trait), trait defines an interface that is similar to pure virtual
  functions.

Special functions are introduced as traits. For example if you need your type:

* to have a non-trivial destructor, you implement `Drop` trait for your type.
    * `std::mem::drop` can drop (call a destructor) earlier than it would happen otherwise.
* to implement `operator*` (in terms of C++), you implement `Deref` trait for your type.
* to implement `operator+`, use `use std::ops::Add;`.


Traits can "inherit" from other traits. In this case the derived trait says that type implementing it, should
also implement the base trait (which is called super-trait):

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


Rust can choose an appropriate trait implementation based on self type. In the example below, map calls
`to_string` function of `ToString` trait and finds the correct implementation of the trait (`i32`):

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> =
    list_of_numbers.iter().map(ToString::to_string).collect();
```


Similar (explicit) syntax of calling a trait is also useful in case of collision of names of functions:

* Explicit specification if we have an object (`self`) which can be used to determine which function we
  wanted:

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


#### **Marker traits**

Sometimes, there is some behaviour that you want to distinguish in the type system, but which cannot be
expressed as some specific method signature in a trait definition. For example, consider a trait for sorting
collections; an implementation might be stable (elements that compare the same will appear in the same order
before and after the sort) but there's no way to express this in the sort method arguments.

In this case, it's still worth using the type system to track this requirement, using a marker trait:

```rust
pub trait Sort {
    /// Re-arrange contents into sorted order.
    fn sort(&mut self);
}

/// Marker trait to indicate that a [`Sortable`] sorts stably.
pub trait StableSort: Sort {}
```

A marker trait has no methods, but an implementation still has to declare that it is implementing the
trait. Code that relies on a stable sort can then specify the `StableSort` trait bound, relying on the
honour system to preserve its invariants. Use marker traits to distinguish behaviours that cannot be
expressed in the trait method signatures.




--------------------------------------------------------------------------------------------------------------
## **References & (smart) pointers**

Unlike C++, references, pointers and smart-pointers are much more similar, and have same usage:

* References has to be dereferenced: `#!rust let y = &x; assert_eq!(5, *y);`
* It allows for interoperability with smart pointers: if we change `y` to be a `Box` the `assert` will still
  compile fine.





--------------------------------------------------------------------------------------------------------------

## **Other**

#### **Logical const-ness {>>aka mutable<<}**

Sometimes there is a field that should be changed on a const object (for example a counter of calls).

In C++, `mutable` or `const_cast<>()` are used. In Rust `RefCell<T>` is the solution: it allows us to get
mutable as well as immutable references, while enforcing borrow-checker invariants in run-time (the app will
crash if more than one mutable reference has been requested).



#### **::clone vs .clone vs `operator=`**

The idea is that `.clone()` perform a deep-copy. It allows us to quickly find places in the code that can be
slow.

Everything else is supposed and designed to be fast (for example, assigning is a copy if it is an small type,
such an integer, or a move otherwise). This is also why Rc pointer is supposed to be copied by

??? info "`::clone`, not `.clone()`"

    > We could have called a.clone() rather than Rc::clone(&a), but Rust’s convention is to use Rc::clone in
      this case. The implementation of Rc::clone doesn’t make a deep copy of all the data like most types’
      implementations of clone do. The call to Rc::clone only increments the reference count, which doesn’t
      take much time. By using Rc::clone for reference counting, we can visually distinguish between the
      deep-copy kinds of clones and the kinds of clones that increase the reference count.

    > When looking for performance problems in the code, we only need to consider the deep-copy clones and
      can disregard calls to Rc::clone

#### **Multithreading**

There are both: message-passing as well as mutexes.

* Mutex wraps and owns a data-structure.



#### **Using & Typedefs**

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
// Same as:
// template <class T>
// using Result = std::result::Result<T, std::io::Error>;
```


#### **Arrays**

* set every value to the same thing with `let x = [val; N]`
* specify each member individually with `let x = [val1, val2, val3]`


--------------------------------------------------------------------------------------------------------------
## **Conversions**

* `char::from_u32`
* `char::from_u32_unchecked`

#### **Almost no implicit conversions**

Rust has very few implicit conversions (coercions). Even save integer conversions must be done explicitly:

```rust
let x: u32 = 2;
let y: u64 = x; // ERROR!
```

#### **But it *might appear* that it has implicit conversions**

But sometimes it *might appear* that it has implicit conversions:

Given these:

```rust
// Integer value from an IANA-controlled range.
#[derive(Clone, Copy, Debug)]
pub struct IanaAllocated(pub u64);

// Need this implementation of From:
impl From<u64> for IanaAllocated {
    fn from(v: u64) -> Self {
        Self(v)
    }
}

// Need also this function that takes a trait bound (template):
pub fn is_iana_reserved<T>(s: T) -> bool
where
    T: Into<IanaAllocated>,
{
    let s = s.into();
    s.0 == 0 || s.0 == 65535
}

```

you can call the last function with just an integer:

```rust
is_iana_reserved(42)
```

#### **Types of casts**

* User defined From / TryFrom
* `as`
* Coercion (implicit conversion)
    * Hardcoded in compiler:
        * mutable reference to a non-mutable references
        * reference to a raw pointer (this isn't unsafe)
        * closure that happens not to capture any variables into a bare function pointer
        * array to a slice
        * concrete item to a trait object
        * item lifetime to a "shorter" one
        * The second coercion of a user-defined type happens when a concrete item is converted to a trait
          object. This operation builds a fat pointer to the item; this pointer is fat because it includes
          both a pointer to the item's location in memory, together with a pointer to the vtable for the
          concrete type's implementation of the trait
    * Using user-defined functions
        * user-defined type implements the `Deref` or the `DerefMut` trait. These traits indicate that the
          user defined type is acting as a smart pointer of some sort

            In particular, a method that expects a reference argument like `&Point` can also be fed a
            `&Box<Point>`, thanks to Deref implemented by `Box`.


#### **Fat pointers & trait objects**

Slice in Rust: `&[u64]` is considered to be a "fat pointer" because it consists two things: start position
and length (aka `std::span`).


The second built-in fat pointer type is a trait object: a reference to some item that implements a particular
trait. It's built from a simple pointer to the item, together with an internal pointer to the type's vtable,
giving a size of 16 bytes (on a 64-bit platform). The vtable for a type's implementation of a trait holds
function pointers for each of the method implementations, allowing dynamic dispatch at runtime.

So, this notation: `&dyn Trait` makes this separation clear (it means the pointer is "fat").


Another interesting peculiarity is that it is impossible to cast an object that implements X interface,
which is itself derived from Y to Y. While you can use functions from both X and Y, you cannot cast it to Y.

Let's say we have this:

```rust
trait Shape: Drawable {
    fn render_in(&self, bounds: Bounds);
    fn render(&self) {
        self.render_in(overlap(SCREEN_BOUNDS, self.bounds()));
    }
}
```

then, a method that accepts a `Shape` trait object:

* can use the methods from `Drawable` (because `Shape` also-implements `Drawable`, and because the relevant
  function pointers are present in the `Shape` vtable)
* but cannot pass/cast the trait object on to another method that expects a `Drawable` trait object (because
  `Shape` is-not Drawable, and because the `Drawable` vtable isn't available).

This is in contrast to generic method that accepts items that implement `Shape`:

* which can use methods from `Drawable`
* and which can pass the item on to another generic method that has a `Drawable` trait bound, because the
  trait bound is monomorphized at compile time to use the Drawable methods of the concrete type.


#### **Deref vs AsRef<T\>**

The Deref traits can't be generic (per-target: `Deref<Target>`) for the destination type. If they were, then
it would be possible for some type `ConfusedPtr` to implement both `Deref<TypeA>` and `Deref<TypeB>`, and
that would leave the compiler unable to deduce a single unique type for an expression like `*x`. So instead
the destination type is encoded as the associated type named Target.

This is in contrast to two other standard pointer traits, the `AsRef` and `AsMut` traits. These traits don't
induce special behaviour in the compiler, but also allow conversions to a reference or mutable reference via
an explicit call to their trait functions (`as_ref()` and `as_mut()` respectively). The destination type for
these conversions is encoded as a type parameter (e.g. `AsRef<Point>`), which means that a single container
type can support multiple destinations.



#### **Borrow vs AsRef<T\> & ToOwned & Cow**

`Borrow` and `BorrowMut` traits each have a single method (`borrow` and `borrow_mut` respectively). This
method has the same signature as the equivalent `AsRef` / `AsMut` trait methods.

The key difference in intents between these traits is visible via the blanket implementations that the
standard library provides. Given an arbitrary Rust reference `&T`, there is a blanket implementation of
both `AsRef` and `Borrow`.

However, `Borrow` also has a blanket implementation for (non-reference) types: `impl<T> Borrow<T> for T`

This means that a method accepting the `Borrow` trait can cope equally with instances of `T` as well as
references-to-T:

```rust
fn add_four<T: std::borrow::Borrow<i32>>(v: T) -> i32 {
    v.borrow() + 4
}
assert_eq!(add_four(&2), 6);
assert_eq!(add_four(2), 6);
```

The standard library's container types have more realistic uses of `Borrow`; for example, `HashMap::get`
uses `Borrow` to allow convenient retrieval of entries whether keyed by value or by reference.

The `ToOwned` trait builds on the `Borrow` trait, adding a `to_owned()` method that produces a new owned
item of the underlying type. This is a generalization of the `Clone` trait: where `Clone` specifically
requires a Rust reference `&T`, `ToOwned` instead copes with things that implement `Borrow`.

This means that:

* A function that operates on references to some type can accept `Borrow` so that it can also be called
  with moved items as well as references.
* A function that operates on owned items of some type can accept `ToOwned` so that it can also be called
  with references to items as well as moved items; any references passed to it will be replicated into a
  locally owned item.

Although it's not a pointer type, it's worth mentioning the `Cow` type at this point, because it provides
an alternative way of dealing with the same kind of situation. `Cow` is an enum that can hold either owned
data, or a reference to borrowed data. A `Cow` input can stay as borrowed data right up to the point where
it needs to be modified, but becomes an owned copy at the point where the data needs to be altered.




--------------------------------------------------------------------------------------------------------------
## **Self-Referential Data Structures & Pin**


Self-Referential Data Structure is a struct that contains a mixture of owned data together with references to
within that owned data:

```rust
struct SelfRef {
   text: String,
   title: Option<&str>, // The slice of `text` that holds the title text.
}
```

The main reason why it does not "just" work is move semantics:

Data structures can move: from the stack to the heap, from the heap to the stack, and from one place to
another. If that happens, the "interior" `title` pointer would no longer be valid, and there's no way to
keep it in sync.

There is one workaround and one proper solution.

A simple alternative for this case is to use the indexing approach; a range of offsets into the text is
not invalidated by a move, and is invisible to the borrow checker because it doesn't involve references:

```rust
struct SelfRefIdx {
   text: String,
   title: Option<Range<usize>>, // Indices into `text` where the title text is.
}
```


However, this indexing approach only works for simple examples. A more general version of the self-reference
problem turns up when the compiler deals with async code. Roughly speaking, the compiler bundles up a
pending chunk of async code into a lambda, and the data for that lambda can include both values and
references to those values.

That's inherently a self-referential data structure, and so async support was a prime motivation for the
`Pin` type in the standard library. This pointer type "pins" its value in place, forcing the value to
remain at the same location in memory, thus ensuring that internal self-references remain valid.

So `Pin` is available as a possibility for self-referential types, but it's tricky to use correctly (as
its official docs make clear):

* The internal reference fields need to use raw pointers, or near relatives (e.g. NonNull) thereof.
* The type being pinned needs to not implement the Unpin marker trait. This trait is automatically
  implemented for almost every type, so this typically involves adding a (zero-sized) field of type
  PhantomPinned to the struct definition.
* The item is only pinned once it's on the heap and held via Pin; in other words, only the contents of
  something like Pin<Box<MyType>> is pinned. This means that the internal reference fields can only be
  safely filled in after this point, but as they are raw pointers the compiler will give you no warning
  if you incorrectly set them before calling `Box::pin`.



--------------------------------------------------------------------------------------------------------------
## **Iterators**

#### **Iterator Trait**

Despite being able to iterate over elements, references to elements and mutable references to elements of
a collection, there is only *one* trait.

Collections that allow iteration over their contents – called iterables – implement the `IntoIterator` trait.
The `into_iter` method of this trait consumes `Self` and emits an Iterator in its stead.

But the catch is that `Self` is either `Vec<_>` or `&Vec<_>` or &mut `Vec<_>`.

The compiler will automatically use this trait for expressions of the form:

```rust
for item in collection {
    // body
}
```



#### **Iterator transformations**

Some of these tranformations affect the overall iteration process:

* `take(n)`, `skip(n)`
* `take_while()` and `skip_while()`: same as above, but based on a predicate.
* `filter(|item| {...})` is the most general version
* `step_by(n)`:emits every n-th item.
* `rev()` reverses the direction of an iterator.
* `cycle()`: converts an iterator that terminates into one that repeats forever, starting at the beginning
  again whenever it reaches the end. (The iterator must support Clone to allow this.)
* `partition`: splits an iterator into two collections based on a predicate

Combining iterators:

* `chain(other)` glues together two iterators, to build a combined iterator that moves through one then
  the other.
* `zip(it)`, `unzip`: zip joins an iterator with a second iterator, to produce a combined iterator that emits pairs of
  items, one from each of the original iterators, until the shorter of the two iterators is finished.
* The `flatten()` method deals with an iterator whose items are themselves iterators, flattening the result.
  On its own, this doesn't seem that helpful, but it becomes much more useful when combined with the
  observation that both `Option` and `Result` act as iterators: they produce either zero
  (for `None`, `Err(e)`) or one (for `Some(v)`, `Ok(v)`) items. This means that flattening a stream of
  `Option` / `Result` values is a simple way to extract just the valid values, ignoring the rest.

Other transformations affect the nature of the Item that's the subject of the Iterator:

* `map(|item| {...})` repeatedly applies a closure to transform each item in turn.
* `cloned()` produces a clone of all of the items in the original iterator; this is particularly useful with
  iterators over &Item references.
* `copied()` produces a copy of all of the items in the original iterator; this is particularly useful
  with iterators over &Item references.
* `enumerate()` converts an iterator over items to be an iterator over (usize, Item) pairs


Finally, iterator consumers:

* Building a single value out of the collection:

    * `sum()`, `product()`
    * `min()`, `max()`
    * `min_by(f)`, `max_by(f)` for finding the extreme values of a collection
    * `reduce(f)` is a more general operation that encompasses the previous methods, building an
      accumulated value of the Item type by running a closure at each step that takes the value accumulated
      so far and the current item.
    * `fold(f)`, `try_fold` is a generalization of `reduce`, allowing the "accumulated value" to be of an arbitrary
      type (not just the `Iterator::Item` type).
    * `scan(f)` generalizes in a slightly different way, giving the closure a mutable reference to some
      internal state at each step.

* Selecting a single value out of the collection:

    * `find(p)`, `try_find` finds the first item that satisfies a predicate
    * `position(p)` also finds the first item satisfying a predicate, but this time it returns the index of
      the item.
    * `nth(n)` returns the n-th element of the iterator, if available.

* Testing against every item in the collection:

    * `any(p)` indicates whether a predicate is true for any item
    * `all(p)` indicates whether a predicate is true for all items

* Accumulate all of the iterated items into a new collection:
    * `collect()`. In addition to transforming an iterator into a new collection, it can also transform
      a sequence of Result's into Result of a sequence:

        ```rust
        let res: Vec<u8> = in.into_iter().map(|v| <u8>::try_from(v)).collect::<Result<Vec<_>, _>>()?;
        ```





--------------------------------------------------------------------------------------------------------------
## **FFI: foreign function interface**

It is possible to call C/C++ function from within Rust (and vice-versa).
See [this](https://lurklurk.org/effective-rust/ffi.html) for general info and
[bindgen](https://lurklurk.org/effective-rust/bindgen.html) for C bindings and [cxx](https://cxx.rs/) for C++.






--------------------------------------------------------------------------------------------------------------
## **Macros**

#### **Declarative Macros**

The [cargo-expand](https://github.com/dtolnay/cargo-expand) tool shows the code that the compiler sees,
after macro expansion:

```rust
macro_rules! inc_item {
    { $x:ident } => { $x.contents += 1; }
}

let mut x = Item { contents: 42 }; // type is not `Copy`
inc_item!(x);
println!("x is {x:?}");
```

```rust
let mut x = Item { contents: 42 };
x.contents += 1;
{
    ::std::io::_print(format_args!("x is {0:?}\n", x));
};
```


#### **Procedural Macros**

There are three distinct types of procedural macro:

* Function-like macros: Invoked with an argument:

    ```rust
    my_func_macro!(15, x + y, f32::consts::PI);
    ```

* Attribute macros: Attached to some chunk of syntax in the program:

    ```rust
    #[log_invocation]
    fn add_three(x: u32) -> u32 {
        x + 3
    }
    ```

* Derive macros: Attached to the definition of a data structure.

    Derive macros add to the input tokens, instead of replacing them altogether. This means that the data
    structure definition is left intact but the macro has the opportunity to append related code.

    Derive macro can declare associated helper attributes, which can then be used to mark parts of the data
    structure that need special processing:

    ```rust
    #[derive(Debug, Deserialize)]
    struct MyData {
        // If `value` is missing when deserializing, invoke
        // `generate_value()` to populate the field instead.
        #[serde(default = "generate_value")]
        value: String,
    }
    ```


--------------------------------------------------------------------------------------------------------------
## **Unsafe Rust**

There is an interesting asymmetry between trust:

* safe code trusts unsafe code (trust is based on assumption that unsafe code was manually checked / verified)
* but, unsafe code cannot trust safe code

    For example, BTreeMap cannot trust that `Ord` trait has been implemented correctly for the key user type.
    While this might counter-intuitive, the reason for this is that a logical error in implementation of
    safe code (`Ord`) leads to a much larger issues in unsafe code. If a faulty `Ord` implementation is used
    in safe code, rust will catch it (in the sense that it will not cause memory-related issues). But this is
    not the case for unsafe code using a faulty implementation of `Ord`.


#### **Exotically Sized Types**

There are two major DSTs (dynamically sized types) exposed by the language:

* trait objects: `dyn MyTrait`
* slices: `[T]`, `str`, and others

See Wide/Fat pointers & trait objects above.

Structs can actually store a single DST directly as their last field, but this makes them a DST as well:

```rust
// Can't be stored on the stack directly
struct MySuperSlice {
    info: u32,
    data: [u8],
}
```

Although such a type is largely useless without a way to construct it. Currently the only properly supported
way to create a custom DST is by making your type generic and performing an unsizing coercion:

```rust
struct MySuperSliceable<T: ?Sized> {
    info: u32,
    data: T,
}
fn main() {
    let sized: MySuperSliceable<[u8; 8]> = MySuperSliceable {
        info: 17,
        data: [0; 8],
    };
    let dynamic: &MySuperSliceable<[u8]> = &sized;
    // prints: "17 [0, 0, 0, 0, 0, 0, 0, 0]"
    println!("{} {:?}", dynamic.info, &dynamic.data);
}
```

Rust also allows types to be specified that occupy no space:


```rust
struct Nothing; // No fields = no size

// All fields have no size = no size
struct LotsOfNothing {
    foo: Nothing,
    qux: (),      // empty tuple has no size
    baz: [u8; 0], // empty array has no size
}
```

Rust also enables types to be declared that cannot even be instantiated. These types can only be talked about
at the type level, and never at the value level. Empty types can be declared by specifying an enum with
no variants:

```rust
enum Void {} // No variants = EMPTY
```


#### **Struct layout types: repr(C), repr(transparent), repr(u*), repr(i*), repr(packed), repr(align(n))**

Use `repr(C)` to pass a struct through FFI or be able to manually control layout (disable Rust tricks,
enabled by default)



#### **Lifetimes**


Lifetime positions can appear as either "input" or "output":

For `fn` definitions, `fn` types, and the traits `Fn`, `FnMut`, and `FnOnce`, input refers to the types of
the formal arguments, while output refers to result types. So fn `foo(s: &str) -> (&str, &str)` has elided
one lifetime in input position and two lifetimes in output position.

There is also the logic that allows to change lifetime from one to another (from longer one, such as
'static to a shorter one). This is called subtyping. And this is why this code compilers:

In almost any context, when you see some requirement of a longer lifetime, like 'static, it is possible to
"convert" it to smaller (because 'static lives as long as a smaller lifetime). This property is called
"being covariant".

But this is wrong for function arguments. Function arguments require that a parameter live at least as long
as given lifetime ('static). Unlike the case above, we can NOT use a parameter with smaller lifetime for
function argument that requires 'static. This property is called "being contravariant".


```rust
fn debug<'a>(a: &'a str, b: &'a str) {
    println!("a = {a:?} b = {b:?}");
}

fn main() {
    let hello: &'static str = "hello";
    {
        let world = String::from("world");
        let world = &world; // 'world has a shorter lifetime than 'static
        debug(hello, world); // hello silently downgrades from `&'static str` into `&'world str`
    }
}
```


We're going to pretend that we're actually allowed to label scopes with lifetimes, and desugar the examples
from the start of this chapter. One particularly interesting piece of sugar is that each let statement
implicitly introduces a scope

Example:

```rust
let x = 0;
let z;
let y = &x;
z = y;

// desugars to:

'a: {
    let x: i32 = 0;
    'b: {
        let z: &'b i32;
        'c: {
            // Must use 'b here because the reference to x is
            // being passed to the scope 'b.
            let y: &'b i32 = &'b x;
            z = y;
        }
    }
}
```


Example: references that outlive referents

```rust
fn as_str(data: &u32) -> &str {
    let s = format!("{}", data);
    &s
}

// desugars to:

fn as_str<'a>(data: &'a u32) -> &'a str {
    'b: {
        let s = format!("{}", data);
        return &'a s;
    }
}
```

This signature of `as_str` takes a reference to a `u32` with some lifetime, and promises that it can produce
a reference to a `str` that can live just as long. Already we can see why this signature might be trouble.
That basically implies that we're going to find a `str` somewhere in the scope the reference to the
`u32` originated in, or somewhere even earlier.

Since the contract of our function says the reference must outlive 'a, that's the lifetime we infer for the
reference. Unfortunately, s was defined in the scope 'b, so the only way this is sound is if 'b contains
'a -- which is clearly false since 'a must contain the function call itself. We have therefore created a
reference whose lifetime outlives its referent.

To make this more clear, we can expand the example:

```rust
fn as_str<'a>(data: &'a u32) -> &'a str {
    'b: {
        let s = format!("{}", data);
        return &'a s
    }
}
fn main() {
    'c: {
        let x: u32 = 0;
        'd: {
            // An anonymous scope is introduced because the borrow does not
            // need to last for the whole scope x is valid for. The return
            // of as_str must find a str somewhere before this function
            // call. Obviously not happening.
            println!("{}", as_str::<'d>(&'d x));
        }
    }
}
```

This is another example that shows why Rust thinks that if a function that takes a reference to X
(`Vec<u8>`) is called, and the function returns a reference to Y (`u8`), then there is a reference to
X in the scope (even if we consider lines after the function).

```rust
let mut data = vec![1, 2, 3];
let x = &data[0];
data.push(4);
println!("{}", x);

'a: {
    let mut data: Vec<i32> = vec![1, 2, 3];
    'b: {
        // 'b is as big as we need this borrow to be
        // (just need to get to `println!`)
        let x: &'b i32 = Index::index::<'b>(&'b data, 0);
        'c: {
            // Temporary scope because we don't need the
            // &mut to last any longer.
            Vec::push(&'c mut data, 4);
        }
        println!("{}", x);
    }
}
```

The problem here is a bit more subtle and interesting. We want Rust to reject this program for the
following reason: We have a live shared reference x to a descendant of data when we try to take a
mutable reference to data to push. This would create an aliased mutable reference, which would violate
the second rule of references.

However this is not at all how Rust reasons that this program is bad. Rust **doesn't understand that x is
a reference to a part of data**. It doesn't understand Vec at all. What it does see is that x has to
live for 'b in order to be printed. The signature of Index::index subsequently demands that the reference
we assign to `data` **has to survive for 'b** (<--- this is exactly what makes rust think that a mutable
reference to vector exists after the function call). When we try to call push, it then sees us try to
make an &'c mut data. Rust knows that 'c is contained within 'b, and rejects our program because the
&'b data reference must still be alive!



#### **Lifetime Elision**

* Each elided lifetime in input position becomes a distinct lifetime parameter.
* If there is exactly one input lifetime position (elided or not), that lifetime is assigned to all elided
  output lifetimes.
* If there are multiple input lifetime positions, but one of them is &self or &mut self, the lifetime of
  self is assigned to all elided output lifetimes.
* Otherwise, it is an error to elide an output lifetime.




#### **Drop Check**

There are some checks performed by rust when a type that holds references is dropped. This code causes
compilation error:

```rust
struct Inspector<'a>(&'a u8);

impl<'a> Drop for Inspector<'a> {
    fn drop(&mut self) {
        println!("I was only {} days from retirement!", self.0);
    }
}

struct World<'a> {
    inspector: Option<Inspector<'a>>,
    days: Box<u8>,
}

fn main() {
    let mut world = World {
        inspector: None,
        days: Box::new(1),
    };
    world.inspector = Some(Inspector(&world.days));
    // Let's say `days` happens to get dropped first.
    // Then when Inspector is dropped, it will try to read free'd memory!
}
```

because drop access the references, which is not guaranteed to live long enough.

If you need circumvent this:

```rust
#![feature(dropck_eyepatch)]

struct Inspector<'a>(&'a u8, &'static str);

unsafe impl<#[may_dangle] 'a> Drop for Inspector<'a> {
    fn drop(&mut self) {
        println!("Inspector(_, {}) knows when *not* to inspect.", self.1);
    }
}

```


#### **PhantomData**

When working with unsafe code, we can often end up in a situation where types or lifetimes are logically
associated with a struct, but not actually part of a field. This most commonly occurs with lifetimes.
For instance, the Iter for &'a [T] is (approximately) defined as follows:

```rust
struct Iter<'a, T: 'a> {
    ptr: *const T,
    end: *const T,
}
```

PhantomData is a special marker type. PhantomData consumes no space, but simulates a field of the given type
for the purpose of static analysis. This was deemed to be less error-prone than explicitly telling the
type-system the kind of variance that you want, while also providing other useful things such as auto
traits and the information needed by drop check.

Iter logically contains a bunch of `&'a T`s, so this is exactly what we tell the PhantomData to simulate:

```rust
use std::marker;
struct Iter<'a, T: 'a> {
    ptr: *const T,
    end: *const T,
    _marker: marker::PhantomData<&'a T>,
}
```

and that's it. The lifetime will be bounded, and your iterator will be covariant over 'a and T.



#### **Splitting Borrows**

borrowck doesn't understand arrays or slices in any way, so this doesn't work:

```rust
let mut x = [1, 2, 3];
let a = &mut x[0];
let b = &mut x[1];
println!("{} {}", a, b);
```

In order to "teach" borrowck that what we're doing is ok, we need to drop down to unsafe code. For instance,
mutable slices expose a `split_at_mut` function that consumes the slice and returns two mutable slices.
One for everything to the left of the index, and one for everything to the right. Intuitively we know this is
safe because the slices don't overlap, and therefore alias. However the implementation requires some unsafety:

```rust
pub fn split_at_mut(&mut self, mid: usize) -> (&mut [T], &mut [T]) {
    let len = self.len();
    let ptr = self.as_mut_ptr();

    unsafe {
        assert!(mid <= len);

        (from_raw_parts_mut(ptr, mid),
         from_raw_parts_mut(ptr.add(mid), len - mid))
    }
}
```


#### **The Dot Operator**

Suppose we have a function `foo` that has a receiver (a `self`, `&self` or `&mut self` parameter). If we
call `value.foo()`, the compiler needs to determine what type `Self` is before it can call the correct
implementation of the function. For this example, we will say that `value` has type `T`.

We will use fully-qualified syntax to be more clear about exactly which type we are calling a function on.

* First, the compiler checks if it can call `T::foo(value)` directly. This is called a "by value" method call.
* If it can't call this function (for example, if the function has the wrong type or a trait isn't
  implemented for `Self`), then the compiler tries to add in an automatic reference. This means that the
  compiler tries `<&T>::foo(value)` and `<&mut T>::foo(value)`. This is called an "autoref" method call.
* If none of these candidates worked, it dereferences `T` and tries again. This uses the `Deref` trait - if
  T: `Deref<Target = U>` then it tries again with type `U` instead of `T`. If it can't dereference `T`, it
  can also try unsizing `T`. This just means that if T has a size parameter known at compile time, it
  "forgets" it for the purpose of resolving methods. For instance, this unsizing step can convert
  `[i32; 2]` into `[i32]` by "forgetting" the size of the array.


Example of the method lookup algorithm:

* Let's say we have this:

    ```rust
    let array: Rc<Box<[T; 3]>> = ...;
    let first_entry = array[0];
    ```

* First, `array[0]` is really just syntax sugar for the `Index` trait - the compiler will convert `array[0]`
  into `array.index(0)`. Now, the compiler checks to see if array implements `Index`

* The compiler checks if `Rc<Box<[T; 3]>>` implements `Index`, but it does not, and neither do
  `&Rc<Box<[T; 3]>>` or `&mut Rc<Box<[T; 3]>>`.
* Since none of these worked, the compiler dereferences the `Rc<Box<[T; 3]>>` into `Box<[T; 3]>` and tries
  again. `Box<[T; 3]>`, `&Box<[T; 3]>`, and `&mut Box<[T; 3]>` do not implement `Index`,
* So it dereferences again. `[T; 3]` and its autorefs also do not implement `Index`. It can't
  dereference `[T; 3]`, so the compiler unsizes it, giving `[T]`. Finally, `[T]` implements `Index`, so it
  can now call the actual index function.


#### **Transmutes** (aka reinterpret_cast)


`mem::transmute<T, U>` takes a value of type `T` and reinterprets it to have type `U`. The only restriction
is that the `T` and `U` are verified to have the same size. The ways to cause Undefined Behavior with this
are mind boggling.

* First and foremost, creating an instance of any type with an invalid state is going to cause arbitrary
  chaos that can't really be predicted. Do not transmute 3 to bool. Even if you never do anything with
  the bool. Just don't.

* Transmute has an overloaded return type. If you do not specify the return type it may produce a surprising
  type to satisfy inference.

* Transmuting an `&` to `&mut` is Undefined Behavior. While certain usages may appear safe, note that the
  Rust optimizer is free to assume that a shared reference won't change through its lifetime and thus such
  transmutation will run afoul of those assumptions.

* When transmuting between different compound types, you have to make sure they are laid out the same way!
  If layouts differ, the wrong fields are going to get filled with the wrong data, which will make you
  unhappy and can also be Undefined Behavior (see above).

    So how do you know if the layouts are the same? For `repr(C)` types and `repr(transparent)` types, layout
    is precisely defined. But for your run-of-the-mill `repr(Rust)`, it is not. Even different instances of
    the same generic type can have wildly different layout. `Vec<i32>` and `Vec<u32>` *might* have their
    fields in the same order, or they might not.


#### **Working With Uninitialized Memory**


Like C, all stack variables in Rust are uninitialized until a value is explicitly assigned to them.
Unlike C, Rust statically prevents you from ever reading them until you do. Interestingly, Rust doesn't
require the variable to be mutable to perform a delayed initialization.

So this compiles:

```rust
fn main() {
    let x: i32;
    if true {
        x = 1;
    } else {
        x = 2;
    }
    println!("{}", x);
}
```


There is an interesting case for partially initialised arrays: Safe Rust doesn't permit you to partially
initialize an array.

Unsafe Rust gives us a powerful tool to handle this problem:
[`MaybeUninit`](https://doc.rust-lang.org/core/mem/union.MaybeUninit.html). This type can be used to handle
memory that has not been fully initialized yet. Example of usage:

```rust
use std::mem::{self, MaybeUninit};

// Size of the array is hard-coded but easy to change (meaning, changing just the constant is
// sufficient). This means we can't use [a, b, c] syntax to initialize the array, though, as we would
// have to keep that in sync with `SIZE`!
const SIZE: usize = 10;

let x = {
    // Create an uninitialized array of `MaybeUninit`. The `assume_init` is safe because the type we are
    // claiming to have initialized here is a bunch of `MaybeUninit`s, which do not require initialization.
    let mut x: [MaybeUninit<Box<u32>>; SIZE] = unsafe {
        MaybeUninit::uninit().assume_init()
    };

    // Dropping a `MaybeUninit` does nothing. Thus using raw pointer assignment instead of `ptr::write`
    // does not cause the old uninitialized value to be dropped.
    for i in 0..SIZE {
        x[i] = MaybeUninit::new(Box::new(i as u32));
    }

    // Everything is initialized. Transmute the array to the initialized type.
    unsafe { mem::transmute::<_, [Box<u32>; SIZE]>(x) }
};

dbg!(x);
```

It's worth spending a bit more time on the loop in the middle, and in particular the assignment operator
and its interaction with drop. If we wrote something like:

```rust
*x[i].as_mut_ptr() = Box::new(i as u32); // WRONG!
```

we would actually overwrite a `Box<u32>`, leading to drop of uninitialized data, which would cause much
sadness and pain.

The correct alternative, if for some reason we cannot use `MaybeUninit::new`, is to use the `ptr` module.
In particular, it provides three functions that allow us to assign bytes to a location in memory without
dropping the old value: `write`, `copy`, and `copy_nonoverlapping`.

* `ptr::write(ptr, val)` takes a val and moves it into the address pointed to by ptr.
* `ptr::copy(src, dest, count)` copies the bits that count T items would occupy from src to dest. (this
  is equivalent to C's memmove -- note that the argument order is reversed!)
* `ptr::copy_nonoverlapping(src, dest, count)` does what copy does, but a little faster on the assumption
  that the two ranges of memory don't overlap. (this is equivalent to C's memcpy -- note that the argument
  order is reversed!)

It's worth noting that you don't need to worry about ptr::write-style shenanigans with types which don't
implement Drop or contain Drop types, because Rust knows not to try to drop them. This is what we relied
on in the above example.

Note that, to use the ptr methods, you need to first obtain a raw pointer to the data you want to initialize.
It is illegal to construct a reference to uninitialized data, which implies that you have to be careful when
obtaining said raw pointer:

* For an array of T, you can use `base_ptr.add(idx)` where `base_ptr: *mut T` to compute the address of array
  index idx. This relies on how arrays are laid out in memory.
* For a struct, however, in general we do not know how it is laid out, and we also cannot use
  `&mut base_ptr.field` as that would be creating a reference. So, you must carefully use the `addr_of_mut`
  macro. This creates a raw pointer to the field without creating an intermediate reference:

    ```rust
    use std::{ptr, mem::MaybeUninit};

    struct Demo {
        field: bool,
    }

    let mut uninit = MaybeUninit::<Demo>::uninit();
    // `&uninit.as_mut().field` would create a reference to an uninitialized `bool`,
    // and thus be Undefined Behavior!
    let f1_ptr = unsafe { ptr::addr_of_mut!((*uninit.as_mut_ptr()).field) };
    unsafe { f1_ptr.write(true); }

    let init = unsafe { uninit.assume_init() };
    ```





--------------------------------------------------------------------------------------------------------------
## **std types**

Some "notable" combinations of smart-pointers:

* `Rc<RefCell<T>>`: for interior mutability in single-threaded code
* `Arc<Mutex<T>>`: for interior mutability in multithreaded code


#### **Other**

* `std::any`





--------------------------------------------------------------------------------------------------------------
## **Useful crates**

* anyhow error
* [derive_builder](https://docs.rs/derive_builder/latest/derive_builder/)
