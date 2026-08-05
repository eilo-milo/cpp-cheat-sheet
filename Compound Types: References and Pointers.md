# C++ Complete Architecture Reference: Compound Types, Memory Mechanics, & Type Systems

An exhaustive, production-grade reference manual detailing the C++ type hierarchy, expression value categories, reference binding internals, pointer mechanics, argument-passing benchmarks, lifetime management, auto-deduction type mechanics, and nullability abstractions.

---

## 1. C++ Type System Architecture

The C++ language standard strictly classifies every type into either a **Fundamental Type** or a **Compound Type** ([basic.types]).

```
                                      [ C++ Type System ]
                                               │
               ┌───────────────────────────────┴───────────────────────────────┐
               │                                                               │
      Fundamental Types                                                 Compound Types
(Core language primitives)                                     (Constructed from existing types)
               │                                                               │
 ├── Void (`void`)                                              ├── Reference Types (`T&`, `T&&`)
 ├── Null Pointer (`std::nullptr_t`)                            ├── Pointer Types (`T*`, `T* const`, etc.)
 ├── Arithmetic Types                                           ├── Enumerated Types (`enum`, `enum class`)
 │    ├── Boolean (`bool`)                                      ├── Array Types (`T[]`, `T[N]`)
 │    ├── Character Types (`char`, `wchar_t`, `char8_t`, etc.)  ├── Function Types (`R(Args...)`)
 │    ├── Signed Integers (`short`, `int`, `long`, `long long`) └── Class Types (`struct`, `class`, `union`)
 │    ├── Unsigned Integers (`unsigned int`, `std::size_t`)
 │    └── Floating-Point (`float`, `double`, `long double`)
```

### Key Differences
* **Fundamental Types** represent single, indivisible scalar values implemented directly at the machine instruction level (register operations).
* **Compound Types** wrap, aggregate, or reference existing fundamental or compound types, defining structural relationships and memory layouts.

---

## 2. Expression Mechanics: Types & Value Categories

In C++, an **expression** is a sequence of operators and operands that specifies a computation. Every expression is characterized by two independent properties:
1. **Type:** The data type resulting from evaluating the expression (determined at compile time).
2. **Value Category:** Governs how the compiler manages the evaluation, assignment, identity, and temporary materialization of the expression during execution.

### Complete Value Category Taxonomy

```
                                [ Value Categories (Glvalue) ]
                                              │
                      ┌───────────────────────┴───────────────────────┐
                      │                                               │
                   Lvalue                                          xvalue
          (Identity, cannot move)                        (Identity, can move)
                      │                                               │
                      └───────────────────────┬───────────────────────┘
                                              │
                                           Glvalue
                                 (Generalized Locator Value)
                                              │
                      ┌───────────────────────┴───────────────────────┐
                      │                                               │
                   Glvalue                                         prvalue
          (Has memory address)                           (Pure temporary value)
                                              │
                                              └───────────┬───────────┘
                                                          │
                                                       Rvalue
                                              (Movable from)
```

### Value Categories Comparison Matrix

| Value Category | Identity? | Movable? | Memory Address? | Standard Definition & Behavior | Practical Code Examples |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Lvalue** | **Yes** | No | **Yes** (`&expr` is valid) | Designates a persistent object or function. Survives past the expression. | Variables (`x`), lvalue reference returns (`ref`), `++x`, string literals (`"hello"`). |
| **prvalue** | No | **Yes** | No (No persistent address) | Computes a value that initializes an object or serves as an operand. | Literals (`42`, `3.14`), `x + y`, `x++`, pass-by-value function returns. |
| **xvalue** | **Yes** | **Yes** | **Yes** | An **eXpiring** object with identity whose resources can be reused/moved. | `std::move(x)`, explicit cast to rvalue reference `static_cast<T&&>(var)`. |

> **Crucial Rules:**
> 1. Assignment operators require a **modifiable lvalue** on the left-hand side (`lhs = rhs`).
> 2. When an lvalue is evaluated in a context expecting a value (like `x + 5`), it undergoes **lvalue-to-rvalue conversion** to load the value from memory into a CPU register.

---

## 3. Lvalue References (`&`): Mechanics & Binding Rules

An lvalue reference (`T&`) creates an alias for an existing lvalue object. It does not create a new object in memory; it acts as a direct alias to its referent.

```cpp
#include <iostream>

int main() {
    int x { 5 };
    int& ref { x }; // ref is bound to x

    ref = 10;       // Modifies x through the alias
    std::cout << x; // Prints 10
}
```

