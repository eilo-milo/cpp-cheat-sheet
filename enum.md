# C++ Production Reference: Program-Defined Types & Enumerations

An exhaustive, production-grade reference manual detailing C++ program-defined types, type definition mechanics, One-Definition Rule (ODR) exemptions, unscoped enumerations (`enum`), scoped enumerations (`enum class`), underlying base types, integral conversion safety, string parsing with `std::optional`, stream I/O operator overloading (`<<` / `>>`), and C++20 `using enum` mechanics.

---

## 1. Type Classification & Program-Defined Type Architecture

The C++ language standard distinguishes types by their origin and core semantics ([basic.types]).

```
                                [ C++ Type System ]
                                         │
                ┌────────────────────────┴────────────────────────┐
                │                                                 │
       Fundamental Types                                   Compound Types
  (Language Core Primitives)                           (Built on other types)
                │                                                 │
  ├── `int`, `double`, `char`               ┌─────────────────────┴─────────────────────┐
  └── `void`, `std::nullptr_t`              │                                           │
                                  Non-Program-Defined                      Program-Defined
                                  (Standard/Engine)                     (User/Library Defined)
                                            │                                           │
                                ├── `int*`, `int&`, `int[5]`                ├── Enumerations (`enum`, `enum class`)
                                └── `std::string`, `std::vector`            └── Class Types (`struct`, `class`, `union`)
```

### Type Taxonomy Definitions

| Category | Standard Definition | Examples | Memory Allocation on Type Definition? |
| :--- | :--- | :--- | :--- |
| **Fundamental** | Primitives built directly into the core language compiler. | `int`, `bool`, `double`, `std::nullptr_t` | N/A (Language built-in) |
| **Compound** | Types defined in terms of fundamental or other compound types. | `int*`, `int&`, `std::string`, `Fraction` | Only when instantiated |
| **User-Defined** | Standard language term for any `class` or `enum` type defined by the user, standard library, or compiler extension. | `std::string`, `std::vector`, `Fraction` | No |
| **Program-Defined** | C++20 formal term for `class` and `enum` types defined **strictly by the programmer or a 3rd-party library** (excludes `std::*` types). | `Fraction`, `Color`, `Status` | No |

### Type Definition vs. Object Instantiation
* **Type Definition:** Tells the compiler what the layout, members, and identity of a type look like. **Allocates zero memory**.
* **Instantiation:** Declares a concrete object variable of that type in memory.

```cpp
// TYPE DEFINITION: Defines structure only. Must end with a semicolon.
struct Fraction {
    int numerator   { 0 };
    int denominator { 1 };
}; // <-- Mandatory trailing semicolon

int main() {
    // INSTANTIATION: Allocates memory on the stack for object 'f'
    Fraction f { 3, 4 };
}
```

> **Warning:** Omitting the trailing semicolon at the end of a type definition causes severe cascade errors where the compiler interprets subsequent function declarations as part of the type definition.

### Header Propagation & ODR Partial Exemption
Every code file (translation unit) that uses a program-defined type requires the **full type definition** before use; a forward declaration is insufficient because the compiler needs to compute the exact object size in bytes.

* **One-Definition Rule (ODR) Partial Exemption:** While non-inline functions and global variables can only be defined once per program, **types can be defined across multiple translation units**, provided each type definition is **100% identical**.
* **Header Best Practice:** Define multi-file program-defined types in a dedicated header file (e.g., `Fraction.h`) guarded by `#ifndef` / `#define` header guards or `#pragma once`.

```cpp
// Fraction.h
#ifndef FRACTION_H
#define FRACTION_H

struct Fraction {
    int numerator   { 0 };
    int denominator { 1 };
};

#endif
```

---

## 2. Unscoped Enumerations (`enum`)

An **unscoped enumeration** is a program-defined compound type whose values are restricted to a named set of symbolic integral constants called **enumerators**.

```cpp
// Unscoped Enum Definition
enum Color {
    red,   // Enumerator 0
    green, // Enumerator 1
    blue,  // Enumerator 2 (Trailing comma is valid & recommended)
}; // Semicolon required

int main() {
    Color shirt { green }; // Initialized with enumerator 'green'
}
```

### Scope Pollution & Naming Collisions
Unscoped enumerators are placed into the **same scope region where the enum itself is defined**. If defined at global scope, all enumerator identifiers pollute the global namespace.

```cpp
enum Color   { red, green, blue };
// enum Feeling { happy, tired, blue }; // COMPILE ERROR: Identifier 'blue' collides with Color::blue
```

### Mitigation Strategies for Scope Pollution

#### 1. Namespace Wrapping Pattern (Preferred for Unscoped Enums)
```cpp
namespace Color {
    enum Color {
        red,
        green,
        blue,
    };
}

namespace Feeling {
    enum Feeling {
        happy,
        tired,
        blue, // Safe: Feeling::blue does not collide with Color::blue
    };
}

int main() {
    Color::Color shirt { Color::blue };
}
```

