title: C++

# **C++**

## **Talks & Articles**

### `signed` vs `unsigned int`'s --- performance-wise
[CppCon 2016: Chandler Carruth](https://youtu.be/yG1OZ69H_-o?t=3094)


### Non-constant constant-exspressions in C++

```cpp linenums="1"
// <insert solution here>
int main () {
   constexpr int a = f ();
   constexpr int b = f ();

   static_assert (a != b, "fail");
}
```

[Solution](https://b.atch.se/posts/non-constant-constant-expressions/).



### When fences and weak ordering needed

[CppCon 2016: Hans Boehm “Using weakly ordered C++ atomics correctly"](https://www.youtube.com/watch?v=M15UKpNlpeM&ab_channel=CppCon)



### C++17 overview

[Here](https://jfbastien.com/what-is-cpp17/#/)



### C++ and exceptions

[reddit](https://www.reddit.com/r/cpp/comments/cliw5j/should_not_exceptions_be_finally_deprecated/)





## **Tricky parts**

### Reference collapsing rules
A good explanation of [rvalues](http://thbecker.net/articles/rvalue_references/section_01.html).

Geven:

```cpp linenums="1"
template<typename T> 
void foo(T&&);
```

There is a special template argument deduction rule for function templates that take an argument by
rvalue reference to a template argument:

* When `foo()` is called on an **lvalue of type `A`**, then **`T` resolves to `A&`** and hence, by
  the reference collapsing rules above, the argument type effectively becomes `A&`.
* When `foo()` is called on an **rvalue of type `A`**, then **`T` resolves to `A`**, and hence
  the argument type becomes `A&&`.


### auto vs decltype

[Src](http://thbecker.net/articles/auto_and_decltype/section_01.html).

`rvalue` is an `xvalue` if it is one of the following:

* A function call where the function's return value is declared as an `rvalue` reference.
  An example would be `std::move(x)`.
* A cast to an `rvalue` reference. An example would be `static_cast<A&&>(a)`.
* A member access of an `xvalue`. Example: `(static_cast<A&&>(a)).m_x`.

All other `rvalues` are `prvalues`. We are now in a position to describe how `decltype` deduces
the type of a complex expression.

Let `expr` be an expression that is not a plain, unparenthesized variable, function parameter, or
class member access. Let `T` be the type of `expr`.

* If `expr` is an `lvalue`, then `decltype(expr)` is `T&`.
* If `expr` is an `xvalue`, then `decltype(expr)` is `T&&`.
* Otherwise, `expr` is a `prvalue`, and `decltype(expr)` is `T`.



### RVO / URVO / NRVO

Consider the following:

```cpp linenums="1"
class TString {
  public:
    TString()                           { std::cout << "ctor()"       << std::endl; }
    TString(const char *)               { std::cout << "ctor(char *)" << std::endl; }
    TString(const TString &)            { std::cout << "ctor const &" << std::endl; }
    TString &operator=(const TString &) { std::cout << "= const &"    << std::endl; return *this; }
    TString(TString &&)                 { std::cout << "ctor &&"      << std::endl; }
    TString &operator=(TString &&)      { std::cout << "= &&&"        << std::endl; return *this; }
};

TString Meow() {
  return "Meow";

  // ------ or ------
  return TString("Meow");

  // ------ or ------
  TString a("Meow");
  return a;
}

int main() {
  TString s = Meow();
  return 0;
}
```

The output is just
```
ctor(char *)
```

As you can see there is no things like: (1) construct temporary string in the function `Meow`,
(2) construct string `s` in `main`, (3) call `s.operator=`, (4) destruct temporary string (constructed in `Meow`).

Instead of it just a single ctor called.





## **Lesser known C++ std algorithms**
[Full list](https://en.cppreference.com/w/cpp/algorithm).

### Uncategorised
* [`slice`](https://en.cppreference.com/w/cpp/numeric/valarray/slice), [`valarray`](https://en.cppreference.com/w/cpp/numeric/valarray)


### Non-modifying sequence operations

* [`all_of`, `any_of`, `none_of`](https://en.cppreference.com/w/cpp/algorithm/all_any_none_of)
* [`count_if`](https://en.cppreference.com/w/cpp/algorithm/count)
* [`adjacent_find`](https://en.cppreference.com/w/cpp/algorithm/adjacent_find)


### Modifying sequence operations

* [`remove_if`](https://en.cppreference.com/w/cpp/algorithm/remove), [`replace_if`](https://en.cppreference.com/w/cpp/algorithm/replace)
* [`swap_ranges`](https://en.cppreference.com/w/cpp/algorithm/swap_ranges)


### Sort related
* [`merge`](https://en.cppreference.com/w/cpp/algorithm/merge), [`inplace_merge`](https://en.cppreference.com/w/cpp/algorithm/inplace_merge)
* [`partial_sort`](https://en.cppreference.com/w/cpp/algorithm/partial_sort)
* [`nth_element`](https://en.cppreference.com/w/cpp/algorithm/nth_element)
* [`partition`](https://en.cppreference.com/w/cpp/algorithm/partition), [`stable_partition`](https://en.cppreference.com/w/cpp/algorithm/stable_partition)
* [`is_partitioned`](https://en.cppreference.com/w/cpp/algorithm/is_partitioned), [`partition_point`](https://en.cppreference.com/w/cpp/algorithm/partition_point)


### Minimum/maximum
* [`minmax`](https://en.cppreference.com/w/cpp/algorithm/minmax), [`minmax_element`](https://en.cppreference.com/w/cpp/algorithm/minmax_element)





## [**C++ Core Guidelines**](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md) --- excerpts



### C.138: Create an overload set for a derived class and its bases with `using`

##### Reason
Without a `using` declaration, member functions in the derived class hide the entire inherited overload sets.

```cpp linenums="1" title="Example, bad"
#include <iostream>
class B {
public:
    virtual int f(int i) { std::cout << "f(int): "; return i; }
    virtual double f(double d) { std::cout << "f(double): "; return d; }
    virtual ~B() = default;
};
class D: public B {
public:
    int f(int i) override { std::cout << "f(int): "; return i + 1; }
};
int main()
{
    D d;
    std::cout << d.f(2) << '\n';   // prints "f(int): 3"
    std::cout << d.f(2.3) << '\n'; // prints "f(int): 3"
}
```

```cpp linenums="1" title="Example, good"
class D: public B {
public:
    int f(int i) override { std::cout << "f(int): "; return i + 1; }
    using B::f; // exposes f(double)
};
```

##### Note
This issue affects both virtual and non-virtual member functions

For variadic bases, C++17 introduced a variadic form of the using-declaration,

```cpp linenums="1"
template<class... Ts>
struct Overloader : Ts... {
    using Ts::operator()...; // exposes operator() from every base
};
```



### C.165: Use `using` for customization points

##### Reason
To find function objects and functions defined in a separate namespace to "customize" a common function.

##### Example
Consider `swap`. It is a general (standard-library) function with a definition that will work for just about
any type. However, it is desirable to define specific `swap()`'s for specific types. For example, the general
`swap()` will copy the elements of two vectors being swapped, whereas a good specific implementation will not
copy elements at all.

```cpp linenums="1" title="bad"
namespace N {
    My_type X { /* ... */ };
    void swap(X&, X&);   // optimized swap for N::X
    // ...
}

void f1(N::X& a, N::X& b)
{
    std::swap(a, b);   // probably not what we wanted: calls std::swap()
}
```

The `std::swap()` in `f1()` does exactly what we asked it to do: it calls the `swap()` in namespace `std`.
Unfortunately, that's probably not what we wanted. How do we get `N::X` considered?

```cpp linenums="1" title="better"
void f2(N::X& a, N::X& b) {
    swap(a, b);   // calls N::swap
}
```

But that might not be what we wanted for generic code. There, we typically want the specific function if it exists
and the general function if not. This is done by including the general function in the lookup for the function:

```cpp linenums="1"  title="good"
void f3(N::X& a, N::X& b) {
    using std::swap;  // make std::swap available
    swap(a, b);       // calls N::swap if it exists, otherwise std::swap
}
```




### C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance

##### Reason
Implementation details in an interface make the interface brittle; that is, make its users vulnerable to having
to recompile after changes in the implementation. Data in a base class increases the complexity of implementing
the base and can lead to replication of code.

##### Note

`Interface inheritance`:

:   is the use of inheritance to separate users from implementations, in particular to allow derived classes to
    be added and changed without affecting the users of base classes.

`Implementation inheritance`:
:   is the use of inheritance to simplify implementation of new facilities by making useful operations available
    for implementers of related new operations (sometimes called "programming by difference").

A pure interface class is simply a set of pure virtual functions; see I.25.

In early OOP (e.g., in the 1980s and 1990s), implementation inheritance and interface inheritance were often mixed
and bad habits die hard. Even now, mixtures are not uncommon in old code bases and in old-style teaching material.

The importance of keeping the two kinds of inheritance increases

* with the size of a hierarchy (e.g., dozens of derived classes),
* with the length of time the hierarchy is used (e.g., decades), and
* with the number of distinct organizations in which a hierarchy is used (e.g., it can be difficult to distribute an update to a base class)

```cpp linenums="1" title="Example, bad:"
class Shape {   // BAD, mixed interface and implementation
public:
    Shape();
    Shape(Point ce = {0, 0}, Color co = none): cent{ce}, col {co} { /* ... */}

    Point center() const { return cent; }
    Color color() const { return col; }

    virtual void rotate(int) = 0;
    virtual void move(Point p) { cent = p; redraw(); }

    virtual void redraw();

    // ...
private:
    Point cent;
    Color col;
};

class Circle : public Shape {
public:
    Circle(Point c, int r) : Shape{c}, rad{r} { /* ... */ }

    // ...
private:
    int rad;
};

class Triangle : public Shape {
public:
    Triangle(Point p1, Point p2, Point p3); // calculate center
    // ...
};
```

Problems:

* As the hierarchy grows and more data is added to Shape, the constructors get harder to write and maintain.
* Why calculate the center for the Triangle? we might never use it.
* Add a data member to Shape (e.g., drawing style or canvas) and all classes derived from Shape and all code
  using Shape will need to be reviewed, possibly changed, and probably recompiled.

The implementation of `#!cpp Shape::move()` is an example of implementation inheritance: we have defined
`move()` once and for all for all derived classes. The more code there is in such base class member function
implementations and the more data is shared by placing it in the base, the more benefits we gain --- and the
less stable the hierarchy is.

This `Shape` hierarchy can be rewritten using interface inheritance:

```cpp linenums="1"
class Shape {  // pure interface
public:
    virtual Point center() const = 0;
    virtual Color color() const = 0;

    virtual void rotate(int) = 0;
    virtual void move(Point p) = 0;

    virtual void redraw() = 0;

    // ...
};
```

Note that a pure interface rarely has constructors: there is nothing to construct.

```cpp linenums="1"
class Circle : public Shape {
public:
    Circle(Point c, int r, Color c) : cent{c}, rad{r}, col{c} { /* ... */ }

    Point center() const override { return cent; }
    Color color() const override { return col; }

    // ...
private:
    Point cent;
    int rad;
    Color col;
};
```

The interface is now less brittle, but there is more work in implementing the member functions.
For example, center has to be implemented by every class derived from Shape.


##### Dual hierarchy

How can we gain the benefit of stable hierarchies from implementation hierarchies and the benefit of
implementation reuse from implementation inheritance? One popular technique is dual hierarchies. There
are many ways of implementing the idea of dual hierarchies; here, we use a multiple-inheritance variant.

First we devise a hierarchy of interface classes:

```cpp linenums="1"
class Shape {   // pure interface
public:
    virtual Point center() const = 0;
    virtual Color color() const = 0;

    virtual void rotate(int) = 0;
    virtual void move(Point p) = 0;

    virtual void redraw() = 0;

    // ...
};

class Circle : public virtual Shape {   // pure interface
public:
    virtual int radius() = 0;
    // ...
};
```

To make this interface useful, we must provide its implementation classes (here, named equivalently, but
in the `Impl` namespace):

```cpp linenums="1"
class Impl::Shape : public virtual ::Shape { // implementation
public:
    // constructors, destructor
    // ...
    Point center() const override { /* ... */ }
    Color color() const override { /* ... */ }

    void rotate(int) override { /* ... */ }
    void move(Point p) override { /* ... */ }

    void redraw() override { /* ... */ }

    // ...
};
```

Now Shape is a poor example of a class with an implementation, but bear with us because this is just a simple
example of a technique aimed at more complex hierarchies.

```cpp linenums="1"
class Impl::Circle : public virtual ::Circle, public Impl::Shape {   // implementation
public:
    // constructors, destructor

    int radius() override { /* ... */ }
    // ...
};
```

And we could extend the hierarchies by adding a `Smiley` class ( `:-)` ):

```cpp linenums="1"
class Smiley : public virtual Circle { // pure interface
public:
    // ...
};

class Impl::Smiley : public virtual ::Smiley, public Impl::Circle {   // implementation
public:
    // constructors, destructor
    // ...
}
```

There are now two hierarchies:

* interface: `Smiley` -> `Circle` -> `Shape`
* implementation: `Impl::Smiley` -> `Impl::Circle` -> `Impl::Shape`

Since each implementation is derived from its interface as well as its implementation base class we get a lattice (DAG):

```
Smiley     ->         Circle     ->  Shape
  ^                     ^               ^
  |                     |               |
Impl::Smiley -> Impl::Circle -> Impl::Shape
```

As mentioned, this is just one way to construct a dual hierarchy.

The implementation hierarchy can be used directly, rather than through the abstract interface.

```cpp linenums="1"
void work_with_shape(Shape&);

int user() {
    Impl::Smiley my_smiley{ /* args */ };   // create concrete shape
    // ...
    my_smiley.some_member();        // use implementation class directly
    // ...
    work_with_shape(my_smiley);     // use implementation through abstract interface
    // ...
}
```

This can be useful when the implementation class has members that are not offered in the abstract interface or if direct
use of a member offers optimization opportunities (e.g., if an implementation member function is final).

##### Note
Another (related) technique for separating interface and implementation is Pimpl.

##### Note
There is often a choice between offering common functionality as (implemented) base class functions and free-standing
functions (in an implementation namespace). Base classes gives a shorter notation and easier access to shared data
(in the base) at the cost of the functionality being available only to users of the hierarchy.



### R.37: Do not pass a pointer or reference obtained from an aliased smart pointer

##### Reason
Violating this rule is the number one cause of losing reference counts and finding yourself with a dangling pointer.
Functions should prefer to pass raw pointers and references down call chains. At the top of the call tree where you
obtain the raw pointer or reference from a smart pointer that keeps the object alive. You need to be sure that the
smart pointer cannot inadvertently be reset or reassigned from within the call tree below.

##### Note
To do this, sometimes you need to take a local copy of a smart pointer, which firmly keeps the object alive for the
duration of the function and the call tree.

##### Example

Consider this code:

```cpp linenums="1"
// global (static or heap), or aliased local ...
shared_ptr<widget> g_p = ...;

void f(widget& w) {
    g();
    use(w);  // A
}

void g() {
    g_p = ...; // oops, if this was the last shared_ptr to that widget, destroys the widget
}
```

The following should not pass code review:

```cpp linenums="1"
void my_code() {
    // BAD: passing pointer or reference obtained from a non-local smart pointer
    //      that could be inadvertently reset somewhere inside f or its callees
    f(*g_p);

    // BAD: same reason, just passing it as a "this" pointer
    g_p->func();
}
```

The fix is simple --- take a local copy of the pointer to "keep a ref count" for your call tree:

```cpp linenums="1"
void my_code() {
    // cheap: 1 increment covers this entire function and all the call trees below us
    auto pin = g_p;

    // GOOD: passing pointer or reference obtained from a local unaliased smart pointer
    f(*pin);

    // GOOD: same reason
    pin->func();
}
```



### ES.50: Don't cast away const

##### Reason
It makes a lie out of `const`. If the variable is actually declared `const`, modifying it results in undefined behavior.

```cpp linenums="1" title="Example, bad"
void f(const int& x) {
    const_cast<int&>(x) = 42;   // BAD
}

static int i = 0;
static const int j = 0;

f(i); // silent side effect
f(j); // undefined behavior
```

##### Example

Sometimes, you might be tempted to resort to `const_cast` to avoid code duplication, such as when two accessor functions
that differ only in const-ness have similar implementations. For example:

```cpp linenums="1"
class Bar;

class Foo {
public:
    // BAD, duplicates logic
    Bar& get_bar() {
        /* complex logic around getting a non-const reference to my_bar */
    }
    const Bar& get_bar() const {
        /* same complex logic around getting a const reference to my_bar */
    }
private:
    Bar my_bar;
};
```

Instead, prefer to share implementations. Normally, you can just have the non-const function call the const function.
However, when there is complex logic this can lead to the following pattern that still resorts to a const_cast:

```cpp linenums="1"
class Foo {
public:
    // not great, non-const calls const version but resorts to const_cast
    Bar& get_bar() {
        return const_cast<Bar&>(static_cast<const Foo&>(*this).get_bar());
    }
    const Bar& get_bar() const {
        /* the complex logic around getting a const reference to my_bar */
    }
private:
    Bar my_bar;
};
```

Although this pattern is safe when applied correctly, because the caller must have had a non-const object to begin
with, it's not ideal because the safety is hard to enforce automatically as a checker rule.

Instead, prefer to put the common code in a common helper function --- and make it a template so that it deduces `const`.
This doesn't use any `const_cast` at all:

```cpp linenums="1"
class Foo {
public:                         // good
          Bar& get_bar()       { return get_bar_impl(*this); }
    const Bar& get_bar() const { return get_bar_impl(*this); }
private:
    Bar my_bar;

    template<class T>           // good, deduces whether T is const or non-const
    static auto& get_bar_impl(T& t)
        { /* the complex logic around getting a possibly-const reference to my_bar */ }
};
```

Note: Don't do large non-dependent work inside a template, which leads to code bloat. For example, a further improvement
would be if all or part of `get_bar_impl` can be non-dependent and factored out into a common non-template function,
for a potentially big reduction in code size.

##### Exception

You might need to cast away `const` when calling const-incorrect functions. Prefer to wrap such functions in inline
const-correct wrappers to encapsulate the cast in one place.

##### Example

Sometimes, "cast away const" is to allow the updating of some transient information of an otherwise immutable object.
Examples are caching, memoization, and precomputation. Such examples are often handled as well or better using mutable
or an indirection than with a `const_cast`.

Consider keeping previously computed results around for a costly operation:

```cpp linenums="1"
int compute(int x); // compute a value for x; assume this to be costly

class Cache {   // some type implementing a cache for an int->int operation
public:
    pair<bool, int> find(int x) const;  // is there a value for x?
    void set(int x, int v);             // make y the value for x
    // ...
private:
    // ...
};

class X {
public:
    int get_val(int x)  {
        auto p = cache.find(x);
        if (p.first) 
           return p.second;
        int val = compute(x);
        cache.set(x, val); // insert value for x
        return val;
    }
    // ...
private:
    Cache cache;
};
```

Here, `get_val()` is logically constant, so we would like to make it a `const` member. To do this we still need to mutate cache,
so people sometimes resort to a `const_cast`:

```cpp linenums="1"
class X {   // Suspicious solution based on casting
public:
    int get_val(int x) const  {
        auto p = cache.find(x);
        if (p.first) 
           return p.second;
        int val = compute(x);
        const_cast<Cache&>(cache).set(x, val);   // ugly
        return val;
    }
    // ...
private:
    Cache cache;
};
```

Fortunately, there is a better solution: State that cache is mutable even for a const object:

```cpp linenums="1"
class X {   // better solution
public:
    int get_val(int x) const {
        auto p = cache.find(x);
        if (p.first) 
           return p.second;
        int val = compute(x);
        cache.set(x, val);
        return val;
    }
    // ...
private:
    mutable Cache cache;
};
```

An alternative solution would be to store a pointer to the cache:

```cpp linenums="1"
class X {   // OK, but slightly messier solution
public:
    int get_val(int x) const {
        auto p = cache->find(x);
        if (p.first) return p.second;
        int val = compute(x);
        cache->set(x, val);
        return val;
    }
    // ...
private:
    unique_ptr<Cache> cache;
};
```

That solution is the most flexible, but requires explicit construction and destruction of `*cache` (most likely in the constructor and destructor of X).

In any variant, we must guard against data races on the cache in multi-threaded code, possibly using a `std::mutex`.
