# C++ Type Conversions, Aliases & Deduction

A deep-dive reference guide into implicit coercion, numeric promotions, narrowing safety bounds, type casting, aliases, and `auto` inference mechanics.

---

## 1. Implicit Type Conversion (Coercion)

Type conversion is the process of translating data of one type into a temporary object of a different target type. Implicit conversion (coercion) occurs automatically when an expression of one type is supplied to a context expecting another.

* **Bit Interpretation:** Variables are underlying sequences of bits. Identical numbers are represented entirely differently across types (e.g., integer `3` vs. float `3.0`), making raw bit copying across distinct types a severe logical error.
* **Trigger Contexts:** Coercion triggers during variable initialization/assignment, function argument mapping, evaluating non-boolean values in conditional statements, and resolving mixed-type binary operators.
* **Standard Conversions:** The C++ language standard defines 14 conversion rules grouped into 5 distinct categories: Numeric Promotions, Numeric Conversions, Qualification Conversions, Value Transformations, and Pointer Conversions.

---

## 2. Numeric Promotion vs. Numeric Conversion

### 1. Numeric Promotion (Always Value-Preserving)
Numeric promotion is a safe, automatic expansion of narrow numeric types to wider types that match the natural data size processed most efficiently by the CPU. Because every source value can fit exactly in the destination, it is completely value-preserving and never triggers compiler warnings.
* **Floating-Point Promotion:** Unconditionally promotes a `float` into a `double`.
* **Integral Promotion:** Converts small integral types like `bool`, `char`, `signed char`, `unsigned char`, `signed short`, and `unsigned short` directly up to an `int` (or `unsigned int` if an `int` cannot hold the range).
* **Signedness Note:** While value-preserving, integral promotion does not necessarily preserve the explicit signedness of the original narrow type.

### 2. Numeric Conversion (Potentially Unsafe)
Numeric conversions handle all fundamental type translations that do not fit into the exact promotion rules. They are split into three structural safety zones:

| Category | Definition & Behavior | Example |
| :--- | :--- | :--- |
| **Value-Preserving** | Safe conversions where the destination safely represents all values of the source type. | `int` to `long`, or `short` to `double`. |
| **Reinterpretive** | Unsafe conversions where values may wrap modulo, but no absolute precision data is dropped. | Mixing signed/unsigned boundary fields. |
| **Lossy** | High-risk unsafe conversions where fractional components or raw precision digits are discarded. | `double` to `int` (chops fractions), or `double` to `float`. |

---

## 3. Narrowing Conversions & Constexpr Exceptions

A narrowing conversion is a potentially unsafe numeric conversion where the destination type is mathematically incapable of holding all possible values of the source type.

* **Disallowed in Brace Initialization:** Attempting a narrowing conversion inside brace initialization (list-initialization) triggers a strict **compile-time error**, which is why list-initialization is strongly preferred.
* **The Constexpr Loophole:** If the source expression is a `constexpr` value, the compiler evaluates it instantly. If the explicit value can fit perfectly into the destination without truncation, **it is not considered a narrowing conversion**, allowing clean list-initialization without suffixes or casts:

    unsigned int u { 5 }; // ✅ Valid: 5 fits in unsigned int without narrowing
    constexpr int n { -5 };
    unsigned int u2 { n }; // ❌ Compile Error: -5 changes value in unsigned context

* **Floating-Point Oddity:** Conversions from a `constexpr double` to a `float` are uniquely exempted from the narrowing category even if an actual loss of statistical precision occurs, provided the value fits the global range boundaries.

---

## 4. Usual Arithmetic Conversions & Common Type

When binary operators (like `+`, `-`, or relational comparisons) receive operands of mismatched types, they enforce the **Usual Arithmetic Conversions** to convert inputs into a shared **common type**.

### The Priority Rank (Highest to Lowest)
1. `long double`
2. `double`
3. `float`
4. `long long`
5. `long`
6. `int`

### Resolution Steps
* **Step 1:** If any operand is a floating-point type, the lower-ranked operand is converted directly to match that floating-point type.
* **Step 2:** Otherwise, both operands instantly undergo integral promotion. If a sign mismatch persists (signed vs. unsigned), complex rank wrapping applies.
* **The Unsigned Trap:** Mixing signed and unsigned integers inside arithmetic or comparison operators causes signed arguments to unsafely coerce into large wrapped unsigned values:

    std::cout << (-3 < 5u); // ❌ Prints 'false' because -3 wraps to a massive unsigned integer

