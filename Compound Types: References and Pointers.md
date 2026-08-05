# C++ Compound Types Cheat Sheet (References, Pointers, & Parameters)

A practical, detailed C++ reference guide focused on **lvalue/rvalue categories, references, pointers, function parameter modes (In, Out, In-Out), and std::optional**, based on modern C++ best practices.

---

## 1. Fundamental vs. Compound Types & Value Categories

### Fundamental vs. Compound Types
* **Fundamental Types:** Core built-in types (`int`, `double`, `char`, `bool`).
* **Compound Types:** Types constructed from existing types (Functions, References, Pointers, Arrays, Structs, Classes, Enums).

### Expression Properties: Type & Value Category
Every C++ expression has two primary properties:
1. **Type:** Data type resulting from evaluating the expression (determined at compile time).
2. **Value Category:** Determines how an expression can be evaluated and used in operations (e.g., assignment).

#### Value Categories (Pre-C++11 View)
* **Lvalue (Locator Value):** Evaluates to an **identifiable object or function** that persists beyond the expression. Has an accessible memory address.
  * **Modifiable Lvalue:** Non-const object (e.g., `int x;`).
  * **Non-modifiable Lvalue:** Const object (e.g., `const int x;`).
* **Rvalue:** Evaluates to a **value/temporary** that does not persist beyond the expression.
  * Examples: Literals (`5`, `1.2`), expression results (`x + 1`), values returned by value.
  * *Exception:* C-style string literals (`"Hello"`) are **lvalues** due to array-to-pointer decay.

```cpp
int x { 5 };       // 'x' is a modifiable lvalue; 5 is an rvalue
const int y { 2 }; // 'y' is a non-modifiable lvalue
int z { x + y };   // '(x + y)' is an rvalue expression
```

#### Lvalue-to-Rvalue Conversion
When an rvalue is expected but a modifiable or non-modifiable lvalue is provided, implicit conversion evaluates the lvalue to extract its underlying value.

```cpp
int x { 1 };
int y { 2 };
x = y; // 'y' is an lvalue, implicitly converted to rvalue (2) to assign to 'x'
```

---

## 2. Lvalue References (`T&`)

An lvalue reference acts as an **alias** for an existing, identifiable object.

```cpp
int x { 5 };
int& ref { x }; // ref is an alias for x

ref = 10;       // Modifies x to 10
```

### Key Rules
1. **Must be initialized:** Non-const references must bind to a **modifiable lvalue**.
2. **Cannot be reseated:** Once initialized, a reference always points to the same underlying object. Assignment modifies the *referent*, not the reference binding itself.
3. **Lifetimes are independent:** Destroying a reference does not affect the referent.
4. **Dangling Reference:** Occurs when the referent is destroyed before the reference. Accessing it is **Undefined Behavior (UB)**.

```cpp
int main()
{
    int x { 5 };
    int& ref { x };

    int y { 20 };
    ref = y; // Assigns the value of y (20) to x! Does NOT reseat ref to y.
}
```

---

## 3. Lvalue References to Const (`const T&`)

Can bind to **modifiable lvalues**, **non-modifiable lvalues**, and **rvalues**.

```cpp
int x { 5 };
const int& r1 { x }; // Bound to modifiable lvalue (read-only access via r1)

const int y { 10 };
const int& r2 { y }; // Bound to non-modifiable lvalue

const int& r3 { 15 }; // Bound to rvalue (temporary object created)
```

### Lifetime Extension Rule
When a `const` lvalue reference is bound directly to a temporary object (rvalue), the lifetime of the temporary is extended to match the lifetime of the reference.

```cpp
#include <iostream>

int main() {
    {
        const int& ref { 42 }; // Temporary object created, lifetime extended to match ref
        std::cout << ref << '\n'; // Safe
    } // Both ref and temporary object destroyed here
}
```

> **Warning:** Lifetime extension does **not** work across function boundaries when returning references to local temporaries!

---

## 4. Function Parameter Direction Modes (In, Out, In-Out)

Function parameters can be classified by data flow direction:

### Classification Summary

| Mode | Direction | Preferred Types | Primary Mechanism |
| :--- | :--- | :--- | :--- |
| **In Parameters** | Caller -> Function | Read-only input data | Value (`T`) or Const Ref (`const T&`) |
| **Out Parameters** | Function -> Caller | Return multiple results | Non-const Reference (`T&`) or Pointer (`T*`) |
| **In-Out Parameters** | Caller <-> Function | Modify existing data | Non-const Reference (`T&`) |

---

