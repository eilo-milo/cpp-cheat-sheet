# C++ Constants, Strings, and Compile-Time Evaluation (Chapter 5)

A comprehensive, practical C++ reference guide focused on **named constants, literal suffixes, compile-time evaluation (`constexpr`), `std::string`, and `std::string_view`**, structured for GitHub/Markdown documentation based on modern C++ best practices.

---

## 1. Constants Overview & Named Constants

A **constant** is a value that cannot be changed during a program’s execution.

### Types of Named Constants
1. **Constant Variables:** Declared using the `const` or `constexpr` keyword.
2. **Enumerated Constants:** Scoped and unscoped enums (covered in Chapter 13).
3. **Macro Constants:** `#define` preprocessor macros (**avoid** in modern C++).

```cpp
#include <iostream>

int main()
{
    const double gravity { 9.8 }; // Preferred: const before type
    // gravity = 9.9;             // Compile Error: Cannot assign to const variable
    
    int age { 20 };
    const int constAge { age };   // OK: Initializer can be a non-const value
}
```

> **Best Practice:** Place `const` before the type (`const double`), not after ("east const"). Avoid using `const` for by-value function parameters or by-value return types.

---

## 2. Literals & Literal Suffixes

Literals are values inserted directly into source code. Every literal has a deduced type.

```cpp
#include <iostream>
#include <string>

int main()
{
    auto i { 5 };      // Deduced as int
    auto f { 5.0f };   // 'f' suffix forces type float
    auto d { 5.0 };    // Deduced as double
    auto L { 5L };     // 'L' suffix forces type long
    auto u { 5u };     // 'u' suffix forces type unsigned int
}
```

### Literal Suffix Quick Reference

| Data Type | Suffix | Example | Notes |
| :--- | :--- | :--- | :--- |
| `float` | `f` / `F` | `4.1f` | Prevents `double`-to-`float` conversion warnings |
| `long` | `L` | `5L` | Use uppercase `L` to avoid confusion with `1` |
| `unsigned int` | `u` / `U` | `5u` | Converts integral literal to unsigned |
| `std::size_t` | `uz` / `UZ` | `5uz` | C++23 |
| `std::string` | `s` | `"Hello"s` | Requires `using namespace std::string_literals;` |
| `std::string_view` | `sv` | `"Hello"sv` | Requires `using namespace std::string_view_literals;` |

---

## 3. Numeral Systems & Binary Literals

C++ supports four numeral systems for integral literals:

```cpp
#include <iostream>
#include <bitset>

int main()
{
    int dec { 12 };         // Decimal (base 10)
    int oct { 014 };        // Octal (base 8, prefixed with 0 - avoid!)
    int hex { 0xC };        // Hexadecimal (base 16, prefixed with 0x)
    int bin { 0b1100 };     // Binary (base 2, C++14, prefixed with 0b)

    // C++14 Digit Separators for Readability
    long distance { 2'132'673'462 };
    int binFormatted { 0b1011'0010 };

    // Printing Binary using std::bitset
    std::cout << std::bitset<8>{ 0b1100'0101 } << '\n'; // Outputs: 11000101
}
```

---

## 4. Compile-Time Evaluation & Optimization

Modern C++ compilers optimize code using compile-time evaluation and the **As-If Rule** (compilers may alter code as long as observable behavior remains identical).

### Optimization Techniques
* **Constant Folding:** Replaces expressions of constant operands with their evaluated result (e.g., `3 + 4` -> `7`).
* **Constant Propagation:** Replaces variables known to hold constant values directly with their values.
* **Dead Code Elimination:** Removes code that has no effect on program behavior.

```cpp
#include <iostream>

int main()
{
    // Compiler optimizes this entire block down to std::cout << 10 << '\n';
    const int x { 7 };
    const int y { 3 };
    std::cout << x + y << '\n'; 
}
```

---

## 5. Constant Expressions & `constexpr`

A **constant expression** is an expression that *must* be evaluatable at compile time.