### Deep Dive: Reference Rules & Invariants
1. **Mandatory Initialization:** A reference must be bound to an object upon declaration (`int& ref;` triggers a compile error).
2. **Reseating Impossibility:** References cannot be "reseated" to point to a different object after initialization. Writing `ref = y` assigns the value of `y` into the object currently referenced by `ref`.
3. **Underlying Implementation:** References are purely a compile-time alias concept. In machine code, compilers optimize references out completely. When forced to store them (e.g., as class fields), compilers implement references as `T* const` (a constant pointer).
4. **Dangling References:** Accessing a reference after its referent has been deallocated triggers **Undefined Behavior (UB)**.

```cpp
int& getDanglingReference() {
    int localVariable { 42 };
    return localVariable; // UB: localVariable dies at function scope exit!
}
```

---

## 4. References to Const (`const T&`) & Lifetime Extension

Declaring an lvalue reference with `const` disables writing through that alias, exposing a read-only interface to the referent.

```cpp
#include <iostream>

int main() {
    const int x { 5 };
    const int& ref1 { x }; // OK: Binding to const lvalue

    int y { 10 };
    const int& ref2 { y }; // OK: Binding to non-const lvalue (read-only through ref2)
    // ref2 = 20;          // COMPILE ERROR: Cannot modify through const reference

    const int& ref3 { 42 }; // OK: Binds directly to an rvalue temporary
}
```

### Temporary Lifetime Extension Rule
Binding a `const T&` directly to a prvalue temporary **extends the lifetime of that temporary object** to match the scope of the `const` reference itself.

```cpp
{
    const int& ref = 100 + 200; // Temporary int(300) created in memory
    std::cout << ref << '\n';   // Safe: Temporary exists as long as ref exists
} // Temporary object destroyed here along with ref
```

> **Warning:** Lifetime extension **does not cross function boundaries**. Returning a `const T&` bound to a local temporary inside a function returns a dangling reference.

---

## 5. Argument-Passing Strategies & Performance Benchmarks

Selecting the correct parameter-passing mechanism directly affects performance, memory layout, and function semantics.

| Strategy | Signature Syntax | Copy Cost | Accepts Rvalues? | Can Mutate Caller Data? | Ideal Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Pass-by-Value** | `void f(T x)` | Copy / Move | Yes | No (Operates on copy) | Primitive types (`int`, `double`), small types ($\le 2$ words), sink parameters. |
| **Pass-by-Lvalue-Ref** | `void f(T& x)` | Zero Copy | No | **Yes** | In-out parameters, mutating existing structures. |
| **Pass-by-Const-Ref** | `void f(const T& x)` | Zero Copy | Yes | No | Read-only access to large structures/class objects. |
| **Pass-by-Address** | `void f(const T* x)` | Pointer Copy | No (Requires `&`) | Optional (Depends on `const`) | Optional parameters (allows passing `nullptr`). |

### Modern String Parameter Idioms

```cpp
#include <iostream>
#include <string>
#include <string_view>

// PREFERRED: Zero-copy parameter accepting std::string, std::string_view, and C-style literals
void printModern(std::string_view sv) {
    std::cout << sv << '\n';
}

// DISCOURAGED: Forces expensive heap allocations when called with C-style string literals ("hello")
void printLegacy(const std::string& s) {
    std::cout << s << '\n';
}
```

---

## 6. Pointers (`*`): Mechanics & Memory Representation

A pointer is a first-class variable whose stored value is a **raw memory address**.

```cpp
#include <iostream>

int main() {
    int x { 42 };
    int* ptr { &x }; // ptr holds the memory address of x

    std::cout << ptr << '\n';  // Prints address (e.g., 0x7ffd16a4)
    std::cout << *ptr << '\n'; // Dereferences ptr: evaluates to 42

    *ptr = 99;                 // Modifies x via pointer dereference
}
```

### Architectural Comparison: References vs. Pointers

| Dimension | Reference (`T&`) | Pointer (`T*`) |
| :--- | :--- | :--- |
| **Entity Type** | Syntactic Alias (Not an object) | First-class Object (Has its own address & size) |
| **Nullability** | Cannot be null | Can be null (`nullptr`) |
| **Reassignment** | Cannot be reseated | Can be assigned new addresses |
| **Syntax** | Implicit dereferencing | Explicit dereference (`*`) and address-of (`&`) |
| **Storage Overhead** | Typically 0 bytes | 4 bytes (32-bit CPU) / 8 bytes (64-bit CPU) |

---

## 7. Null Pointer Mechanics & Safety Guarantees

`nullptr` is a keyword representing a null pointer literal of type `std::nullptr_t`. It explicitly indicates that a pointer is not pointing to a valid memory location.

