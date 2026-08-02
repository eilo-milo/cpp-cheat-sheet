# 11.1–11.9 — Function Overloading & C++ Templates

A comprehensive technical reference covering function overloading mechanics, overload resolution hierarchies, deleted functions, default arguments, and the generic programming architecture of C++ templates.

---

## 1. Function Overloading & Differentiation

**Function overloading** allows multiple functions in the same scope to share the exact same identifier, provided the compiler can structurally differentiate their signatures.

### The Differentiation Matrix

| Function Header Attribute | Can Overload/Differentiate? | Technical Notes |
| :--- | :--- | :--- |
| **Number of Parameters** | **Yes** | Standard parameter count variance. |
| **Type of Parameters** | **Yes** | Distinct parameter types differentiate overloads. |
| **Return Type** | **No** | Return types are **ignored** during differentiation. |
| **Type Aliases / Typedefs** | **No** | Aliases are not distinct types (`Age` vs `int` is ambiguous). |
| **Value Parameter Constness**| **No** | `const int` passed by value is not distinct from `int`. |

```cpp
// ✅ Valid differentiation by parameter count and type
int add(int x, int y);
double add(double x, double y);
int add(int x, int y, int z);

// ❌ Invalid: Differs ONLY by return type (Compile Error)
int getRandomValue();
double getRandomValue(); 

// ❌ Invalid: Type aliases do not create distinct types
using Age = int;
void print(int value);
void print(Age value); // Compile Error: Redefinition
```

* **Type Signature:** The unique set of header parameters (identifier, parameter types, count, and constness modifiers) used by the compiler to identify a function.
* **Name Mangling:** The compilation process where function signatures are transformed into unique internal symbol names (e.g., `__add_ii` vs `__add_dd`) so the linker can differentiate overloads.

---

## 2. Overload Resolution Sequence

When an overloaded function is called, the compiler resolves the call using a strict, 6-step argument matching hierarchy:

1. **Exact Match:** Matches raw types exactly or via trivial conversions (e.g., lvalue to rvalue, non-const to const, or non-reference to reference).
2. **Numeric Promotion:** Promotes narrow integral or floating-point types to wider base types (`char`/`bool` to `int`, `float` to `double`).
3. **Numeric Conversion:** Applies standard conversions (e.g., `double` to `int`, `int` to `float`, or `long` to `double`).
4. **User-Defined Conversion:** Applies class-defined implicit typecast operators or constructors.
5. **Ellipsis Match:** Matches functions using variadic ellipsis (`...`) parameters.
6. **Compile Error:** Build fails if no match is found.

### Ambiguous Matches
An **ambiguous match** occurs when two or more candidate functions match equally well at the *same step* of the resolution sequence.

```cpp
void print(unsigned int);
void print(float);

int main()
{
    // ❌ Ambiguous Match: '0' (int) can convert to 'unsigned int' or 'float' in Step 3
    // print(0); 

    // ✅ Resolution via explicit casting or literal suffixes
    print(static_cast<unsigned int>(0)); 
    print(0u); 
}
```

---

## 3. Deleting Functions (`= delete`)

The `= delete` specifier explicitly forbids callers from invoking specific function signatures. 

> 💡 **Key Insight:** `= delete` means "I forbid this call", not "this function does not exist". Deleted functions actively participate in overload resolution. If selected as the best match, compilation fails immediately.

```cpp
#include <iostream>

void printInt(int x) { std::cout << x << '\n'; }

// Explicitly forbid char and bool arguments
void printInt(char) = delete; 
void printInt(bool) = delete; 

int main()
{
    printInt(97);   // ✅ Valid: Calls printInt(int)
    // printInt('a');  // ❌ Compile Error: Function explicitly deleted
    // printInt(true); // ❌ Compile Error: Function explicitly deleted
}
```

---

## 4. Default Arguments

A **default argument** is a pre-specified parameter value automatically inserted by the compiler at the function call site if omitted by the caller.

