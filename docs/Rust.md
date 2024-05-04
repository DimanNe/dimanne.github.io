title: Rust

# **Rust**

[**The Rust Programming Language book**](https://rust-book.cs.brown.edu/experiment-intro.html)

[**Effective rust**](https://lurklurk.org/effective-rust/cover.html)

[**Rust API Guidelines**](https://rust-lang.github.io/api-guidelines/about.html)

[**The Rustonomicon**](https://doc.rust-lang.org/nomicon/)

[**Rust cookbook**](https://rust-lang-nursery.github.io/rust-cookbook/)

[**The Little Book of Rust Macros**](https://veykril.github.io/tlborm/)


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
## **Project organisation & Structure. Cargo**

* **Get unused dependencies**: `cargo-udeps`
* **Check for security issues, licensing conflicts**: `cargo-deny`

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






--------------------------------------------------------------------------------------------------------------
## **Cargo**

* **Run**: `cargo run --bin digester -- --read-from-file $HOME/file1.txt --log-level trace`
* **Build**: `cargo build`
* **Tests**:
    * Build & run test:
        * **Run specific test with stdout**: `cargo test --release -- --nocapture decoded_reader_zstd_encoder_zstd_decoder_decoded_writer`
        * **Parallel**: `cargo test -- --test-threads=2`
        * **Run ignored**: `cargo test -- --ignored`
        * **Run only non-integration tests**: `cargo test --lib -- --nocapture`





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
## **std types**

Some "notable" combinations of smart-pointers:

* `Rc<RefCell<T>>`: for interior mutability in single-threaded code
* `Arc<Mutex<T>>`: for interior mutability in multithreaded code


#### **Other**

* `std::any`



--------------------------------------------------------------------------------------------------------------
## **Diagnostic & sanitisers**

* [Miri](https://github.com/rust-lang/miri)
* Deadlock detection tools: [no_deadlocks](https://docs.rs/no_deadlocks/latest/no_deadlocks/), ThreadSanitizer,
  [parking_lot::deadlock](https://amanieu.github.io/parking_lot/parking_lot/deadlock/index.html)
* Clippy

#### **Performance**



--------------------------------------------------------------------------------------------------------------
## **Useful crates**

* anyhow error
* [derive_builder](https://docs.rs/derive_builder/latest/derive_builder/)