#### 2. Explicit Scope Resolution
Unscoped enumerators are accessible both directly (`red`) and via qualified scope resolution (`Color::red`).

```cpp
Color c1 { red };        // OK: Direct access
Color c2 { Color::red }; // OK: Qualified access
```

---

## 3. Underlying Base Types & Integral Conversions

### Default vs. Explicit Enumerator Values
Enumerators implicitly map to integers. By default, the first enumerator evaluates to `0`, and each subsequent enumerator increments by `1`. Custom integer assignments are permitted, including negative values and shared values.

```cpp
enum Animal {
    cat = -3,    // Explicit -3
    dog,         // Auto: -2
    pig,         // Auto: -1
    horse = 5,   // Explicit 5
    giraffe = 5, // Shares value 5 with horse (interchangeable; generally avoid)
    chicken,     // Auto: 6
};
```

### Value-Initialization (Zero-Initialization) Safety
Value-initializing an enumeration (`Color c{};`) sets its stored value to `0`, even if no enumerator was explicitly assigned `0`.

* **Best Practice:** Ensure the enumerator mapped to `0` represents the most logical default state, or define an explicit `unknown` or `invalid` state at `0`.

```cpp
enum GameResult {
    resultUnknown, // Map 0 to explicit uninitialized/unknown state
    player1Won,    // 1
    player2Won,    // 2
    draw,          // 3
};

int main() {
    GameResult res {}; // Safe: Explicitly defaults to resultUnknown (0)
}
```

### Specifying the Underlying Base Type
By default, the compiler selects an implementation-defined signed or unsigned integral type (typically `int`). You can explicitly specify the underlying type to control size or optimize network bandwidth.

```cpp
#include <cstdint>
#include <iostream>

// Force storage into a single 8-bit unsigned byte
enum Color : std::uint8_t {
    red,
    green,
    blue,
};

int main() {
    Color c { red };
    std::cout << sizeof(c); // Guarantees output of 1 byte
}
```

> **Warning:** Specifying `std::int8_t` or `std::uint8_t` as the underlying base causes I/O streams (`std::cout`) to treat converted enumerator values as printable `char` ASCII characters rather than numeric integers.

### Conversion Rules: Enum $\leftrightarrow$ Integer

| Direction | Conversion Type | Syntax / Behavior |
| :--- | :--- | :--- |
| **Unscoped Enum $\rightarrow$ Integer** | **Implicit** | Converts automatically to underlying integral type in expressions or function calls. |
| **Integer $\rightarrow$ Unscoped Enum** | **Explicit Only** | Requires `static_cast<EnumType>(integer)` |
| **Integer $\rightarrow$ Enum (C++17)** | **Brace Direct** | Allowed without cast **only** if enum has an explicitly specified base type: `Color c{2};` |

```cpp
enum Pet : int { cat, dog, pig };

int main() {
    // 1. Implicit Enum -> Integer
    int val = cat; // OK: val = 0

    // 2. Explicit Integer -> Enum (static_cast)
    Pet p1 { static_cast<Pet>(2) }; // OK: p1 = pig

    // 3. C++17 Direct List Initialization (Requires explicit base type)
    Pet p2 { 1 }; // OK in C++17: p2 = dog
    // Pet p3 = 1; // COMPILE ERROR: Copy-initialization not allowed
}
```

---

## 4. String Conversions & Case-Insensitive Parsing

Because enumerations do not retain string metadata at compile time, bidirectional conversion between string representations and enumerator values must be defined manually.

```cpp
#include <iostream>
#include <string_view>
#include <string>
#include <optional>
#include <algorithm>
#include <cctype>

enum class Pet {
    cat,
    dog,
    pig,
    whale,
};

// 1. Enum -> String View Conversion (constexpr safe)
constexpr std::string_view getPetName(Pet pet) {
    switch (pet) {
        case Pet::cat:   return "cat";
        case Pet::dog:   return "dog";
        case Pet::pig:   return "pig";
        case Pet::whale: return "whale";
        default:         return "???";
    }
}

// 2. String View -> Enum Conversion using std::optional
constexpr std::optional<Pet> getPetFromString(std::string_view sv) {
    if (sv == "cat")   return Pet::cat;
    if (sv == "dog")   return Pet::dog;
    if (sv == "pig")   return Pet::pig;
    if (sv == "whale") return Pet::whale;
    return std::nullopt; // Safe empty optional on invalid input
}

// 3. Helper: Convert string to ASCII lowercase for robust user input
std::string toLowerCase(std::string_view sv) {
    std::string lower{};
    lower.reserve(sv.size());
    std::transform(sv.begin(), sv.end(), std::back_inserter(lower),
        [](unsigned char c) { return static_cast<char>(std::tolower(c)); });
    return lower;
}
```

---

## 5. Overloading I/O Operators (`<<` and `>>`)

Overloading `operator<<` and `operator>>` integrates program-defined enumerations into standard C++ iostreams smoothly.