```cpp
#include <iostream>

void print(int x, int y = 4); // Best Practice: Declare defaults in header/forward declaration

void print(int x, int y)
{
    std::cout << "x: " << x << " | y: " << y << '\n';
}

int main()
{
    print(1, 2); // Explicit arguments: x = 1, y = 2
    print(3);    // Default inserted:   x = 3, y = 4
}
```

* **Rightmost Rule:** If a parameter receives a default value, all trailing parameters to its right **must** also have default values.
* **Ambiguity Risk:** Overloaded functions with default arguments can easily collide:
  ```cpp
  void foo(int x = 0);
  void foo(double d = 0.0);

  // foo(); // ❌ Compile Error: Ambiguous call (could match either default)
  ```

---

## 5. Introduction to C++ Function Templates

**Function templates** act as stencils for generating generic code. Instead of manually writing multiple identical functions for different types, a single template definition allows the compiler to generate function specializations dynamically.

```cpp
#include <iostream>

// Primary Template Declaration
template <typename T> // 'T' is a type template parameter
T max(T x, T y)
{
    return (x < y) ? y : x;
}

int main()
{
    // Explicit type specification
    std::cout << max<int>(1, 2) << '\n';

    // Template Argument Deduction (Compiler infers 'double')
    std::cout << max(1.5, 2.5) << '\n'; 
}
```

### Template Argument Deduction Rules
* When called without explicit angled brackets (`max(1, 2)`), the compiler deduces `T` directly from argument types.
* **Non-Template Preference:** If a non-template overload and a template specialization match a call equally well, the **non-template function is preferred**.

---

## 6. Multi-Type Templates & Abbreviated Templates

If a template function requires independent parameters that may differ in type, supply multiple type parameters (`typename T, typename U`).

```cpp
#include <iostream>

template <typename T, typename U>
auto max(T x, U y) // 'auto' deduces common return type to avoid narrowing
{
    return (x < y) ? y : x;
}

int main()
{
    std::cout << max(2, 3.5) << '\n'; // Deduces T = int, U = double -> returns 3.5
}
```

### Abbreviated Function Templates (C++20)
C++20 simplifies multi-type template syntax by allowing `auto` in parameter lists:

```cpp
// C++20 Abbreviated Function Template
auto max(auto x, auto y) // Implicitly creates template <typename T, typename U>
{
    return (x < y) ? y : x;
}
```

> 💡 **Best Practice:** Use abbreviated function templates freely when parameters are intended to vary independently. If parameters MUST enforce identical types, stick to traditional `template <typename T>` syntax.

---

## 7. Non-Type Template Parameters

A **non-type template parameter** is a parameter with a fixed type that holds a compile-time `constexpr` value rather than a type.

```cpp
#include <iostream>

template <int N> // 'N' is a non-type template parameter
void printNumber()
{
    std::cout << "Compile-time constant: " << N << '\n';
}

// C++17 'auto' non-type parameter deduction
template <auto N>
void printAuto()
{
    std::cout << N << '\n';
}

int main()
{
    printNumber<5>();   // Instantiates printNumber<5>()
    printAuto<'c'>();   // Deduces N as char 'c'
}
```

* **Primary Use Case:** Non-type template parameters are used when values are strictly required at compile-time (such as `static_assert` checks or buffer sizes like `std::bitset<8>`).

---

## 8. Multi-File Template Architecture

Function templates cannot easily separate forward declarations in `.h` files from definitions in `.cpp` files. 

* **The Linker Failure:** Compiling a `.cpp` file that calls a template only sees the header's forward declaration. The translation unit containing the template definition cannot see the target instantiation types, causing the compiler to omit binary code generation, resulting in an `unresolved external symbol` linker error.
* **The Header-Only Solution:** Place complete template definitions inside header files (`.h`) and `#include` them wherever needed.

### ODR & Inline Exemption
* Template definitions themselves are exempt from standard single-definition limits across translation units.
* Implicitly instantiated template specializations are **implicitly inline**, preventing ODR violations when included across multiple source files.

```cpp
// max.h
#ifndef MAX_H
#define MAX_H

template <typename T>
T max(T x, T y) // Full definition MUST reside in header file
{
    return (x < y) ? y : x;
}

#endif
```
