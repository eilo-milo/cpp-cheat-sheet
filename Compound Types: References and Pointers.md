# C++ Cheat Sheet: Compound Types, References, Pointers & std::optional

A comprehensive reference guide covering C++ compound data types, expression value categories (lvalues vs rvalues), references, pointers, parameter-passing semantics, return types, auto deduction rules, and `std::optional`.

---

## 1. Type Systems & Expression Value Categories

### Data Type Classification
* **Fundamental Types:** Core language primitives (`int`, `double`, `char`, `bool`, `void`).
* **Compound Types:** Types built on top of other underlying types.
  * *Categories:* Functions, Arrays, Reference Types (`&`, `&&`), Pointer Types (`*`), Enumerations (`enum`), Class Types (`struct`, `class`, `union`).

### Properties of Expressions
Every C++ expression possesses two compile-time properties:
1. **Type:** The data type resulting from evaluating the expression.
2. **Value Category:** Dictates how an expression behaves during assignment, evaluation, and lifetime binding.

| Value Category | Definition | Lifetime | Examples |
| :--- | :--- | :--- | :--- |
| **Lvalue** (Locator Value) | Evaluates to an **identifiable object or function** with a distinct memory location. | Outlives the single expression statement. | Named variables (`x`), function calls returning references, `++x`, string literals (`"hello"`). |
| **Rvalue** | Evaluates to a **temporary value** without a persistent identity/address. | Expires at the end of the full expression statement. | Literals (`5`, `3.14`), temporary object instances, arithmetic results (`x + 1`), `x++`, return of value-returning functions. |

> **Key Rule:** Modifiable lvalues are required on the **left side** of assignment operators (`x = 5`). An lvalue implicitly converts to an rvalue via **lvalue-to-rvalue conversion** when evaluated for its value on the right side of an expression.

---

## 2. Lvalue References (`&`)

An lvalue reference creates an immutable alias for an existing modifiable lvalue object.

```cpp
#include <iostream>

int main() {
    int x { 5 };
    int& ref { x }; // ref is an alias for x

    ref = 10;       // Modifies x to 10
    std::cout << x; // Prints 10
}
```

### Reference Binding Rules
* **Non-const lvalue references (`T&`):** Must bind **only** to modifiable lvalues of matching types. They cannot bind to const lvalues, different types, or rvalues.
* **Reseating:** References **cannot** be reseated to refer to another object after initialization. Assigning a value to a reference modifies the referent object itself.
* **Lifetimes:** A reference and its referent have independent lifetimes. Accessing a reference after its referent is destroyed causes **undefined behavior** (Dangling Reference).

---

## 3. Lvalue References to Const (`const T&`)

Declaring an lvalue reference with `const` treats the referenced object as read-only through that specific alias.

```cpp
#include <iostream>

int main() {
    const int x { 5 };
    const int& ref1 { x }; // Binds to const lvalue

    int y { 10 };
    const int& ref2 { y }; // Binds to non-const lvalue (read-only access via ref2)
    // ref2 = 20;          // COMPILE ERROR: Cannot modify through const reference

    const int& ref3 { 42 }; // Binds to rvalue temporary
}
```

### Lifetime Extension Rule
When a `const` lvalue reference is bound directly to an rvalue temporary, **the lifetime of the temporary object is extended** to match the scope lifetime of the reference.

> **Best Practice:** Favor `const T&` over non-const `T&` for read-only references and function parameters.

---

## 4. Parameter Passing Mechanics Matrix

| Pass Mechanism | Syntax | Copy Cost | Accepts Rvalues? | Can Modify Argument? | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Pass by Value** | `void f(T x)` | Low (Small) / High (Class) | Yes | No (Modifies local copy) | Fundamental types, small cheap-to-copy types. |
| **Pass by Non-Const Ref** | `void f(T& x)` | Zero (Alias) | No | **Yes** | In-out parameters, performance-critical out-parameters. |
| **Pass by Const Ref** | `void f(const T& x)` | Zero (Alias) | Yes | No | Class types, large structures, read-only parameters. |
| **Pass by Address** | `void f(const T* x)` | Low (Pointer copy) | No (Requires `&` lvalue) | Optional (Depends on pointer constness) | Optional parameters (allows passing `nullptr`). |

### String Parameter Best Practice
Prefer passing string parameters using `std::string_view` by value over `const std::string&`, unless calling external/legacy C-APIs requiring null-terminated strings or `std::string` explicitly.

```cpp
#include <iostream>
#include <string_view>

// Preferred modern string parameter passing mechanism
void printText(std::string_view sv) {
    std::cout << sv << '\n';
}
```

---

## 5. Pointers (`*`)

A pointer is a variable object that holds a **memory address** of another object or function.