```cpp
#include <iostream>

constexpr int five() { return 5; }

int main()
{
    constexpr double gravity { 9.8 }; // Always evaluated at compile-time
    constexpr int sum { 4 + 5 };      // Always evaluated at compile-time
    
    int age { 20 };
    // constexpr int myAge { age };   // Compile Error: 'age' is not a compile-time constant
}
```

### `const` vs. `constexpr` Variables

| Keyword | Evaluation Time | Meaning | Usage |
| :--- | :--- | :--- | :--- |
| `const` | Compile-time or Runtime | Value cannot change after initialization | Use for runtime constants |
| `constexpr` | **Compile-time** | Object can be used in a constant expression | Use for compile-time constants |

> **Best Practice:** Declare any constant variable whose initializer is a constant expression as `constexpr`. Use `const` for runtime constants.

---

## 6. Dynamic Strings: `std::string`

`std::string` provides dynamic, mutable string management (allocates memory dynamically on the heap).

```cpp
#include <iostream>
#include <string>

int main()
{
    std::string name { "Alex" };
    name = "Alexander"; // Dynamically resizes

    // Standard Console Input Breaks on Whitespace
    // Use std::getline with std::ws to capture full lines
    std::cout << "Enter full name: ";
    std::getline(std::cin >> std::ws, name);

    // Get Length
    int length { static_cast<int>(name.length()) }; // std::string::length() returns unsigned
}
```

> **Best Practice:** Do **not** pass `std::string` by value to functions, as it performs an expensive dynamic allocation copy. Pass via `std::string_view` (for read-only) or `const std::string&`.

---

## 7. String Views: `std::string_view` (C++17)

`std::string_view` provides inexpensive, **read-only** access to an existing string (C-style string, `std::string`, or another `std::string_view`) **without making a copy**.

```cpp
#include <iostream>
#include <string>
#include <string_view>

// Parameter accepts C-strings, std::string, or std::string_view without copying
void printSV(std::string_view str)
{
    std::cout << str << '\n';
}

int main()
{
    std::string_view sv { "Hello, world!" }; // Views string literal without copying
    printSV(sv);

    // View Modifications (Does NOT modify the underlying string)
    sv.remove_prefix(1); // sv is now "ello, world!"
    sv.remove_suffix(1); // sv is now "ello, world"
}
```

### Dangling View Hazards (Undefined Behavior)

A `std::string_view` does **not** own the data it views. If the underlying data is destroyed or modified, the view becomes **dangling**.

```cpp
#include <iostream>
#include <string>
#include <string_view>

std::string_view getDanglingView()
{
    std::string s { "Hello" };
    return s; // UB! 's' is destroyed at function exit, view dangles!
}

int main()
{
    using namespace std::string_literals;
    
    std::string_view bad { "Hello"s }; // UB! "Hello"s creates a temporary std::string destroyed at end of statement
    // std::cout << bad << '\n';       // Accessing dangling view causes Undefined Behavior!
}
```

---

## 8. Selection Guide: `std::string` vs. `std::string_view`

### Variables
* Use `std::string` when you need an **owning** string object that you modify or receive from runtime user input.
* Use `constexpr std::string_view` for compile-time string symbolic constants.

### Function Parameters
* Prefer **`std::string_view` by value** for read-only string parameters.
* Use **`std::string&`** for in-out parameters that modify the caller's `std::string`.

### Return Types
* Return **`std::string` by value** when returning a local `std::string` object.
* Return **`std::string_view`** when returning C-style string literals (which exist for the lifetime of the program) or passing through a `std::string_view` parameter.

---

## Summary Best Practices

1. **Use `constexpr`:** Mark all compile-time constants as `constexpr`.
2. **Avoid Magic Numbers:** Replace numeric and string literals with named `constexpr` variables.
3. **Prefer `std::string_view` for Read-Only Function Parameters:** Eliminates dynamic memory allocation overhead.
4. **Guard Against Dangling Views:** Ensure the underlying string outlives any `std::string_view` referencing it.
5. **Use `std::ws` with `std::getline`:** Flush leading whitespace when reading full lines after using `std::cin >>`.
