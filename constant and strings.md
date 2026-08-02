# 🚀 C++ Constants, Optimizations & Strings Cheat Sheet

> A fast-reference summary covering C++ compile-time evaluation, string types (`std::string` vs `std::string_view`), memory allocation mechanics, and execution best practices.

---

## 📄 1. Constants & Type Qualifiers

### 💡 Core Distinction
- **Named Constants:** Identifiers bound to an immutable value (`const`, `constexpr`, `enum`, `#define`).
- **Literal Constants:** Unnamed explicit values directly written in code (`5`, `"hello"`).

### ✍️ Declaration Styles

```cpp
// PREFERRED: Modifiers come before type (standard English convention)
const double earthGravity { 9.8 };

// ACCEPTABLE: East Const style
double const standardPressure { 101.325 };

// BAD PRACTICE: Preprocessor macro (ignores scope rules, hard to debug)
#define EARTH_GRAVITY 9.8 
```

### ⚔️ `const` vs `constexpr`

| Qualifier | Meaning | Evaluation Time | Primary Use Case |
| :--- | :--- | :--- | :--- |
| `const` | Value cannot change after initialization | Runtime or Compile-time | Runtime constants (e.g., user input) |
| `constexpr` | Expresses a true compile-time constant | Must be Compile-time | Array dimensions, template bounds, constants |

```cpp
#include <iostream>

int getRuntimeValue() { return 42; }

int main() {
    const int runtimeConst { getRuntimeValue() }; // OK: resolved at runtime
    constexpr int compileConst { 100 };           // OK: resolved at compile-time
    
    // constexpr objects are implicitly const!
    // compileConst = 200; // Compile Error
}
```

> **📌 Best Practice Rules:**
> - Prefer `constexpr` for any constant whose initializer can be calculated at compile-time.
> - Fallback to `const` for runtime-only initializers.
> - **Avoid** declaring function value parameters or return types as `const`.

---

## 🔢 2. Literals, Suffixes & Numeral Systems

### 🏷️ Literal Suffixes

```cpp
// Integral Suffixes
auto a { 5 };     // int (default)
auto b { 5L };    // long (Always prefer uppercase 'L' over 'l')
auto c { 5u };    // unsigned int
auto d { 5ULL };  // unsigned long long
auto e { 5z };    // signed std::size_t (C++23)

// Floating-Point Suffixes
auto f1 { 4.1 };  // double (default)
auto f2 { 4.1f }; // float (Avoids implicit double-to-float narrowing warnings)

// Scientific Notation
double avogadro { 6.02e23 }; // 6.02 * 10^23
```

### ⚙️ Numeral Systems & Digits Separation

```cpp
#include <iostream>
#include <bitset>

int main() {
    int dec { 12 };          // Decimal (Base 10)
    int oct { 014 };         // Octal (Base 8, Prefix 0) -> DISCOURAGED
    int hex { 0x0C };        // Hexadecimal (Base 16, Prefix 0x)
    int bin { 0b0000'1100 }; // Binary (Base 2, Prefix 0b) [C++14]
                             // Note: Single quote `'` is a visual digit separator

    // Formatting Stream Output
    std::cout << std::hex << dec << '\n'; // Prints 'c'
    std::cout << std::dec << dec << '\n'; // Resets to decimal

    // Binary Formatting with std::bitset
    std::cout << std::bitset<8>{ 0b0000'1100 } << '\n'; // Prints "00001100"
}
```

---

## ⚡ 3. Compiler Optimizations & Compile-Time Programming

### 🎯 Key Optimization Mechanics
- **As-If Rule:** The compiler can freely alter machine instruction order or drop code completely as long as the **observable behavior** remains identical.
- **Constant Folding:** Replacing expressions containing fixed values with calculated totals at compile-time (`3 + 4` $\rightarrow$ `7`).
- **Constant Propagation:** Substituting variable identifiers directly with their known constant value.
- **Dead Code Elimination:** Stripping out unused runtime variables or unreachable code paths.

### 🛠️ `constexpr` Functions

A `constexpr` function can execute at compile time **if** its arguments are constant expressions, but remains reusable at runtime for non-constant arguments.

```cpp
#include <iostream>