```cpp
#include <iostream>
#include <string_view>
#include <string>
#include <optional>

enum class Color {
    red,
    green,
    blue,
};

constexpr std::string_view getColorName(Color color) {
    switch (color) {
        case Color::red:   return "red";
        case Color::green: return "green";
        case Color::blue:  return "blue";
        default:           return "???";
    }
}

constexpr std::optional<Color> getColorFromString(std::string_view sv) {
    if (sv == "red")   return Color::red;
    if (sv == "green") return Color::green;
    if (sv == "blue")  return Color::blue;
    return std::nullopt;
}

// OVERLOAD OUTPUT OPERATOR (<<)
// Reference to std::ostream prevents copying; returns stream for chaining
std::ostream& operator<<(std::ostream& out, Color color) {
    return out << getColorName(color);
}

// OVERLOAD INPUT OPERATOR (>>)
// 'color' is passed as non-const lvalue reference (out-parameter)
std::istream& operator>>(std::istream& in, Color& color) {
    std::string input{};
    in >> input;

    std::optional<Color> match { getColorFromString(input) };
    if (match) {
        color = *match; // Set referent on valid match
    } else {
        in.setstate(std::ios_base::failbit); // Trigger stream failstate on error
    }

    return in;
}

int main() {
    Color shirt { Color::blue };
    std::cout << "Shirt color: " << shirt << '\n'; // Prints: "Shirt color: blue"

    std::cout << "Enter color (red, green, blue): ";
    if (std::cin >> shirt) {
        std::cout << "Successfully set to: " << shirt << '\n';
    } else {
        std::cout << "Invalid color entered.\n";
    }
}
```

---

## 6. Scoped Enumerations (`enum class`)

Scoped enumerations (`enum class` or `enum struct`) solve the design drawbacks of unscoped enumerations by enforcing strict scope isolation and strong type safety.

```cpp
enum class Color {
    red,
    blue,
};

enum class Fruit {
    banana,
    apple,
};

int main() {
    Color color { Color::red };   // Explicit qualification required
    Fruit fruit { Fruit::banana };

    // COMPILER ERRORS:
    // std::cout << red;         // Error: 'red' not in this scope
    // if (color == fruit)       // Error: Cannot compare distinct types Color and Fruit
    // int val = color;          // Error: No implicit conversion to int
}
```

### Converting Scoped Enumerators to Integers

```cpp
#include <iostream>
#include <utility> // std::to_underlying (C++23)

enum class Status : int {
    ok = 200,
    notFound = 404,
};

// Advanced Technique: Overload unary + operator for quick integer evaluation
template <typename T>
    requires std::is_enum_v<T>
constexpr auto operator+(T e) noexcept {
    return static_cast<std::underlying_type_t<T>>(e);
}

int main() {
    Status s { Status::notFound };

    // Method 1: Explicit static_cast (C++11)
    int code1 = static_cast<int>(s);

    // Method 2: std::to_underlying (C++23 Standard Helper)
    auto code2 = std::to_underlying(s);

    // Method 3: Unary + Overload Hack
    auto code3 = +s; // Evaluates instantly to int(404)
}
```

---

## 7. C++20 `using enum` Declarations

Introduced in C++20, the `using enum` directive imports all enumerators of an `enum class` into the current local scope region. This eliminates repetitive scoped prefixes inside localized contexts like `switch` blocks while preserving overall type safety.

```cpp
#include <iostream>
#include <string_view>

enum class Mode {
    read,
    write,
    append,
    exec,
};

constexpr std::string_view getModeString(Mode mode) {
    using enum Mode; // Import all Mode enumerators into local function scope

    switch (mode) {
        case read:   return "READ";   // Equivalent to case Mode::read
        case write:  return "WRITE";  // Equivalent to case Mode::write
        case append: return "APPEND";
        case exec:   return "EXEC";
    }
}

int main() {
    Mode m { Mode::write };
    std::cout << getModeString(m) << '\n';
}
```

---

## 8. Architectural Comparison: Unscoped vs. Scoped Enums

| Feature Dimension | Unscoped Enumeration (`enum`) | Scoped Enumeration (`enum class`) |
| :--- | :--- | :--- |
| **Declaration Syntax** | `enum Name { ... };` | `enum class Name { ... };` |
| **Enumerator Scope** | Parent scope (Pollutes enclosing namespace) | Local Enum Scope (`Name::Enumerator`) |
| **Implicit Integer Conversion** | **Yes** (Decays to underlying int implicitly) | **No** (Strict type isolation; requires explicit cast) |
| **Default Underlying Base** | Implementation-defined (`int`, `unsigned int`) | Fixed to `int` |
| **Forward Declaration** | Allowed **only** if base type is specified | **Always allowed** (`enum class Name;`) |
| **Comparison Behavior** | Compares freely across different unscoped enums | Disallowed across different enum types |
| **C++17 Direct Integral Init** | Requires explicit base type | Supported natively (`Name obj{0};`) |
| **C++20 `using enum` Support** | Supported | Supported |
| **Production Guideline** | **Avoid** (Except for low-level C API boundaries) | **Preferred Default Choice** |
