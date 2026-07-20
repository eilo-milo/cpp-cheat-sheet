# 11.7 — Constexpr Functions

In C++, standard function calls are generally not allowed inside constant expressions. This means we cannot use normal function calls anywhere a compile-time constant is strictly required. 

To solve this limitation, C++ provides **constexpr functions**, which are functions that can be safely evaluated at compile-time.

---

## 1. Defining a Constexpr Function

To convert a normal function into a constexpr function, simply prepend the `constexpr` keyword to the function's return type.

### The Problem (Normal Function Failure)
The following code results in a compilation error because a standard function call cannot initialize a `constexpr` variable:

```cpp
#include <iostream>

double calcCircumference(double radius)
{
    constexpr double pi { 3.14159265359 };
    return 2.0 * pi * radius;
}

int main()
{
    // ❌ Compile Error: calcCircumference(3.0) is not a constant expression
    constexpr double circumference { calcCircumference(3.0) }; 

    std::cout << "Our circle has circumference " << circumference << "\n";
    return 0;
}
```

### The Solution (Using `constexpr`)
Adding `constexpr` allows the compiler to evaluate the function call entirely at compile-time:

```cpp
#include <iostream>

constexpr double calcCircumference(double radius) // Now a constexpr function
{
    constexpr double pi { 3.14159265359 };
    return 2.0 * pi * radius;
}

int main()
{
    // ✅ Valid: Evaluates at compile-time and replaces the call with 18.8496
    constexpr double circumference { calcCircumference(3.0) }; 

    std::cout << "Our circle has circumference " << circumference << "\n";
    return 0;
}
```

---

## 2. Compile-Time vs. Runtime Evaluation

A common misconception is that `constexpr` functions *always* run at compile-time. Instead, their execution timing depends heavily on the evaluation context:

* **Compile-Time Evaluation (Guaranteed):** Triggered only when the function is called in a context where a constant expression is explicitly required (e.g., initializing a `constexpr` variable).
* **Runtime Evaluation:** Occurs if the function arguments contain non-constexpr values, or if it is evaluated in a non-constant context (such as an output stream `std::cout` without optimizations enabled).

```cpp
constexpr int greater(int x, int y)
{
    return (x > y ? x : y);
}

int main()
{
    constexpr int g { greater(5, 6) }; // Case 1: Evaluated at compile-time (Required)
    
    std::cout << greater(5, 6) << '\n'; // Case 2: May evaluate at compile-time or runtime

    int x { 5 }; 
    std::cout << greater(x, 6) << '\n'; // Case 3: Evaluated at runtime (Argument is non-const)
}
```

> 💡 **Best Practice:** Always test your `constexpr` functions inside a strict constant context (like initializing a `constexpr` variable). A function might successfully pass compilation for runtime usage but break the build when forced into compile-time evaluation.

---

## 3. Critical Constraints of Constexpr Functions

While highly flexible, `constexpr` functions must adhere to strict rules to guarantee compile-time execution eligibility:

* **Parameters are NOT Constexpr:** The parameters of a `constexpr` function are not implicitly constant expressions, even if a compile-time literal is passed. Therefore, you cannot use function parameters as initializers for internal `constexpr` variables.
* **Non-Const Local Variables:** Within the function scope, you *are* allowed to instantiate non-const local variables and freely modify their states or values during execution.
* **Implicitly Inline:** To evaluate the function, the compiler must see the full definition—not just a forward declaration—before any call invocation. Consequently, `constexpr` functions are implicitly marked `inline`, making them exempt from the One-Definition Rule (ODR) and safe to define in header files.

```cpp
constexpr int foo(int b) 
{
    // ❌ Compile Error: 'b' is a runtime parameter, not a constant expression
    constexpr int b2 { b }; 
    return b;
}
```

---

## 4. Consteval (C++20 Immediate Functions)

C++20 introduces the `consteval` keyword to explicitly force compile-time evaluation. Functions marked `consteval` are known as **immediate functions**. If an immediate function call cannot be resolved at compile-time, a compilation error is immediately generated.

```cpp
#include <iostream>

consteval int greater(int x, int y) // Must evaluate at compile-time
{
    return (x > y ? x : y);
}

int main()
{
    constexpr int g { greater(5, 6) }; // ✅ Valid
    
    int x { 5 };
    std::cout << greater(x, 6) << '\n'; // ❌ Compile Error: Cannot evaluate at compile-time
}
```

---

## 5. Detecting Contexts: `std::is_constant_evaluated` & `if consteval`

C++ allows specialized behavior tuning within a single function depending on whether it is running in a constant context or a runtime context:

* **`std::is_constant_evaluated()` (C++20):** Returns `true` if executed inside a strictly required constant-evaluated context.
* **`if consteval` (C++23):** Provides an updated, cleaner syntax replacement for `std::is_constant_evaluated()`.

```cpp
#include <type_traits>

constexpr int compare(int x, int y)
{
    if (std::is_constant_evaluated()) // Or 'if consteval' in C++23
        return (x > y ? x : y); // Logic for compile-time
    else
        return (x < y ? x : y); // Logic for runtime
}
```

---

## 6. When to Use Constexpr Functions

* **Pure Functions:** Functions that yield the exact same return value given identical arguments and trigger no observable side effects should generally be marked `constexpr`.
* **Future-Proofing:** Unless you have an explicit reason not to, any function capable of being evaluated in a constant expression should be marked `constexpr` from the start to allow compiler optimizations and downstream flexibility.
