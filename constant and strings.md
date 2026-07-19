# C++ Constants, Literals & String Mechanics

A definitive reference guide for named constants, compile-time optimization, literal suffixes, `std::string` ownership, and `std::string_view` constraints.

---

## 1. Named Constants & Preprocessor Macros

A constant is a value that cannot be altered during the program's execution. C++ separates constants into named (symbolic) constants and literal constants.

* **Constant Variables:** Declared by placing the `const` type qualifier adjacent to the object's data type. The preferred convention is to place `const` before the type (e.g., `const double gravity`).
* **Mandatory Initialization:** Constant variables must be initialized upon definition. Crucially, the initializer of a `const` variable can be a non-constant runtime value.
* **Function Parameters & Returns:** Making value parameters or return-by-value types `const` adds unnecessary clutter, can impede move semantic optimizations, and should be avoided.
* **Macro Evils:** Avoid using `#define` object-like macros for constants. Macros do not obey normal C++ scoping rules, can cause silent naming collisions, and are invisible to the compiler and debugger.

---

## 2. Compile-Time Optimization & Constant Expressions

Optimization is the process of modifying software to make it run faster or use fewer resources. Modern optimizing compilers leverage the **As-if Rule**, allowing them to rearrange or restructure code arbitrarily as long as the *observable behavior* remains completely identical.

### Foundational Optimization Techniques
* **Constant Folding:** The compiler computes expressions containing purely literal operands at compile-time, replacing the statement with the pre-calculated result to save runtime CPU cycles.
* **Constant Propagation:** The compiler replaces variables known to hold constant values directly with their literal values, eliminating redundant memory fetch operations.
* **Dead Code Elimination:** The compiler completely removes code that has no effect on the program's behavior (such as variables that are defined but never used, known as being *optimized out*).

### Constant Expressions (`constexpr`)
* A **constant expression** is a sequence of literals, constant variables, operators, and function calls where *each individual part* is strictly evaluatable at compile-time.
* **The `constexpr` Enforcer:** To guarantee compile-time evaluation, use the `constexpr` keyword. A `constexpr` variable is implicitly `const` and *must* be initialized with a valid constant expression.
* *Best Practice:* Use `constexpr` for any constant variable whose initializer is known at compile-time. Use `const` only for runtime constants.

---

## 3. Literal Types & Suffix Casing

Every literal embedded directly in code possesses an inferred data type. Literal suffixes allow developers to explicitly override these defaults.

| Value Example | Default Type | Target Type Suffix |
| :--- | :--- | :--- |
| `5` | `int` | `u` (unsigned int), `L` (long), `LL` (long long) |
| `3.4` | `double` | `f` / `F` (forces `float` representation) |
| `'a'` | `char` | *None* |
| `"Hello"` | `const char[N]` | `s` (`std::string`), `sv` (`std::string_view`) |

> ⚠️ **Narrowing Trap:** Writing `float f { 4.1 };` causes a compiler warning or error because `4.1` is a `double` literal. Initialize using `4.1f` or shift the variable type to `double`.
> 💡 **Digit Separators (C++14):** Single quotation marks can be used as visual separators inside long numeric literals (e.g., `0b1011'0010` or `2'132'673'462`) without impacting the value.

---

## 4. Operational Numeral Systems

C++ natively interprets constants across four distinct bases, using explicit prefixes to identify the active system:

* **Decimal (Base 10):** Default format; digits `0-9`.
* **Hexadecimal (Base 16):** Prefixed with `0x` or `0X` using digits `0-9` and letters `A-F`. Highly concise for representing raw memory bytes.
* **Binary (Base 2):** Prefixed with `0b` or `0B` using digits `0` and `1` (supported since C++14).
* **Octal (Base 8):** Prefixed with a leading `0` using digits `0-7`. *Avoid octal entirely as it easily leads to accidental conversion bugs*.

### Stream Formats & `std::bitset`
* Modifying stream outputs to print non-decimal formats requires the sticky I/O manipulators `std::hex`, `std::oct`, and `std::dec`.
* Binary stream output requires passing data through a temporary `std::bitset<N>` wrapper (from the `<bitset>` header).