### Detailed Mechanics & Examples

#### 1. In Parameters (Input Only)
Used strictly to supply data into the function. The function does not modify the caller's copy.

* **Rule of Thumb:**
  * Fundamental types, Enums, and cheap-to-copy views (`std::string_view`) -> **Pass by Value**
  * Expensive Class Types (`std::string`, `std::vector`) -> **Pass by `const T&`**

```cpp
#include <iostream>
#include <string>
#include <string_view>

// In Parameter: Passed by value (cheap type)
void printInt(int x) { 
    std::cout << x << '\n';
}

// In Parameter: Passed by const reference (expensive type)
void printString(const std::string& str) { 
    std::cout << str << '\n';
}

// In Parameter: Modern string handling
void printSV(std::string_view sv) { 
    std::cout << sv << '\n';
}
```

#### 2. Out Parameters (Output Only)
Used to return information back to the caller when a single return value is insufficient.

* **Drawbacks of Out Parameters:**
  1. Unnatural syntax (requires pre-allocating mutable local variables before invocation).
  2. Cannot take temporaries or rvalue literals as arguments.
  3. Obscures whether arguments will be modified at the call site.
* **Modern Best Practice:** Prefer returning values via `std::tuple`, custom `struct`, or `std::optional` instead of using out parameters.

```cpp
#include <cmath>
#include <iostream>

// degrees is an 'In' parameter
// sinOut and cosOut are 'Out' parameters (modified inside function)
void getSinCos(double degrees, double& sinOut, double& cosOut) {
    constexpr double pi { 3.14159265358979323846 };
    double radians = degrees * pi / 180.0;

    sinOut = std::sin(radians); // Assigning return values via references
    cosOut = std::cos(radians);
}

int main() {
    double sin { 0.0 };
    double cos { 0.0 };

    // Caller MUST instantiate modifiable variables first
    getSinCos(45.0, sin, cos); 
}
```

#### 3. In-Out Parameters (Input & Output)
The function reads the initial value provided by the caller, modifies it, and passes the updated state back.

```cpp
#include <string>
#include <algorithm>

// 'strInOut' is an In-Out parameter
void sanitizeString(std::string& strInOut) {
    // Reads input data
    if (strInOut.empty()) return; 

    // Modifies caller's object directly
    std::ranges::transform(strInOut, strInOut.begin(), ::tolower); 
}

int main() {
    std::string text { "HELLO WORLD" };
    sanitizeString(text); // 'text' is updated in-place to "hello world"
}
```

---

## 5. Pointers (`T*`)

A pointer is an **object** that holds a memory address as its value.

```cpp
#include <iostream>

int main() {
    int x { 5 };
    int* ptr { &x }; // 'ptr' stores memory address of x

    std::cout << ptr << '\n';  // Prints address (e.g., 0x7ffeefbff5ac)
    std::cout << *ptr << '\n'; // Dereference operator (*): Prints value at address (5)

    *ptr = 10; // Changes value of x to 10
}
```

### Operators
* `&` (**Address-of operator**): Unary operator returning the memory address of its operand.
* `*` (**Dereference operator**): Unary operator returning the object/value located at the target address.

### Differences Between Pointers and References
| Feature | References (`T&`) | Pointers (`T*`) |
| :--- | :--- | :--- |
| **Is an Object?** | No (alias) | Yes (occupies memory: 4 bytes on 32-bit, 8 bytes on 64-bit) |
| **Can be Null?** | No | Yes (`nullptr`) |
| **Reseatable?** | No | Yes (can point to a different address) |
| **Syntax** | Implicit dereferencing | Explicit dereferencing via `*` |
| **Initialization** | Must be initialized | Optional (uninitialized = wild pointer) |

---

## 6. Null Pointers & Null Checking

A null pointer holds a special value meaning it points to no object.

```cpp
int* ptr { nullptr }; // Best Practice: Value-initialize or use nullptr
```

### Key Safety Rules
1. **Dereferencing a null pointer or dangling pointer is Undefined Behavior** (will likely crash your program).
2. Always null-check before dereferencing if there is any doubt.

```cpp
#include <iostream>

void processPointer(const int* ptr) { // In parameter via pointer
    // Implicit conversion to bool: nullptr -> false, valid address -> true
    if (!ptr) {
        return; // Early exit strategy
    }

    std::cout << "Value: " << *ptr << '\n';
}
```

---

## 7. Pointers and `const`

Read syntax **from right to left** to determine pointer `const`-ness:

