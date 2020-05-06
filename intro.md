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


### Enum Classes

```cpp
enum class Day {
  Sun, Mon, Tue, Wed, Thu, Fri, Sat
};

Day d = Day::Fri; // implicitly equivalent to 5
```

Enum classes are generally used to symbolize a fixed set of constants / identifiers; they are a higher-level alternative to using arbitrary int values typical in a C program.

In C++, each value is implicitly assigned an integer value beginning at 0; these values can be explicitly assigned to any value.

It is a fairly common practice to use switch-case statements, in which each enum value is a different case.

> An advanced usage of enums is when APIs overload the binary *or* (`|`) operator for enum classes. This is typically done to design functions that can accept a variable number of different options or settings.

```cpp
enum class Unit { Celsius, Farenheit, Kelvin };

double to_kelvin(double temp, Unit unit) {
  switch (unit) {
    case Unit::Celsius: return temp + 273.15;
    case Unit::Farenheit: return (temp - 32.0) * (5.0 / 9.0) + 273.15;
    default: return temp;
  }
}

auto tk = to_kelvin(0.0, Unit::Celsius);
```


### Type Qualifiers
Several C++ keywords serve as *type qualifiers*, which modify attributes of a given type when applied to them. For a type `T`.
- `const T` means the value of the variable is constant and cannot be changed. Use whenever possible for better software design.
- `*T` is a raw pointer to a type `T` (a pointer is the memory address of a particular variable)
- `unsigned T`, when applied to numeric types, restricts the range of `T` to positive values, and doubles the range. In embedded applications, it is common for `uint8_t` (an `int` the size of one byte) to be aliased for `unsigned char`, which has a range of [0, 255].
- `constexpr T` means the value of the variable is computed at compile-time, maximizing performance and software design. Use whenever possible.

### More on `constexpr`

In C, the `#define A B` command makes `A` an alias for `B`, such that whenever the program is built, all instances of `A` are automatically replaced by the computed value of `B`.

In C++, `constexpr auto A = B;` is virtually identical to `#define`, but is better due to type safety, syntax safety, and limiting arbitrary expressions. Since all `constexpr` expressions are evaluated at *compile time*, the final assembly code has all `constexpr` expressions pre-computed, which greatly increases performance at the cost of compilation time.

Because `constexpr` expressions are evaluated at compile time, these expressions can only use a combination of primitives and other `constexpr` expressions. These expressions are most often used for simple calculations, constants, and *magic numbers*.

```cpp
// computes the factorial of `n`, so long as the result is a number less than 64 bits long
constexpr std::uint64_t factorialOf(const int n) {
    return n > 1 ? n * factorialOf(n - 1) : 1;
}

// f42 is a giant number, which would be slow to usually compute.
// However, with constexpr, `f20` is pre-computed and replaced by 42!
// Because it is pre-computed, there is a 100% performance increase (time to compute ~ 0.0s)
constexpr auto f20 = factorialOf(20);
```

### Implicit Casting

Some primitive types may be automatically casted (converted) to other types:

- `unsigned T <=> T`: unsigned types and their signed counterparts can be implicitly casted to each other. If casting `unsigned T => T`, the cast is safe so long as the unsigned type's value doesn't exceed the signed type's range of values. If casting `T => unsigned T`, the cast is safe so long as the signed type's value is non-negative; else, the value wraps around to the maximum value of `T`.

- `[unsigned] char <=> [unsigned] T`, where `T` is a numeric type. A char is an alias for an 8-bit number (one byte). Since a `char` is a single byte, `unsigned char` is frequently used in embedded codebases to represent a byte of data (a number between 0 - 255). If casting `T => char`, the cast is undefined if the value of `T` exceeds the range of `[unsigned] char`.

- `[unsigned] char <=> bool`: If casting `int => bool`, `0` becomes `false` and any other number becomes `true`. If `bool => int`, `true => 1` and `false => 0`.

- `T1 <=> T2`, where both are numeric types. If `T1` has a range less than `T2`, the cast `T1 => T2` is known as a *widening conversion* and can be performed safely. Otherwise, a *narrowing conversion* is performed, and is safe only if the value of `T2` is within the range of `T1`.

```cpp
char a_char = 'a';
int a_int = a_char; // 97
char P_char = 80; // 'P'

unsigned int i = -1; // 2^32 - 1 because of underflow
int t = true; // 1

char x = '9'; // the character '9', or the integer 71
int n = x - '0'; // converts the character '9' to the number 9
```

