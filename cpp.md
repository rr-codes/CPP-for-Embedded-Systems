# C++ for Embedded Systems

## Introduction to C++


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

```cpp
auto oi1 = std::make_optional(1); // makes an optional holding an int with value 1
std::optional<int> oi2 = 1; // same as above
std::optional<int> oi3 = {}; // makes an empty optional
std::optional<int> oi4 = std::nullopt; // same as above
```