```cpp
int value { 5 };

int* ptr0 { &value };             // Non-const pointer to non-const value: Everything can change
const int* ptr1 { &value };       // Pointer to CONST value: Cannot change *ptr1, CAN change ptr1
int* const ptr2 { &value };       // CONST pointer to non-const value: CAN change *ptr2, Cannot change ptr2
const int* const ptr3 { &value }; // CONST pointer to CONST value: Cannot change *ptr3 nor ptr3
```

### Quick Reference Table
| Type Declaration | Modify Value (`*ptr = x`) | Modify Pointer Target (`ptr = &y`) |
| :--- | :---: | :---: |
| `int* ptr` | Yes | Yes |
| `const int* ptr` | **No** | Yes |
| `int* const ptr` | Yes | **No** |
| `const int* const ptr` | **No** | **No** |

---

## 8. Pass by Address (`T*`)

Passing a memory address into a pointer function parameter.

```cpp
#include <iostream>

// Out/In-Out parameter using pass-by-address
void setToZero(int* ptr) {
    if (ptr) { // Null check
        *ptr = 0;
    }
}

int main() {
    int x { 10 };
    setToZero(&x); // Explicitly pass address using &
    std::cout << x << '\n'; // Prints 0
}
```

### Pass-by-Address by Reference (`T*&`)
If a function needs to change **where a pointer points to** in the caller's scope, pass the pointer by reference.

```cpp
void reassignPointer(int*& ptrRef, int* newTarget) {
    ptrRef = newTarget; // Modifies the actual pointer passed by caller
}
```

---

## 9. Returning Values: Reference vs. Address

### Return by Reference (`T&`)
Avoids copying the return value. The referent **MUST** outlive the function execution.

```cpp
#include <string>

// SAFE: Static variable outlives function
const std::string& getProgramName() {
    static const std::string name { "Calculator" };
    return name;
}

// SAFE: Reference parameters passed in outlive function execution
const std::string& maxString(const std::string& a, const std::string& b) {
    return (a > b) ? a : b;
}

// DANGEROUS: Local variable destroyed at end of scope -> Dangling Reference (UB!)
const std::string& getBadName() {
    std::string local { "Error" };
    return local; // UB!
}
```

### Return by Address (`T*`)
Used primarily when returning "no object" via `nullptr` is a valid return condition.
* **Rule:** Favor Return by Reference (`T&`) over Return by Address (`T*`) unless `nullptr` needs to be returned.

---

## 10. Type Deduction (`auto`) with References & Pointers

### Rules
1. **References are dropped** during `auto` deduction.
2. **Top-Level `const` is dropped** during `auto` deduction.
3. **Low-Level `const` is preserved**.
4. **Pointers are NOT dropped**.

### Terminology
* **Top-level `const`:** Applies to the variable/pointer itself (`int* const`).
* **Low-level `const`:** Applies to the referenced/pointed-to object (`const int*`, `const int&`).

```cpp
#include <string>

const std::string& getRef();
std::string* getPtr();

int main() {
    auto v1 { getRef() };        // std::string (ref dropped, top-level const dropped)
    const auto& v2 { getRef() }; // const std::string& (explicitly reapplied)

    auto p1 { getPtr() };        // std::string* (pointer retained)
    auto* p2 { getPtr() };       // std::string* (explicit pointer syntax - recommended)
    
    const auto* p3 { getPtr() }; // const std::string* (pointer to const)
    auto* const p4 { getPtr() }; // std::string* const (const pointer)
}
```

---

## 11. Optional Values: `std::optional<T>` (C++17)

Replaces sentinel return values (e.g., returning `-1` or `0.0` for errors) and pointer-based optional values.

```cpp
#include <iostream>
#include <optional>

// Function optionally returns an int value
std::optional<int> divide(int numerator, int denominator) {
    if (denominator == 0) {
        return std::nullopt; // Or return {}
    }
    return numerator / denominator;
}

int main() {
    std::optional<int> result { divide(10, 2) };

    if (result.has_value()) { // Or simply: if (result)
        std::cout << "Result: " << *result << '\n'; // Dereference or result.value()
    } else {
        std::cout << "Division by zero!\n";
    }

    // Default value fallback
    int val = divide(5, 0).value_or(-1); // Returns -1 if nullopt
}
```

### Guidelines for `std::optional`
* Use `std::optional<T>` as return types when a function might fail to return a value.
* Use `std::optional<T>` for optional parameters **only when T is cheap to copy** (since `std::optional` holds value semantics and makes copies).
* For expensive-to-copy optional function parameters, prefer **function overloading** or `const T*`.
