# C++ Function Overloading & Templates

A comprehensive technical reference guide covering function overload differentiation, overload resolution, function deletion via `= delete`, and the mechanics of the C++ template system.

---

## 1. Function Overloading & Differentiation

Function overloading allows multiple functions in the same scope to share the exact same name, provided they can be structurally differentiated by the compiler.

* **The Differentiation Matrix:** The compiler utilizes specific attributes in the function header to differentiate overloads:
    * **Number of parameters:** Overloads can have a different number of parameters.
    * **Type of parameters:** Distinct parameter types (including ellipses) allow for differentiation.
* **Attributes Ignored for Differentiation:**
    * **Return Type:** A function's return type is completely **ignored** for differentiation. Overloading based solely on the return type results in a compile error.
    * **Type Aliases / Typedefs:** These are not distinct types, so aliases do not differentiate an overload from the underlying aliased type.
    * **Value Parameter Constness:** For parameters passed by value, the `const` qualifier is entirely ignored for differentiation.
* **Type Signature:** The unique identifier generated for a function, consisting of its name, number of parameters, parameter types, and function-level qualifiers (excluding the return type).
* **Name Mangling:** The compilation process where the compiler alters function names into unique strings based on parameter counts and types so that the linker has unique handles to connect.

---

## 2. The Overload Resolution Sequence

When an overloaded function is called, the compiler evaluates arguments across a strict sequential matching hierarchy to determine the best single candidate.

### The 6-Step Argument Matching Hierarchy

1. **Exact Match:** The compiler searches for an overload matching the raw argument types exactly. This phase permits *trivial conversions* (such as converting an lvalue to an rvalue, a non-const to a const qualification, or a non-reference to a reference). Trivial matches are exact, meaning overloading `foo(int)` alongside `foo(const int&)` causes an ambiguous match error.
2. **Numeric Promotion:** If no exact match exists, narrow integral and floating-point types are automatically promoted to wider formats (`char`/`bool` to `int`, or `float` to `double`).
3. **Numeric Conversion:** If promotions fail, broader numeric conversions are checked (e.g., converting a `long` or `char` to a `double`, or an `int` to a `bool`).
4. **User-Defined Conversion:** Conversions explicitly defined within program classes or overloaded typecasts are executed.
5. **Ellipsis Match:** The compiler searches for a matching overload that utilizes an ellipsis parameter list.
6. **Compile Error:** If no matching overloads pass any step, compilation fails.

> ⚠️ **The Ambiguous Match:** Occurs if two or more matching functions are found at the *same step* of the resolution sequence. No conversion within a single step takes precedence over another. For example, passing a `long` literal (`5L`) to overloads `foo(int)` and `foo(double)` results in a compile error because both require numeric conversions. 
> 
> * **Disambiguation:** Resolve these errors by explicitly casting the arguments via `static_cast<Type>(arg)` or leveraging explicit literal suffixes (e.g., `0u`).

---

## 3. Deleting Functions (`= delete`)

The `= delete` specifier allows developers to explicitly forbid a function from being callable, halting compilation if a call resolves to it.

* **Behavior Context:** Deleted functions do **not** mean the function does not exist. Instead, they participate in all stages of overload resolution. If the matching algorithm selects a deleted function as the best match, the build immediately breaks.

```cpp
void printInt(int x) { std::cout << x << '\n'; }
void printInt(char) = delete; // Explicitly forbid char inputs

int main() {
    printInt(97);  // ✅ Valid
    printInt('a'); // ❌ Compile Error: Function explicitly deleted!
}
```

---

## 4. Introduction to C++ Templates

The C++ template system automates generic programming, enabling a single template definition to serve as a stencil pattern for a family of related functions or classes across diverse data types.

* **Primary Template:** The core template definition utilizing placeholder types.
* **Type Template Parameters:** Formal placeholders representing types, declared inside angled brackets using the `typename` (preferred) or `class` keywords.
* **Function Template Instantiation:** The translation phase where the compiler clones the primary template, replaces placeholder types with actual types, and generates a concrete function specialization (instance).
* **Best Practice:** Name simple, open template types using a single capital letter starting with T (e.g., `T`, `U`, `V`). Use explicit descriptive names (e.g., `Allocator`) only when specific type requirements must be self-documented.

