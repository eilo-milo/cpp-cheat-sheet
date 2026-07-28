# C++ Cheat Sheet: Constants, Optimizations & Strings

A reference guide covering compile-time programming, string types, memory mechanics, and performance best practices in C++.

---

## 1. Constants & Type Qualifiers

### Overview
C++ supports two types of constants:
* Named Constants: Symbolic identifiers linked to a fixed value (const, constexpr, enum, #define).
* Literal Constants: Values without identifiers written directly in code (5, "hello").

### Naming & Declaration Best Practices

// PREFERRED: Standard camelCase with const before type
const double earthGravity { 9.8 };

// ACCEPTABLE ("East Const"):
double const standardPressure { 101.325 };

// DISCOURAGED: C-style macro definitions
#define EARTH_GRAVITY 9.8 // Avoid! Ignores scope, breaks debuggers


### const vs constexpr

Qualifier: const
Meaning: Non-modifiable after initialization
Initializer Requirement: Resolved at runtime OR compile-time
Use Case: Runtime constants (e.g., user input)

Qualifier: constexpr
Meaning: Compile-time constant
Initializer Requirement: Must be a compile-time constant expression
Use Case: Array sizes, template bounds, mathematical constants

#include <iostream>

int getRuntimeValue() { return 42; }

int main() {
    // Runtime constant: initialized via runtime function call
    const int runtimeConst { getRuntimeValue() }; 

    // Compile-time constant: initialized via literal
    constexpr int compileConst { 100 }; 

    // Implicit constness: constexpr variables are implicitly const
    // compileConst = 200; // ERROR: assignment of read-only variable
}


Best Practice:
* Declare variables constexpr whenever their value can be calculated at compile time.
* Fall back to const for values that can only be determined at runtime.
* Avoid const for by-value function parameters and return types.

---

## 2. Literals, Suffixes & Numeral Systems

### Literal Types & Suffixes

// Integral Literals
auto a { 5 };     // int (default)
auto b { 5L };    // long (Prefer uppercase 'L' over 'l' to avoid visual confusion with '1')
auto c { 5u };    // unsigned int
auto d { 5ULL };  // unsigned long long
auto e { 5z };    // signed std::size_t (C++23)

// Floating-Point Literals
auto f1 { 4.1 };  // double (default)
auto f2 { 4.1f }; // float (Prevents implicit double-to-float narrowing conversion warnings)

// Scientific Notation
double avogadro { 6.02e23 };    // 6.02 * 10^23
double proton   { 1.6e-19 };    // 1.6 * 10^-19


### Numeral Systems & Separators (C++14 / C++20 / C++23)

#include <iostream>
#include <bitset>
#include <format> // C++20
#include <print>  // C++23

int main() {
    int dec { 12 };         // Base 10
    int oct { 014 };        // Base 8 (Prefix: 0) - Avoid! Easy to misread
    int hex { 0x0C };       // Base 16 (Prefix: 0x)
    int bin { 0b0000'1100 };// Base 2 (Prefix: 0b, C++14) - ' is a digit separator

    // Printing formats via stream manipulators
    std::cout << std::hex << dec << '\n'; // Output: c
    std::cout << std::dec << dec << '\n'; // Reset back to decimal

    // Modern C++ Binary Output
    std::cout << std::bitset<8>{ 0b0000'1100 } << '\n'; // std::bitset approach
    
    // std::cout << std::format("{:#b}\n", 12); // Output: 0b1100 (C++20)
    // std::println("{:#b}", 12);               // Output: 0b1100 (C++23)
}


---

## 3. Compiler Optimizations & Compile-Time Evaluation

### Key Concepts
* As-If Rule: The compiler can reorganize, replace, or eliminate any code provided the program's observable behavior remains unchanged.
* Constant Folding: Evaluation of constant expressions at compile-time instead of runtime.
* Constant Propagation: Replacing variable references directly with known constant values.
* Dead Code Elimination: Removing unused variables and unreachable branches.

#include <iostream>

int main() {
    // Unoptimized code pattern:
    const int x { 3 + 4 };
    std::cout << x << '\n';

    // What the optimizing compiler reduces it to:
    // 1. Constant Folding: 3 + 4 -> 7
    // 2. Constant Propagation: x -> 7
    // 3. Dead Code Elimination: variable x removed
    
    // Equivalent optimized machine codepath:
    std::cout << 7 << '\n';
}


---

## 4. Compile-Time Programming

### constexpr Functions

A constexpr function can evaluate at compile-time if all arguments are constant expressions and its output is required in a compile-time context.

#include <iostream>

constexpr int calcFactorial(int n) {
    return (n <= 1) ? 1 : (n * calcFactorial(n - 1));
}

int main() {
    // Guaranteed compile-time execution (Initializer of constexpr var)
    constexpr int f5 { calcFactorial(5) }; 

    int x = 5;
    // Runtime execution (Argument 'x' is non-const)
    int fRuntime { calcFactorial(x) }; 
}


---

## 5. Strings: std::string vs std::string_view

### Memory Mechanics Comparison

Feature: std::string
* Role: Dynamic Owner
* Allocation: Dynamic Heap Allocation (Large strings)
* Null-Terminated?: Always Guaranteed (\0)
* constexpr Support: Extremely limited (C++20/23)
* Mutation: Allowed

Feature: std::string_view
* Role: Non-owning Observer
* Allocation: Zero allocation (Pointer + Length pair)
* Null-Terminated?: Not guaranteed (Substrings)
* constexpr Support: Fully Supported (C++17)
* Mutation: Read-Only

---

### std::string Cheatsheet

#include <iostream>
#include <string>

int main() {
    // Literals with namespace
    using namespace std::string_literals;
    auto strObj { "Hello World"s }; // std::string type via 's' suffix

    // Input Handling
    std::string fullName {};
    std::cout << "Enter full name: ";
    // std::ws strips leftover newlines and whitespace prior to reading
    std::getline(std::cin >> std::ws, fullName);

    // Length Handling (Avoid Signed/Unsigned comparison bugs)
    int lenCast { static_cast<int>(fullName.length()) };
    // int lenSSize { static_cast<int>(std::ssize(fullName)) }; // C++20 signed size
}


---

### std::string_view Cheatsheet & Pitfalls

#include <iostream>
#include <string>
#include <string_view>

void printView(std::string_view sv) { // Fast: Pass by value without copy
    std::cout << sv << '\n';
}

int main() {
    using namespace std::string_view_literals;

    // Fast zero-copy string initialization
    constexpr std::string_view view { "Static Read-Only String"sv };
    
    std::string mutableStr { "Buffer Data" };
    std::string_view viewMut { mutableStr };

    // View manipulation (Does NOT modify 'mutableStr')
    viewMut.remove_prefix(1); // Drops first character
    viewMut.remove_suffix(2); // Drops last two characters
    std::cout << viewMut << '\n'; // Output: "uffer D"

    // ----------------------------------------------------
    // DANGER ZONE: Undefined Behavior Examples
    // ----------------------------------------------------
    
    // 1. Dangling View via Temporary std::string
    // std::string_view badView { "Temp String"s }; // Temp destroyed at end of line!
    // std::cout << badView; // UNDEFINED BEHAVIOR

    // 2. Invalidation via String Modification
    // std::string base { "Original" };
    // std::string_view sv { base };
    // base = "Reallocated Very Long String That Forces Heap Reallocation";
    // std::cout << sv; // UNDEFINED BEHAVIOR (sv points to deallocated memory)
}


---

## 6. Decision Matrix: When to Use Which

                            [ String Decision Tree ]
                                       |
                   Is this a function parameter?
                    /                         \
                 (Yes)                        (No)
                  /                             \
    Is read-only access needed?           Do you need to store / modify data?
      /                   \                 /                          \
   (Yes)                  (No)           (Yes)                         (No)
    /                       \             /                              \
std::string_view       std::string&    std::string             std::string_view
(C++17 zero-copy)      (In/Out Param)  (Owner of dynamic text) (Compile-time / view constant)


### Quick Rules of Thumb
1. Function Parameters:
   * Prefer std::string_view by value for read-only strings.
   * Use const std::string& if interfacing with legacy APIs requiring null-terminated C-strings.
   * Use std::string& only for out-parameters.

2. Function Return Types:
   * Return std::string by value when returning local variables or temporary string constructions (leverages move semantics/RVO).
   * Return std::string_view ONLY for C-style string literals, constexpr constants, or string inputs passed as std::string_view parameters that outlive the call. Never return a std::string_view pointing to local variables.

3. Variables:
   * Use std::string for dynamic input or owned/modifiable content.
   * Use constexpr std::string_view for string symbolic constants.
