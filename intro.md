# Part 1: Introduction to C++
*By Richard Robinson*

### C++ Types Overview
The primitive types of the C++ language include
- `int`: an integer (typically 32 bits)
- `double`: a decimal number
- `float`: a decimal number, but less precise than `double`
- `char`: an ASCII character (or an 8-bit integer)
- `bool`: a boolean value (true or false)
- `nullptr`: like `null` in other languages, but only for pointers

The type qualifiers `short`, and `long` may be added to `int` or `double` to reduce or extend the type's range of values, respectively. `int` may also have a `long long` qualifier. Note that `short`, `long`, or `long long` are shorthand for `short int`, `long int`, and `long long int`.

The C++ standard library (`std`) includes some other important types and classes:

- `std::string`: a class which provides easy access for string use (instead of the C-equivalent `const char *`)
- `std::optional<T>`: a container which may or may not be empty
- `std::function<R(T)`: the type of a function whose return value is of type `R` and with a single parameter of type `T`
- `std::size_t`: an implementation-defined numeric type for sizes (designed to be used when referring to the size of containers or loop indices; often aliased to `unsigned long long int`)

### More on `std::optional<T>`

`std::optional<T>` is a wrapper around a type `T` which may or may not contain an instance of `T`. With the introduction of `std::optional<T>`, `nullptr` is rendered largely unnecessary, and therefore so are null errors and exceptions. Most modern languages such as Swift have support for similar _nullable reference types_.

If a variable is stored in an optional, the optional instance can largely be treated as if it were the variable itself. An optional may be instantiated in many different ways:

```cpp
auto oi1 = std::make_optional(1); // makes an optional holding an int with value 1
std::optional<int> oi2 = 1; // same as above
std::optional<int> oi3 = {}; // makes an empty optional
std::optional<int> oi4 = std::nullopt; // same as above
```

> *A note on operator overloading*: In C++ and some other languages, operator (such as `++, ==, +, >, *, ->, []`, etc) can be implemented like regular functions but called like operators (alternatively, they can be called via `foo.operator+(bar)` for example).
>
> Except for primitives, operator's can't be called on arbitrary objects which haven't overloaded them. For example, a class representing a Cartesian coordinate may implement a `+` operator to add two points.
>
> Operators may either be prefix (such as `*`), infix (such as `+`), or postfix (such as `[]`).
>
> When the `*` operator is overloaded, `->` is also frequently overloaded (as in `foo->bar()`) to act as a shortcut for `(*foo).bar()`.

The `std::optional<T>` class provides the following members

| Member Signature | Description |
|------------------|-------------|
| `T value()` <br/> `T operator*()` <br/> `T operator->()` | Gets the value stored in the optional. For an optional o, then  `o.value(), *o`, and `o->(function)` all access the value.  |
| `bool has_value()` <br/> `bool operator bool()`  | Does this optional have a variable? For an optional o,  `o.has_value() and (bool) o` both test this condition   |
| `T value_or(T)` | Returns the contained value if available, another value otherwise   |
| `bool operator==(T)` <br/> `bool operator==(std::optional<T>)` <br/> (and `>, >=, <, <=, !=`) | In the first case, tests the value of this optional against the parameter. In the second case, tests the values of this optional against the provided optional. |

---

```cpp
int main() {
    std::optional<std::string> os = "hi";

    bool hasValue1 = os.has_value(); // checks if value is present
    bool hasValue2 = (bool) os; // same as above

    char f1 = os.value().at(0); // gets the first char
    char f2 = os->at(0); // same as above
    char f3 = (*os).at(0); // same as above
}
```
*An overview of some std::optional<T> members and their usage*

---


```cpp
#include <iostream>
#include <optional>

/// Tries to parse `arg` into an integer value
/// \return an optional containing the parsed value if successful
std::optional<int> ParseInt(const std::string& arg)
{
    try {
        return std::stoi(arg); // `stoi` converts a string to an int, or throws an exception if it can't
    } catch (...) {
        std::cout << "cannot convert \'" << arg << "\' to int!\n";
    }

    return std::nullopt;
}

int main() {
    auto maybeValue = ParseInt("s");

    if (maybeValue.has_value()) { // checks if the optional is empty or not
        std::cout << maybeValue.value(); // prints the value in the optional
    } else {
        std::cout << -1; // otherwise print -1
    }

    std::cout << maybeValue.value_or(-1); // shorthand for above
}
```

*Optional values are frequently used when parsing data*

### More on `std::function<R(T)>`

Like some other modern languages, C++ models functions as *first class citizens*, which gives support to *lambda functions* (more on this later). For a function *f* which returns a value of type *R* with parameters of types *A, B, ...*, the type of *f* itself can be expressed as either
- `std::function<R(A, B, ...)> f`, or
- `R (*f)(A, B, ...)` (this variation is usually used when a function is a parameter to another function)

When assigning a function to a variable, the function can be expressed as either
- `f = &foo` (where `foo` is a previously defined function), or
- `f = [](A a, B b, ...) -> R { /* stuff */ }` (used for inner, short functions)

```cpp
// usually we use this syntax for a function
int increment(int i) { return i + 1; }

// but this is also a function
auto increment2 = [](int i) -> int { return i + 1; };

/// For each of `i` iterations, execute `fn`
/// \param fn a function whose parameter is the iteration index
void for_i(std::size_t start, std::size_t end, void (*fn)(std::size_t i)) {
    for (auto i = start; i < end; ++i) fn(i);
}

int main() {
    std::function<int(int)> g1 = &increment;
    int (*g2)(int) = increment2;

    auto res1 = g1(42); // calls increment(42)
    auto res2 = g2(24); // calls increment2(24)

    for_i(0, 5, [](auto i) { std::cout << i << std::endl; });
}
```
*An example using std::function and its syntax*