```cpp
#include <iostream>

void processPointer(const int* ptr) {
    // Null precondition check
    if (!ptr) {
        std::cout << "Error: Null pointer provided.\n";
        return;
    }
    std::cout << "Value: " << *ptr << '\n';
}

int main() {
    int* ptr { nullptr }; // Explicitly initialized null pointer
    processPointer(ptr);
}
```

> **Safety Hazard:** Dereferencing a `nullptr` or a dangling pointer triggers **Undefined Behavior** (typically a Segmentation Fault). Always perform null-checks before dereferencing raw pointers.

---

## 8. Const Pointer Mutability Matrix

The position of the `const` keyword relative to the asterisk (`*`) determines whether the **pointer address** or the **pointed-to value** is immutable.

```cpp
int x { 10 };
int y { 20 };

// 1. Non-const pointer to non-const data
int* p0 { &x };
*p0 = 15; // OK
p0 = &y;  // OK

// 2. Non-const pointer to CONST data (Pointer to const)
const int* p1 { &x };
// *p1 = 15; // ERROR: Pointed-to data is read-only
p1 = &y;     // OK: Address can change

// 3. CONST pointer to non-const data (Const pointer)
int* const p2 { &x };
*p2 = 15;    // OK: Pointed-to data can change
// p2 = &y;  // ERROR: Pointer address is immutable

// 4. CONST pointer to CONST data
const int* const p3 { &x };
// *p3 = 15; // ERROR: Pointed-to data is read-only
// p3 = &y;  // ERROR: Pointer address is immutable
```

```
                 [ Const Pointer Reading Rule ]

    const int * const ptr;
    ─────┬─── ─ ───┬────
         │         │
         │         └─► Const right of *: The pointer address is immutable
         └───────────► Const left of *: The target data is immutable
```

---

## 9. Auto Type Deduction Rules (`auto`, `auto&`, `auto*`)

Type deduction using `auto` strips **top-level const** and **references** by default. Qualifiers must be explicitly reapplied if desired.

```cpp
#include <string>

const std::string& getConstRef();
std::string* getPtr();

int main() {
    // 1. Reference & Top-Level Const Dropped
    auto v1 { getConstRef() };        // Type: std::string (Copies value)
    const auto v2 { getConstRef() };  // Type: const std::string

    // 2. References Reapplied (Preserves Low-Level Const)
    auto& v3 { getConstRef() };       // Type: const std::string&
    const auto& v4 { getConstRef() }; // Type: const std::string&

    // 3. Pointer Deduction Variants
    auto p1 { getPtr() };             // Type: std::string*
    auto* p2 { getPtr() };            // Type: std::string* (Ensures pointer type)
    const auto* p3 { getPtr() };      // Type: const std::string* (Pointer to const)
    auto* const p4 { getPtr() };      // Type: std::string* const (Const pointer)
}
```

### Definitions: Top-Level vs. Low-Level Const
* **Top-Level Const:** Applies to the object itself (e.g., `int* const ptr` or `const int x`). **Stripped by `auto`**.
* **Low-Level Const:** Applies to the target being pointed to or referenced (e.g., `const int* ptr` or `const int& ref`). **Retained by `auto`**.

---

## 10. `std::optional<T>` (C++17)

`std::optional<T>` is a value-semantic template wrapper that contains either a valid value of type `T` or an empty state (`std::nullopt`). It replaces sentinel error values (e.g., `-1`, `nullptr`) with a type-safe interface.

```cpp
#include <iostream>
#include <optional>
#include <string>

// Returns an optional value to express potential runtime failure safely
std::optional<int> parseAge(std::string_view str) {
    if (str.empty())
        return std::nullopt; // Expresses absence of a value
    
    int age = std::stoi(std::string(str));
    if (age < 0 || age > 150)
        return {}; // Equivalent to std::nullopt
        
    return age;
}

int main() {
    auto ageOpt = parseAge("25");

    // Check if a value is present
    if (ageOpt.has_value()) { // Or `if (ageOpt)`
        std::cout << "Parsed Age: " << *ageOpt << '\n'; // Dereference access
        std::cout << "Parsed Age: " << ageOpt.value() << '\n'; // Throws std::bad_optional_access if empty
    }

    // Extract value with a fallback default
    int finalAge = parseAge("invalid").value_or(0); // Returns 0
}
```

### Guidelines for `std::optional`
1. **Optional Return Values:** Use `std::optional<T>` for functions that can fail to calculate a result where all values of `T` represent valid results.
2. **Optional Parameters:** Prefer function overloading for optional input parameters. Use `std::optional<T>` for cheap-to-copy types, or `const T*` / `std::string_view` if `T` is expensive to copy.
3. **Memory Overhead:** `std::optional<T>` requires `sizeof(T) + sizeof(bool)` plus alignment padding.
