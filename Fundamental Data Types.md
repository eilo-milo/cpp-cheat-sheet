# C++ Memory, Types & Conversions

A quick reference for memory, fundamental types, and casting in C++.

---

## 1. Memory & Core Types

* **Bit vs. Byte:** A bit is a 0 or 1. A byte is an 8-bit chunk. Memory is addressed byte-by-byte.
* **`sizeof()`:** Returns the size of an object or type in bytes. The return type is `std::size_t` (an unsigned integer type representing size).
* **`void`:** An incomplete type meaning "no type." Used to indicate a function returns nothing. 
  * *Best Practice:* In C++, use empty parentheses `()` for functions with no parameters, not `(void)`.

---

## 2. Integers (Whole Numbers)

Integers drop any fractional parts during division (e.g., `8 / 5` evaluates to `1`).

### Signed Integers (Default)
Can hold positive, negative, and zero values. 
* **Types:** `short`, `int`, `long`, `long long`.
* **Overflow:** Going outside the valid range causes **Undefined Behavior (UB)**.
* *Best Practice:* Prefer `int` when size doesn't strictly matter. Favor signed integers over unsigned ones for general math and quantities.

### Unsigned Integers
Can only hold non-negative values (0 and up). Defined using the `unsigned` keyword.
* **Overflow:** Wraps around (modulo wrap-around). e.g., Decrementing `0` becomes the maximum possible value.
* *Best Practice:* Avoid `unsigned` types for math to prevent wrap-around bugs and unexpected implicit conversion errors when mixed with signed types. Use them strictly for bit manipulation.

### Fixed-Width Integers (`<cstdint>`)
Guarantees the exact size of the integer across all architectures (`std::int16_t`, `std::int32_t`, `std::uint64_t`, etc.).
* *Best Practice:* Use these when you absolutely need a guaranteed range.
* > ⚠️ **The `int8_t` Gotcha:** Compilers treat `std::int8_t` and `std::uint8_t` as `char` types, not numbers. Reading/printing them will process them as ASCII characters. 
* > *Fix:* Cast them using `static_cast<int>(myInt)` to treat them as numbers.

---

## 3. Floating Point (Decimals & Large Numbers)

Used for numbers with fractions or massive scale. Formatted often in scientific notation (`1.2e4`).

| Type | Typical Size | Precision |
| :--- | :--- | :--- |
| `float` | 4 bytes | ~6-9 significant digits |
| `double` | 8 bytes | ~15-18 significant digits |

* **Literals:** Default to `double` (e.g., `5.0`). Append `f` for floats (e.g., `5.0f`).
* **Rounding Errors:** Computers cannot store infinite fractions (like 0.1) perfectly. **Rounding errors are the norm, not the exception.** 
* *Best Practice:* Favor `double` over `float` for better precision. Never use floating-point numbers for exact calculations like currency.

---

## 4. Booleans (`bool`)

Holds only `true` (evaluates to 1) or `false` (evaluates to 0).
* **Implicit Integer Conversion:** `0` becomes `false`. Any non-zero number becomes `true`.
* **I/O Formatting:** `std::cout` prints 1 or 0 by default. Use `std::boolalpha` to make `cout` and `cin` print/read "true" or "false" instead.

---

## 5. Characters (`char`)

Stores a single character, backed by its ASCII integer value (1 byte).
* **Syntax:** Use single quotes for chars (`'A'`), double quotes for strings (`"A"`).
* **Escape Sequences:** Start with a backslash `\`. Common ones: `\n` (newline), `\t` (tab).
* **Input Quirk:** `std::cin >> ch` ignores leading whitespace. If you need to read a literal space character, use `std::cin.get(ch)`.
* *Best Practice:* Avoid multicharacter literals like `'56'` or `'/n'`—they cause unexpected, compiler-dependent behavior.

---

## 6. Type Conversion

* **Implicit Conversion:** The compiler converts types automatically (e.g., passing an `int` to a function expecting a `double`).
  * *Warning:* Converting a `double` to an `int` implicitly drops the decimal and will likely generate a compiler warning for data loss.
* **Explicit Conversion (`static_cast`):** Tells the compiler you intentionally want to convert a type, suppressing warnings.
  * *Syntax:* `static_cast<new_type>(expression)`
  * *Example:* `static_cast<int>(5.5)` securely casts the float to the integer `5`.