constexpr int factorial(int n) {
    return (n <= 1) ? 1 : (n * factorial(n - 1));
}

int main() {
    // Compile-time evaluation guaranteed (initializer requires compile-time result)
    constexpr int f5 { factorial(5) }; 

    // Runtime evaluation (argument 'x' is non-const)
    int x = 5;
    int fRuntime { factorial(x) }; 
}
```

---

## 🧵 4. Strings Mechanics: `std::string` vs `std::string_view`

### 📊 Structural Comparison

| Feature | `std::string` | `std::string_view` (C++17) |
| :--- | :--- | :--- |
| **Ownership** | Exclusive Owner (Manages its own buffer) | Observer / Viewer (Non-owning reference) |
| **Allocation Cost**| Heavy (Allocates runtime heap memory) | Fast (Zero-allocation: Pointer + Length) |
| **Null-Termination**| Guaranteed (`\0` terminated) | **Not guaranteed** (when viewing substrings) |
| **`constexpr` Support**| Limited (C++20/23) | Fully Supported |
| **Mutability** | Modifiable | Read-Only View |

---

### 📝 `std::string` Usage Guidelines

```cpp
#include <iostream>
#include <string>

int main() {
    using namespace std::string_literals;

    // Literal suffix 's' creates std::string instead of C-string
    auto strObj { "Hello World"s }; 

    // Safe Input Reading with Whitespace Handling
    std::string fullName {};
    std::cout << "Enter full name: ";
    // std::ws clears leftover newline/whitespace buffers before extraction
    std::getline(std::cin >> std::ws, fullName);

    // Length Handling (preventing signed/unsigned conversion warnings)
    int length { static_cast<int>(fullName.length()) };
}
```

---

### 🔍 `std::string_view` Usage & Undefined Behavior Pitfalls

```cpp
#include <iostream>
#include <string>
#include <string_view>

// PREFERRED: Pass std::string_view by value (Copying pointer + size is cheap!)
void printSV(std::string_view sv) {
    std::cout << sv << '\n';
}

int main() {
    using namespace std::string_view_literals;

    // Cheap Zero-Allocation Symbolic Constant
    constexpr std::string_view sv { "Static Read-Only Text"sv };

    // Substring Windowing without Allocation
    std::string text { "Hello World" };
    std::string_view view { text };
    
    view.remove_prefix(1); // Drops 'H' -> "ello World"
    view.remove_suffix(3); // Drops "rld" -> "ello Wo"

    // ----------------------------------------------------
    // ⚠️ CRITICAL PITFALLS (Undefined Behavior)
    // ----------------------------------------------------
    
    // Danger 1: Viewing Temporary std::string Objects
    // std::string_view badView { "Temp String"s }; 
    // std::cout << badView; // BUG: Temporary string was destroyed!

    // Danger 2: Invalidation via Modification
    // std::string base { "Buffer Text" };
    // std::string_view viewBase { base };
    // base = "Reallocating large text triggers memory reallocation!";
    // std::cout << viewBase; // BUG: viewBase points to deallocated heap address!
}
```

---

## 🧠 5. String Decision Matrix

```text
                           [ String Choice Tree ]
                                     |
                   Is this a function parameter?
                   /                           \
                (Yes)                          (No)
                 /                               \
   Is read-only access needed?        Do you need to store / modify data?
     /                  \               /                           \
  (Yes)                 (No)         (Yes)                          (No)
   /                      \           /                               \
std::string_view      std::string&  std::string             std::string_view
(C++17 zero-copy)     (In/Out)      (Owner of text)         (Symbolic constant)
```

### ⚡ Summary Checklist
1. **Function Parameters:**
   - Use `std::string_view` by value for read-only string parameters.
   - Use `const std::string&` only if passing to external legacy code requiring C-style null-termination.
2. **Function Returns:**
   - Return `std::string` by value for local variables (leveraging move semantics).
   - Return `std::string_view` **only** for C-string literals or `std::string_view` parameters passed in that outlive the call. **Never return a view of a local variable!**
3. **Variables:**
   - Use `std::string` when managing owned or dynamic input.
   - Use `constexpr std::string_view` for symbolic text constants.
