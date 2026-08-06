# C++ Cheat Sheet: Type Conversions, Promotions, & Auto Type Deduction

A reference guide covering C++ implicit conversions, standard conversion categories, numeric promotions, numeric conversions, narrowing conversions, usual arithmetic conversions, explicit casting (`static_cast`), type aliases, and `auto` type deduction.

---

## 1. Type Conversion Fundamentals

### Core Definitions
* **Type Conversion:** The process of converting data from its original type to a target type.
* **Implicit Conversion (Coercion):** Performed automatically by the compiler when a specific data type is required in an expression, assignment, or function call but a different type is supplied.
* **Explicit Conversion (Casting):** Requested explicitly by the developer using a cast operator (e.g., `static_cast`).
* **Conversion Mechanics:** Conversions **do not modify** the source object or its bits. Instead, the input value is evaluated to create a **temporary object** of the target type holding the converted result.

---

## 2. Standard Conversion Categories

The C++ standard defines standard conversions across 5 primary categories:

| Category | Description | Examples |
| :--- | :--- | :--- |
| **Numeric Promotions** | Safe, value-preserving widening of narrow types (`char`, `short`, `float`). | `char` $\rightarrow$ `int`, `float` $\rightarrow$ `double` |
| **Numeric Conversions** | Conversions between fundamental types that are not promotions. Can be lossy or unsafe. | `double` $\rightarrow$ `int`, `int` $\rightarrow$ `unsigned int` |
| **Qualification Conversions** | Conversions adding or removing `const` or `volatile` qualifiers. | `int*` $\rightarrow$ `const int*` |
| **Value Transformations** | Changes to the value category of an expression. | Lvalue-to-rvalue conversion, array decay |
| **Pointer Conversions** | Pointer transformations and null pointer mappings. | `std::nullptr_t` $\rightarrow$ `T*`, `Derived*` $\rightarrow$ `Base*` |

---

## 3. Numeric Promotions

Numeric promotions are **always value-preserving (safe)**. Every possible source value fits precisely within the destination type. Compilers never generate warnings for promotions.

### Promotion Categories
1. **Floating-Point Promotions:**
   * `float` $\rightarrow$ `double`

2. **Integral Promotions:**
   * `signed char`, `signed short` $\rightarrow$ `int`
   * `unsigned char`, `unsigned short` $\rightarrow$ `int` (if `int` can hold the range) or `unsigned int`
   * `bool` $\rightarrow$ `int` (`false` $\rightarrow$ `0`, `true` $\rightarrow$ `1`)

```cpp
#include <iostream>

void printInt(int x) {
    std::cout << x << '\n';
}

int main() {
    short s{ 3 };
    printInt(s);    // Promoted: short -> int
    printInt('a');  // Promoted: char -> int (ASCII 97)
    printInt(true); // Promoted: bool -> int (1)
}
```

---

## 4. Numeric Conversions & Safety Classes

Numeric conversions handle fundamental type conversions outside promotion rules. They fall into three safety categories:

| Safety Class | Description | Risk | Example |
| :--- | :--- | :--- | :--- |
| **Value-Preserving** | Target type can accurately hold all values of the source type. | None (Safe) | `int` $\rightarrow$ `long`, `short` $\rightarrow$ `double` |
| **Reinterpretive** | Bit representations reinterpreted; values preserve data but may alter range or sign. | Logical bugs / Modulo wrapping | `int (-5)` $\rightarrow$ `unsigned int (4294967291)` |
| **Lossy** | Precision or fractional components are permanently lost during conversion. | Data loss | `double (3.5)` $\rightarrow$ `int (3)`, `double` $\rightarrow$ `float` |

```cpp
// Reinterpretive conversion (reversible without data loss, but changes value semantics)
int n = static_cast<int>(static_cast<unsigned int>(-5)); // Evaluates back to -5

// Lossy conversion (irreversible data loss)
double d { 3.5 };
int i = static_cast<int>(d); // i = 3 (fractional .5 lost)
```

---

## 5. Narrowing Conversions & `constexpr` Exceptions

A **narrowing conversion** is a numeric conversion where the target type cannot guarantee holding all possible values of the source type.

### Disallowed in List Initialization
Brace initialization (`{}`) **forbids implicit narrowing conversions** at compile time to prevent subtle bugs:

```cpp
int i { 3.5 }; // COMPILE ERROR: narrowing conversion from double to int

// FIX: Explicitly signal intent using static_cast
int i { static_cast<int>(3.5) }; // OK
```

### The `constexpr` Exclusion Clause
Implicit conversions from `constexpr` integer initializers are **not narrowing** if the compiler verifies at compile time that the value fits exactly within the destination type without data loss:

```cpp
constexpr int n1{ 5 };
unsigned int u1 { n1 }; // OK: 5 fits in unsigned int (Not narrowing)

constexpr int n2{ -5 };
// unsigned int u2 { n2 }; // COMPILE ERROR: -5 cannot be represented in unsigned int
```