> 💡 **Utility Hint:** Use `std::common_type_t<T1, T2>` from the `<type_traits>` header to safely resolve the mathematical common type between any two distinct types at compile-time.

---

## 5. Explicit Type Conversion (Casting)

Casts are explicit programmer requests to perform type conversion, converting implicit risk into documented intent.

* **C-Style Casts (Avoid Entirely):** Formatted as `(type)expression` or `type(expression)`. They are dangerous because they can silently swap between static, const, or reinterpretive casting behaviors depending on context, making errors hard to trace and text searches near impossible.
* **`static_cast` (The Gold Standard):** Formatted as `static_cast<TargetType>(expression)`. It provides safe compile-time type validation. If the compiler does not know how to link the types, it throws a compile error, blocking runtime leaks.

    int x { 10 };
    int y { 4 };
    double d { static_cast<double>(x) / y }; // ✅ Forces floating-point division (2.5)

---

## 6. Type Aliases (`using`) vs. Typedefs

Type aliases provide a clean identifier substitute name for an existing data type but **do not create a new, distinct type** (they are not structurally type-safe).

* **Modern Syntax:** Prefer `using NewName = ExistingType;` over the legacy `typedef ExistingType NewName;` syntax. Type aliases separate the name cleanly with an equals sign and easily evaluate complex pointer/function-pointer mappings.
* *Best Practice:* Name custom type aliases starting with a **Capital letter** and avoid using trailing suffixes like `_t` or `_type` to prevent name clashes with system boundaries.

---

## 7. Variable Type Deduction (`auto`)

The `auto` keyword instructs the compiler to automatically deduce a variable's type directly from its initialization expression.

* **Strict Requirement:** `auto` requires a valid initializer expression. It cannot deduce types from uninitialized statements or expressions returning `void`.
* **Qualifier Dropping:** Variable type deduction explicitly **drops top-level `const` and `constexpr` modifiers** from the deduced type. You must re-apply qualifiers manually if preservation is desired:

    const int x { 5 };
    auto y { x };       // y is deduced as a mutable 'int'
    const auto z { x }; // z is explicitly reinforced as a 'const int'

* **String Literal Pitfall:** Deducing directly from a raw string literal results in a legacy C-style character pointer (`const char*`). To correctly infer modern string states, leverage the literal suffixes `s` or `sv` within the `std::literals` namespace:

    using namespace std::literals;
    auto s { "hello"s };  // Deduced as std::string
    auto sv { "world"sv }; // Deduced as std::string_view

* *Best Practice:* Use `auto` when the type details don't matter or readability is improved. Favor explicit types when you must force a conversion or when keeping the exact type visible is crucial to preventing semantic logic errors.

---

## 8. Function Return Type Deduction & Trailing Syntax

### 1. Auto Return Types
Functions can use `auto` to deduce their return type from the internal `return` statement expression. 
* **Constraint 1:** All `return` statements inside an auto-deduced function must resolve to the *exact same type*, otherwise a compiler error occurs.
* **Constraint 2:** Functions utilizing return deduction **must be fully defined before use**. A standard forward declaration is insufficient, meaning they are generally scoped to the local file.
* *Best Practice:* Prefer explicit return types for public interfaces to keep API boundaries obvious, and limit auto return rules to fragile or overly complex types.

### 2. Trailing Return Syntax
Moves the return type to the rear of the prototype using an arrow indicator `->`. This syntax is mandatory for certain advanced architectures like lambdas and when a return type relies on the evaluation of prior function parameters:

    // Explicit trailing return declaration
    auto add(int x, int y) -> int;

    // Advanced contextual evaluation
    auto add(int x, double y) -> std::common_type_t<decltype(x), decltype(y)>;

> ⚠️ **Parameter Warning:** Type deduction cannot be applied to standard function parameters. Writing `void foo(auto x)` does not invoke traditional auto type deduction; instead, it triggers C++20 **Function Templates** under the hood.