---

## 5. Template Type Matching & Multi-Type Parameters

* **Strict Type Matching:** By default, template argument deduction enforces exact type matching against type parameters and **does not perform numeric conversions**. For example, a primary template `template <typename T> T max(T x, T y)` will fail to compile if called with mixed arguments like `max(2, 3.5)` because `T` cannot ambiguously represent both an `int` and a `double` simultaneously.

### Resolving Type Mismatch Roadblocks

* **Explicit Argument Definition:** Bypasses deduction entirely, forcing the compiler to instantiate a specific version and implicitly convert any remaining arguments:

```cpp
std::cout << max<double>(2, 3.5); // Forces double instantiation; converts 2 to 2.0
```

* **Multiple Type Parameters:** Redefine the template layout to accept completely independent placeholders:

```cpp
template <typename T, typename U>
auto max(T x, U y) // Uses 'auto' return type deduction to track the common type safely
{
    return (x < y) ? y : x;
}
```

---

## 6. Abbreviated Function Templates (C++20)

C++20 introduces abbreviated function templates as a clean shorthand syntax, automatically mapping `auto` parameters into distinct template parameters behind the scenes.

```cpp
// C++20 Abbreviated Template Syntax
auto max(auto x, auto y) {
    return (x < y) ? y : x;
}

// Exactly equivalent to:
template <typename T, typename U>
auto max(T x, U y) {
    return (x < y) ? y : x;
}
```

* **Best Practice:** Use abbreviated function templates freely when each parameter can vary independently. If you require multiple parameters to enforce the exact same type, stick to traditional explicit template declarations.

---

## 7. Advanced Template Constraints & Risks

* **Semantic Invalidation:** The compiler verifies syntactic validity during template instantiation, but cannot check semantic sense. For instance, an `addOne(T x)` template implementing `return x + 1;` will successfully compile on a C-style string literal via pointer arithmetic, returning a confusing clipped string substring pointer rather than a logical error.
* **Modifiable Static Local Variables:** If a function template defines a static local variable, each unique type instantiation receives its own independent static variable instance. Modifications to a static variable inside an `int` specialization have zero effect on the static variable inside a `double` specialization.

---

## 8. Non-Type Template Parameters

A non-type template parameter is a parameter with a fixed type that acts as a placeholder for a compile-time `constexpr` value passed as a template argument.

* **Allowed Types:** Includes integral types (conventionally named `N`), enumerations, `std::nullptr_t`, floating-point types (since C++20), and raw pointers/references.
* **Core Value:** Because standard function parameters cannot be marked `constexpr`, non-type template parameters are essential when a function requires a constant expression value for internal operations like `static_assert` or array sizing constraints.

```cpp
template <int N> // 'N' is an integer non-type template parameter
void printTemplateValue() {
    std::cout << N << '\n';
}

int main() {
    printTemplateValue<5>(); // Instantiates a function permanently bound to literal 5
}
```

* **Auto Inference (C++17):** Non-type templates can use `template <auto N>` to automatically infer the underlying value category at compile-time without explicit type binding.

---

## 9. Organizing Templates across Multi-File Projects

Unlike standard non-template functions, templates cannot easily separate forward declarations in header files from definitions in separate source `.cpp` files.

* **The Linker Error Threat:** If a function template definition resides in `add.cpp` but is called as `addOne(1)` inside `main.cpp`, the compiler cannot see the blueprint code within `main.cpp` to perform the necessary implicit instantiation. It generates a forward declaration token, but because `add.cpp` has no matching inner call to trigger instantiation locally, the compiler generates no binary function code, causing a fatal linker error (`unresolved external symbol`).
* **The Header File Solution:** Place the entire function template definition inside a header file (`.h`) and `#include` it wherever required. This guarantees the preprocessor copies the complete blueprint directly into the translation unit, enabling immediate compile-time type instantiation.
* **ODR Exemption Safety:** Function templates are explicitly exempt from the single-definition limits of the One-Definition Rule (ODR). Implicitly instantiated template functions are implicitly marked `inline`, allowing them to safely coexist across multiple code files as long as their definitions are completely identical.
* **Best Practice:** Always completely define multi-file shared templates inside header files and include them as needed.