### Explicit Casting

Both primitive and non-primitive types can be (possibly) casted to other types with explicit casting:

- `static_cast<T>(var)`: this is the 'traditional' explcit cast; it should be considered as the 'default' cast to use when another cast is not better suited.

- `reinterpret_cast<T*>(var)`: this cast is used for low-level pointers and bit manipulation, as it converts any object to a sequence of raw bytes. Do not use unless absolutely necessary.

- `const_cast<T>(var)`: used to add or remove `const` or `volatile` to a variable. Mostly this cast is unnecessary, but has advanced usages such as overloading member functions.

- `dynamic_cast<T>(var)`: used to handle polymorphic casts (via virtual classes).

For example, f class `B` inherits from class `A`, then
```cpp
B b = dynamic_cast<B>(a); // where a is of type `A`
```
casts `a` to type `B`, assuming such cast is viable.

There is also C-style casting (`(T) foo`), which performs the first of the following casts which succeeds:
1. `const_cast`
2. `static_cast`
3. `static_cast`, then `const_cast`
4. `reinterpret_cast`
5. `reinterpret_cast`, then `const_cast`

C-style casts should be avoided in C++ as they are inherently unpredictable, and may decay into the dangerous `reinterpret_cast`.

### Pointers & References

> “Phenomenal cosmic powers… Itty bitty living space.” — Genie (Aladdin)

*Pointers* are variables which store the memory address of other variables / objects.

Assuming `foo` is a `T` located at address *x*:
```cpp
&foo; // this is the memory address of foo (`x`)
T* ptr; // this is a variable of type `T*` (a pointer to T)
ptr = &foo; // assigns the ptr to the memory address of `foo` (ptr == x)

*reinterpret_cast<T*>(x); // an equivalent (very dangerous) way of accessing `foo`
```

The syntax of pointers can be somewhat confusing:
- For an object `foo`, `&foo` gets it's memory address (dereferences it)
- For a pointer `p`, to change what `p` is pointing to, use `p = &foo`
- To change the value of whatever `p` is pointing to, use `*p = bar`
- To access the original value / object to modify it, use `*p`. When accessing member functions of the original object, the syntax is `(*p).foo()` or `p->foo()` (latter is preferred).

```cpp
int main() {
  int i = 1, j = 3;
  int* ptr = &i // ptr points to i
  *ptr = 2; // i is now 2
  ptr = &j; // ptr points to j (doesn't change i)

  // `*ptr = ` doesn't change ptr (affects target)
  // `ptr = ` changes ptr but nothing else
}
```

Historically, `*foo` meant to get the value stored in whichever pointer (dereference). In modern C++, due to operator overloading, it is mostly syntactic sugar meaning to 'get the underlying value'. For example, `std::optional<T>`, smart pointers, iterators, etc. all use this convention.

The arrow operator (`->`) is also frequently overloaded as syntactic sugar to acess the underlying variable's members (usually implemented as shorthand for `(*foo).func()`).

*References* are variables which are just aliases (another name) of other variables / objects. References should be used whenever possible as function arguments (except primitives); doing so
- Allows the argument to be changed
- Greatly saves memory and improves performance

If the function doesn't modify or change the argument, then a const reference (`const T&`) should be used. Some functions use reference arguments as a way to 'return' multiple values.

```cpp
// bad because it copies the Foo object
constexpr int valueOf1(Foo foo) { return foo.value; }

// bad because pointers are dangerous and can be modified and passed nullptr
// pointer params can be reassigned or modified; const pointers only the latter
// foo->bar() is a shortcut for (*foo).bar();
constexpr int valueOf2(const Foo* foo) {  return foo->value; }

// best of both worlds since no copying, unmodifiable, and is safe
// reference params can be reassigned or modified; const refs can do neither
constexpr int valueOf3(const Foo& foo) { return foo.value; }

```

If you change a pointer or a reference, the same change is applied to the original object. By default, pointers shouldn't be used as they are unnecessary and prone to bugs, particularly null pointer faults.

|   | Value | Pointer | Reference |
|---|---|---|---|
| Syntax | `T foo` | `T* ptr = &foo` </p> `T* ptr = new T(args)` | `T& ref = foo` |
| Assignment to `nullptr`? | No | Yes (`ptr == nullptr`) | No |
| Assignment | `foo = baz` </p> `foo = *ptr` | `*ptr = baz` </p> `ptr = &baz` | `ref = baz` |
| Can be changed in functions? | No | Yes | Yes (unless you use `const T&`) |