> **Note:** Floating-point to integer conversions do **not** benefit from the `constexpr` exclusion clause and are always treated as narrowing.

---

## 6. Usual Arithmetic Conversions

When binary operators (e.g., `+`, `-`, `*`, `/`, `<`, `==`) evaluate operands of different types, C++ implicitly converts them to a common type based on a priority hierarchy:

```text
[ Usual Arithmetic Conversion Priority Hierarchy ]

1. long double  (Highest Rank)
2. double
3. float
4. long long
5. long
6. int          (Lowest Rank)
```

### Step-by-Step Conversion Pipeline
1. **Floating-Point Priority:** If any operand is floating-point, the non-floating-point operand converts to the highest-ranked floating-point type.
2. **Integral Promotion:** Otherwise, all integral operands narrower than `int` undergo integral promotion.
3. **Rank Matching:** The lower-ranked operand converts to the higher-ranked operand's type.

### Signed / Unsigned Pitfalls
Mixing signed and unsigned types in arithmetic or comparisons converts the signed value to unsigned, leading to unexpected modulo wrapping:

```cpp
#include <iostream>

int main() {
    // 5u is unsigned int, -10 is signed int -> -10 converted to large unsigned int
    std::cout << (5u - 10) << '\n'; // Output: 4294967291 (32-bit architecture)

    // -3 converted to large unsigned int -> comparison evaluates to false
    std::cout << std::boolalpha << (-3 < 5u) << '\n'; // Output: false
}
```

---

## 7. Explicit Casting & `static_cast`

Explicit casting informs the compiler that a type conversion is intentional, suppressing implicit narrowing warnings.

### Casting Operators Summary

| Cast Operator | Purpose | Safety Level |
| :--- | :--- | :--- |
| `static_cast` | Safe, compile-time checked conversion between compatible types. | Safe |
| `const_cast` | Adds or removes `const` or `volatile` qualifiers. | Use with caution |
| `reinterpret_cast` | Reinterprets underlying bit patterns without conversion. | Dangerous |
| `dynamic_cast` | Runtime-checked polymorphic inheritance conversions. | Safe |
| C-style Cast `(type)val` | Tries `const_cast`, `static_cast`, `reinterpret_cast` sequentially. | **Avoid** (Unsafe/Obscure) |

```cpp
#include <iostream>

int main() {
    int x { 10 };
    int y { 4 };

    // Floating-point division using static_cast
    double result = static_cast<double>(x) / y; // 10.0 / 4 -> 2.5
    
    // Explicit narrowing conversion
    double d { 48.9 };
    char ch { static_cast<char>(d) }; // Suppresses compiler warning
}
```

---

## 8. Type Aliases (`using`)

A type alias provides a readable identifier for an existing data type. It does **not** create a distinct new type (it is not type-safe against aliased primitives).

```cpp
#include <vector>
#include <utility>
#include <string>

// Preferred C++11 syntax
using Distance = double;
using PairList = std::vector<std::pair<std::string, int>>;

// Legacy C-style syntax (DISCOURAGED in modern C++)
typedef double Distance;

int main() {
    Distance miles { 4.5 }; // Evaluated as double by the compiler
}
```

> **Best Practice:**
> * Prefer `using` alias syntax over legacy `typedef`.
> * Capitalize custom alias names (e.g., `Distance`) to distinguish types from variables and functions.

---

## 9. Auto Type Deduction (`auto`)

The `auto` keyword instructs the compiler to deduce an object's variable type from its initializer expression at compile time.

```cpp
#include <string>
#include <string_view>

int main() {
    auto d { 5.0 };     // Deduced as double
    auto i { 1 + 2 };   // Deduced as int
    
    // Literal suffixes dictate deduced types
    auto f { 1.2f };    // float
    auto u { 5u };      // unsigned int

    // String literals require s/sv suffixes to avoid decaying to const char*
    using namespace std::literals;
    auto s1 { "hello"s };  // std::string
    auto s2 { "hello"sv }; // std::string_view
}
```

### Qualifier Dropping Rules
1. **Top-Level `const` Dropping:** `auto` strips top-level `const` and `constexpr` qualifiers from deduced types. They must be explicitly reapplied.
2. **Reference Dropping:** `auto` strips references. Use `auto&` to retain reference semantics.

```cpp
int main() {
    const int x { 5 };
    auto a { x };       // Type: int (const dropped)
    const auto b { x }; // Type: const int (const reapplied)

    constexpr double g { 9.8 };
    auto c { g };       // Type: double (const/constexpr dropped)
    constexpr auto d { g }; // Type: const double (constexpr reapplied)
}
```

### Function Return Type Deduction (`auto`) & Trailing Return Types

```cpp
// C++14: Return type auto deduction
auto add(int x, int y) {
    return x + y; // Deduces int return type
}

// C++11: Trailing return type syntax
auto multiply(double x, double y) -> double {
    return x * y;
}
```

> **Best Practice:** Prefer explicit return types over `auto` return type deduction for public interfaces and multi-file code to keep interfaces clear, documented, and stable.
