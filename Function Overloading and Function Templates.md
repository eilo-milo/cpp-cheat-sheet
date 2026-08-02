# C++ Function Overloading & Templates Cheat Sheet

A reference summary covering C++ function overloading, overload resolution rules, function deletion (`= delete`), default arguments, function templates, template deduction, multi-type parameters, non-type template parameters (NTTP), and header-only template organization.

---

## 1. Function Overloading & Differentiation

### Core Concept
Function overloading allows multiple functions in the same scope to share the same name, provided the compiler can uniquely differentiate them by their parameter profiles.

```cpp
int add(int x, int y);          // Signature: add(int, int)
double add(double x, double y);   // Signature: add(double, double)
```

### Overload Differentiation Matrix

| Function Attribute | Used for Differentiation? | Notes & Exceptions |
| :--- | :--- | :--- |
| **Number of Parameters** | **Yes** | Evaluated at compile-time. |
| **Type of Parameters** | **Yes** | Includes distinct types and ellipses (`...`). |
| **Return Type** | **No** | Functions differing **only** by return type yield compile errors. |
| **Typedefs / Type Aliases** | **No** | `using Age = int;` is not a distinct type from `int`. |
| **Pass-by-Value `const`** | **No** | `void f(int)` and `void f(const int)` are identical overloads. |
| **Member `const` / Ref Qualifiers** | **Yes** | Applies exclusively to class member functions. |

---

## 2. Overload Resolution Rules

When an overloaded function is called, the compiler evaluates candidate functions sequentially through distinct conversion steps. Matching halts at the first step that finds a valid overload.

```text
[ Function Call Evaluation Pipeline ]
                │
    Step 1: Exact Match / Trivial Conversions (lvalue->rvalue, const qualifiers, ref conversions)
                │ (If no match)
    Step 2: Numeric Promotions (e.g., char/bool -> int, float -> double)
                │ (If no match)
    Step 3: Numeric Conversions (e.g., int -> double, double -> int, int -> unsigned int)
                │ (If no match)
    Step 4: User-Defined Conversions (Class constructors / typecast operators)
                │ (If no match)
    Step 5: Ellipsis Matches (...)
                │ (If no match)
      [ Compile Error: No Matching Function ]
```

> **Ambiguous Matches:** If multiple candidate overloads resolve within the **same step**, the compiler aborts with an ambiguous call error.
>
> **Disambiguation Fixes:**
> 1. Supply explicit static type casts: `foo(static_cast<unsigned int>(x))`.
> 2. Use literal suffixes: `foo(0u)`.
> 3. Define an exact-match function overload.

---

## 3. Deleted Functions (`= delete`)

The `= delete` specifier explicitly forbids specific function calls. Deleted functions **participate in overload resolution**; if matched, they halt compilation.

```cpp
void printInt(int x);

// Explicitly forbid problematic conversions
void printInt(char) = delete;
void printInt(bool) = delete;

// Forbid ALL non-int overloads via template deletion
template <typename T>
void printInt(T) = delete;
```

---

## 4. Default Arguments

Default arguments specify fallback parameter values at the call site if arguments are omitted by the caller.

```cpp
// PREFERRED: Specify default arguments in forward declarations / header files
void print(int x, int y = 4); 
```

### Syntax & Usage Rules
1. **Rightmost Rule:** If a parameter has a default argument, all subsequent parameters to its right **must** also have default arguments.
2. **No Redeclaration:** A default argument cannot be declared twice in the same translation unit.
3. **Ambiguity Risk:** Overloads sharing default argument fallback layouts can result in ambiguous calls:

```cpp
void foo(int x = 0);
void foo(double d = 0.0);

// foo(); // COMPILE ERROR: Ambiguous call!
```

---

## 5. Function Templates & Template Argument Deduction

Function templates act as blueprints for generating type-safe functions automatically at compile-time.

### Template Syntax & Instantiation

```cpp
template <typename T> // Preferred over 'class T'
T max(T x, T y)
{
    return (x < y) ? y : x;
}

int main()
{
    // Explicit Template Argument
    auto a = max<double>(2.0, 3.5); // Instantiates max<double>(double, double)

    // Template Argument Deduction (Compiler infers type automatically)
    auto b = max(1, 2);             // Instantiates max<int>(int, int)
}
```

> **Rule:** Normal non-template functions take precedence over an equally viable template specialization during overload resolution.

---

## 6. Multi-Type & Abbreviated Function Templates

### Multiple Template Type Parameters
Prevent deduction mismatches when function arguments have different types:

```cpp
template <typename T, typename U>
auto max(T x, U y) // Auto deduces common return type safely
{
    return (x < y) ? y : x;
}
```

### Abbreviated Function Templates (C++20)
Using `auto` in parameter lists automatically converts standard functions into template functions.

```cpp
// Concise shorthand for template <typename T, typename U> auto max(T x, U y)
auto max(auto x, auto y)
{
    return (x < y) ? y : x;
}
```

---

## 7. Non-Type Template Parameters (NTTP)

A non-type template parameter is a fixed-type placeholder for a `constexpr` value passed at compile-time.

```cpp
#include <iostream>

template <int N> // N is a compile-time constant
void printNTTP()
{
    std::cout << N << '\n';
}

// C++17 Auto NTTP Deduction
template <auto N>
void printAutoNTTP()
{
    std::cout << N << '\n';
}

int main()
{
    printNTTP<5>();       // Instantiates printNTTP<5>()
    printAutoNTTP<'c'>(); // Deduces char 'c'
}
```

---

## 8. Organization & The One Definition Rule (ODR)

### The Multi-File Linking Dilemma
Function template definitions must be visible to the compiler at the call site to instantiate code. Separating template declarations into `.h` files and definitions into `.cpp` files causes unresolved external symbol linker errors (`LNK2019`).

### The Header-Only Pattern
Place full template definitions directly in header files (`.h`). 

```cpp
// max.h
#ifndef MAX_H
#define MAX_H

template <typename T>
T max(T x, T y)
{
    return (x < y) ? y : x;
}

#endif
```

> **Why this violates NO ODR rules:**
> - Template definitions are strictly **exempt** from single-program ODR constraints across distinct translation units.
> - Instantiated template functions are **implicitly inline**, allowing duplicate identical definitions across multiple compilation units without linker clashes.