---

## 5. Modern Strings: `std::string` vs. `std::string_view`

C-style strings are inherited from C, possess a hidden trailing null terminator (`'\0'`), exist globally for the entire program execution, but are inherently dangerous and difficult to safely manipulate. Modern C++ introduces safer class alternatives.

### 1. `std::string` (The Data Owner)
Lives inside the `<string>` header and manages its own independent memory via dynamic allocation.

* **Dynamic Overhead:** Initializing or copying a `std::string` triggers an expensive deep copy operation at runtime.
* **Console Streaming:** The standard extraction operator (`std::cin >> string`) halts extraction instantly upon hitting any whitespace. To read a full line of text containing spaces safely, utilize `std::getline(std::cin >> std::ws, stringVar)` instead.
* **The `std::ws` Input Manipulator:** Instructs `std::cin` to flush leading whitespace characters (like left-over newlines `\n` from prior numeric reads) before pulling text data.
* **Length Inquiries:** Call the nested member function `stringVar.length()` to retrieve its unsigned count (ignoring the null-terminator). In C++20, `std::ssize(stringVar)` returns this count as a safe signed integer.
* *Best Practice:* **Never pass `std::string` by value** to a function as it forces an expensive, unnecessary performance copy penalty.

### 2. `std::string_view` (The Read-Only Viewer)
Introduced in C++17 inside the `<string_view>` header. It acts as a lightweight, cheap, completely non-owning proxy view over an already existing string buffer.

* **Zero Copy Design:** Passing or copying a `std::string_view` object is incredibly performant because it only copies pointer tracks and string lengths without copying the text memory buffer.
* **Implicit Conversion Boundaries:** While strings convert to views implicitly, a `std::string_view` **will not implicitly convert** back into a `std::string` to shield developers from accidental, expensive memory allocation copies. Use an explicit `static_cast<std::string>(view)` or explicit initialization syntax instead.
* **Constexpr Compatibility:** Full support for `constexpr` initialization makes `constexpr std::string_view` the absolute best choice for symbolic global string constants.

---

## 6. The Dangling View Hazard

Because `std::string_view` is a non-owning viewer, its lifespan remains absolutely bound to the underlying string object it is observing. If the tracked string object is modified or destroyed while the view is still active, the view instantly becomes a **dangling view**, triggering severe **Undefined Behavior (UB)**.

### Common Pitfalls leading to Undefined Behavior
* **Nested Lifespan Expirations:** Storing a view of a local string inside an inner nested block. Once the block exits, the string is destroyed, leaving the view dangling.
* **Temporary Return Captures:** Assigning a view to the direct return of a function returning a `std::string` by value. The returned string object is a short-lived temporary destroyed at the end of the expression line.
* **The `std::string` Suffix Trap:** 

    std::string_view name { "Alex"s }; // ❌ CRITICAL BUG: "Alex"s builds a temporary std::string object, leaving name instantly dangling!

* **Modification Invalidation:** Altering the underlying observed string variable (e.g., resizing it or changing its text content) invalidates all active view pointers. You must re-validate the view by re-assigning it (`view = stringVar`) before safely printing again.

---

## 7. Architectural Guide: When to Choose What

### Choosing Variable Archetypes
* Use **`std::string`** variables when you must explicitly mutate or append text data over time, or when capturing raw, interactive user console input.
* Use **`std::string_view`** variables for cheap read-only access to existing immutable strings, or as static symbolic header text constants.

### Choosing Function Parameter Patterns
* Prefer **`std::string_view`** parameters for general read-only string inputs.
* Use **`const std::string&`** parameters if interacting with legacy C++14 architectures, or when calling inner subsystems that strictly require null-terminated tracking.

### Choosing Return Specifications
* Safely return a **`std::string` by value** if returning a local string variable or a transient temporary copy.
* Only return a **`std::string_view`** when passing back an unchanged function parameter view, or when dealing exclusively with raw C-style string literals (`"text"`) which are guaranteed to survive for the entire program runtime.