```cpp
#include <iostream>

int main() {
    int x { 5 };
    int* ptr { &x }; // ptr holds the memory address of x

    std::cout << ptr << '\n';  // Prints address (e.g., 0x7ffd16)
    std::cout << *ptr << '\n'; // Dereferences pointer: prints 5

    *ptr = 10;                 // Modifies x via pointer dereference
}
```

### References vs. Pointers Comparison

| Property | Reference (`T&`) | Pointer (`T*`) |
| :--- | :--- | :--- |
| **Is an Object?** | No | **Yes** (Size: 4 bytes on 32-bit, 8 bytes on 64-bit) |
| **Can be Null?** | No (Must bind to a valid object) | **Yes** (`nullptr`) |
| **Can Reseat?** | No | **Yes** (Can assign new addresses) |
| **Syntax Usage** | Implicit dereferencing | Explicit dereference (`*`) and address-of (`&`) |

---

## 6. Null Pointers & Safety

A null pointer holds a special value indicating that it is not pointing to any valid object.

```cpp
#include <iostream>

int main() {
    int* ptr { nullptr }; // Explicitly initialize as null pointer

    if (ptr) { // Implicit conversion: false if nullptr, true if valid
        std::cout << *ptr << '\n';
    } else {
        std::cout << "Pointer is null\n";
    }
}
```

> **Warning:** Dereferencing a `nullptr` or a **dangling pointer** results in **undefined behavior** (typically an immediate runtime crash). Always perform null checks before dereferencing raw pointers.

---

## 7. Const Pointer Syntax Matrix

The relative position of the `const` keyword to the asterisk (`*`) controls the immutability of the pointer address versus the underlying value.

```cpp
int v { 5 };

int* ptr0 { &v };             // Non-const pointer to non-const int (Can modify address and value)
const int* ptr1 { &v };       // Non-const pointer to CONST int (Can modify address, NOT value)
int* const ptr2 { &v };       // CONST pointer to non-const int (Can modify value, NOT address)
const int* const ptr3 { &v }; // CONST pointer to CONST int (Cannot modify address OR value)
```

> **Memory Rule:**
> * `const` to the **left** of `*` $\rightarrow$ Points to a constant **value**.
> * `const` to the **right** of `*` $\rightarrow$ The **pointer variable itself** is constant.

---

## 8. Return Types & Lifetime Management

### Return by Reference / Return by Address Rules

```cpp
// SAFE: Returns reference to static object outliving function scope
const std::string& getProgramName() {
    static const std::string name { "CalculatorApp" };
    return name; // OK: Static duration
}

// DANGER: Undefined Behavior
const std::string& badReturn() {
    std::string local { "TempText" };
    return local; // BUG: Returns dangling reference to local stack variable!
}
```

> **Critical Rule:** Never return local automatic variables or temporaries by reference or address. Reference lifetime extension **does not apply across function boundaries**.

---

## 9. Auto Type Deduction Rules (`auto`, `auto&`, `auto*`)

When deducing types with `auto`, the compiler strips **top-level const** and **reference** qualifiers unless explicitly reapplied at the definition site.

```cpp
#include <string>

const std::string& getConstRef();
std::string* getPtr();

int main() {
    // 1. Reference & Top-Level Const Dropped
    auto v1 { getConstRef() };        // Type: std::string
    const auto v2 { getConstRef() };  // Type: const std::string

    // 2. References Reapplied
    auto& v3 { getConstRef() };       // Type: const std::string& (Low-level const retained)
    const auto& v4 { getConstRef() }; // Type: const std::string&

    // 3. Pointer Deduction
    auto p1 { getPtr() };             // Type: std::string*
    auto* p2 { getPtr() };            // Type: std::string* (Explicit pointer form)
    const auto* p3 { getPtr() };      // Type: const std::string* (Pointer to const)
    auto* const p4 { getPtr() };      // Type: std::string* const (Const pointer)
}
```

---

## 10. Optional Values via `std::optional` (C++17)

`std::optional<T>` represents a value wrapper that may or may not contain a valid value of type `T`. It eliminates the need for sentinel error values or returning pointers just to express nullability.

```cpp
#include <iostream>
#include <optional>

// Returns an optional value to express failure safely
std::optional<int> divide(int x, int y) {
    if (y == 0)
        return std::nullopt; // Or return {}
    return x / y;
}

int main() {
    std::optional<int> result = divide(10, 2);

    if (result.has_value()) { // Or `if (result)`
        std::cout << "Success: " << *result << '\n'; // Dereference or result.value()
    } else {
        std::cout << "Division by zero\n";
    }

    // Safely extract value or fallback to a default
    int val = divide(5, 0).value_or(-1); // Returns -1
}
```

### Best Practice Rules for `std::optional`
* **Optional Return Values:** Use `std::optional<T>` for functions that may fail to calculate a valid return value.
* **Optional Parameters:** Prefer function overloading for optional input parameters. Use `std::optional<T>` for cheap-to-copy types, or `const T*` if `T` is expensive to copy.
