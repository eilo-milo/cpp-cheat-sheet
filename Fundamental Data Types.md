# C++ Fundamental Data Types & Memory Mechanics

A comprehensive reference guide for object sizes, integral types (signed/unsigned), fixed-width types, scientific notation, floating-point behaviors, and explicit casting via `static_cast`.

---

## 1. Bits, Bytes, and Memory Addressing

Variables are symbolic names for a piece of memory used to store information. 

* **The Bit Boundary:** The smallest unit of memory is a binary digit (bit), which can only hold a value of `0` or `1`.
* **Memory Addresses:** Memory is organized into sequential units called memory addresses.
* **The Byte Standard:** In modern computer architectures, individual bits do not get unique addresses because accessing data bit-by-bit is rare. Instead, each memory address holds exactly 1 byte of data, which by modern de-facto standard is comprised of 8 sequential bits.
* **Data Types:** A data type tells the compiler how to interpret a sequence of bits in a meaningful way. The compiler and CPU handle the encoding of values into bits and decoding them back automatically.

---

## 2. Fundamental Data Types Overview

C++ contains three sets of types: fundamental data types, compound data types, and standard library types. The fundamental types (primitive/basic types) are built into the language core and categorized as follows:

| Category | Type Options | Meaning |
| :--- | :--- | :--- |
| **Floating-Point** | `float`, `double`, `long double` | Numbers with a fractional component. |
| **Boolean** | `bool` | True or false conditions. |
| **Character** | `char`, `wchar_t`, `char8_t`, `char16_t`, `char32_t` | A single character of text. |
| **Integer** | `short int`, `int`, `long int`, `long long int` | Positive and negative whole numbers (including 0). |
| **Null Pointer** | `std::nullptr_t` | A null pointer. |
| **Void** | `void` | Represents "no type" / incomplete type. |

---

## 3. The `void` Type (Incomplete Type)

The `void` keyword represents the lack of a type. It is an **incomplete type** (declared but not defined), meaning the compiler does not know how much memory to allocate for it.

* **No Instantiation:** Variables cannot be defined with type `void` (`void value;` triggers a compile error).
* **Function Return Context:** Most commonly used to explicitly indicate that a function does not return any value to its caller. Trying to return a value inside a `void` function triggers a compilation failure.
* **Parameters Context (Deprecated):** In C, `void` inside a parameter list indicates the function accepts no parameters. While this compiles in C++ for backwards compatibility, it is deprecated.
* *Best Practice:* Use an empty parameter list `int getValue()` instead of `int getValue(void)` to indicate that a function has no parameters.

---

## 4. Object Sizes & The `sizeof` Operator

Most objects take up more than 1 byte of memory, occupying consecutive memory addresses based on their data type. An object with $n$ bits can hold $2^n$ unique values.

* **Size Variability:** The C++ standard does not define exact bit sizes for fundamental types. Instead, it guarantees minimum sizes: `char` (8 bits), `short` (16 bits), `int` (16 bits), `long` (32 bits), and `long long` (64 bits).
* **The `sizeof` Operator:** A unary operator that takes a type or a variable name and returns its size in bytes. It cannot be used on incomplete types like `void` and does not include dynamically allocated memory.

    int x{};
    std::cout << sizeof(x);    // Outputs size of x in bytes (typically 4)

* **Performance:** Fundamental types are optimized to match the target CPU's architecture size (natural size). On a 32-bit machine, a 32-bit `int` can be processed quicker than an 8-bit `char` or a 16-bit `short`.

---

## 5. Signed Integers & Overflow Mechanics

Signed integers can store positive whole numbers, negative whole numbers, and zero. By default, integer types in C++ are signed.

* **Preferred Definitions:** Omit the redundant `int` suffix or `signed` prefix when declaring integers.

    short s;      // Prefer over "short int" or "signed short"
    long long ll; // Prefer over "long long int"

* **Ranges:** Standard modern ranges assume **Two's Complement** binary representation (required as of C++20). An $n$-bit signed variable has a range of $-2^{n-1}$ to $(2^{n-1}) - 1$.
* **Signed Integer Overflow:** If an arithmetic operation produces a value outside the representable range of a signed integer, it triggers **Undefined Behavior (UB)**. Data is lost or wrapped unpredictably.
* **Integer Division:** Dividing two integers always produces an integer result. Any fractional component is completely **dropped (truncated)**, not rounded (e.g., `8 / 5` evaluates to `1`, and `-8 / 5` evaluates to `-1`).

---

## 6. Unsigned Integers & The Wrap-Around Hazard

Unsigned integers can only hold non-negative whole numbers (positive numbers and 0). They are declared using the `unsigned` keyword.

* **Unsigned Range Expansion:** Because they do not spend half their bits on negative values, an $n$-bit unsigned variable can hold twice as many positive numbers: $0$ to $(2^n) - 1$.
* **Well-Defined Overflow (Modulo Wrapping):** Unlike signed integers, if an unsigned value goes out of range, the C++ standard defines that it undergoes modulo wrapping. The value is divided by one greater than the largest number of the type, and only the remainder is kept (e.g., inside a 2-byte unsigned short, `65535 + 1` wraps to `0`, and `0 - 1` wraps to `65535`).
* **The Dangerous Signed-Unsigned Mix:** When an arithmetic or relational comparison operation mixes a signed and an unsigned integer, C++ implicitly converts the signed integer to unsigned. This triggers catastrophic bugs:

    signed int s{ -1 };
    unsigned int u{ 1 };
    if (s < u) { /* Does NOT execute! -1 implicitly converts to 4294967295 */ }

* *Best Practice:* **Favor signed numbers over unsigned numbers** for holding quantities, even for values that should be non-negative. Avoid mixing signed and unsigned numbers. Restrict unsigned usage exclusively to bit manipulation, encryption algorithms requiring wrapping, or mandatory standard library array matching.

---

## 7. Fixed-Width Integers (`<cstdint>`) & `std::size_t`

To solve platform-specific size variations where an `int` might be 16-bits on one system and 32-bits on another, C++11 introduced fixed-width integers guaranteed to be the exact same size cross-platform.

* **Standard Widths:** `std::int8_t` / `std::uint8_t` (1 byte), `std::int16_t` / `std::uint16_t` (2 bytes), `std::int32_t` / `std::uint32_t` (4 bytes), and `std::int64_t` / `std::uint64_t` (8 bytes).
* **The 8-bit Character Trap:** On most modern systems, `std::int8_t` and `std::uint8_t` are implemented as aliases for `signed char` and `unsigned char`. Therefore, printing or reading them via `std::cout`/`std::cin` makes them behave like text characters, not integers.
* **Fast and Least Types:** C++ also defines fast types (`std::int_fast32_t`) and least types (`std::int_least32_t`). However, their sizes are implementation-defined and cause inconsistent wrapping bugs across different systems.
* **`std::size_t` (`<cstddef>`):** An alias for an implementation-defined unsigned integral type returned by the `sizeof` operator. It represents the length or size of objects and typically scales to match the system's native address width (32-bit or 64-bit). It imposes a strict upper limit on the maximum size of any single object.
* *Best Practice:* Use `int` when the exact size doesn't matter. Use `std::int#_t` when storing quantities that require a guaranteed range. Use `std::uint#_t` for bit flags or encryption math. **Avoid fast/least types entirely.**

---

## 8. Scientific Notation & Floating-Point Precision

Scientific notation takes the form: $\text{significand} \times 10^{\text{exponent}}$. In C++, the letter `e` or `E` represents the base-10 exponent power multiplier (e.g., `5.9722e24`, `5e-2` = `0.05`).

* **Significant Digits:** The digits inside the significand dictate the absolute precision of the number. Trailing zeroes after a decimal point are considered significant (`87.0` has 3 significant digits).
* **Floating-Point Types:** Conventional formats match the IEEE 754 standard: `float` (typically 4 bytes), `double` (typically 8 bytes), and `long double` (size varies; avoid it).

    double b { 5.0 };  // Floating-point literal (defaults to double)
    float c { 5.0f };  // 'f' suffix forces literal to float type

* **Default Output Truncation:** By default, `std::cout` has a precision of 6 significant digits and will drop trailing fractional zeroes if the decimal part evaluates to 0. To override this, include the `<iomanip>` header and apply the `std::setprecision()` sticky manipulator.
* **The Norm of Rounding Errors:** Floating-point memory is finite, so infinite fractions (like $1/3$) or decimal fractions that cannot be represented exactly in binary (like $0.1 = 0.000110011..._2$) accumulate binary truncation precision faults. 

    double d2{ 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 };
    std::cout << std::setprecision(17) << d2; // ❌ Outputs 0.99999999999999989 instead of 1.0!

* **Special IEEE 754 States:** Supports signed infinity (`inf`/`-inf`), NaN (Not a Number, representing mathematically invalid states like `0.0 / 0.0`), and signed zero (`+0.0`/`-0.0`).
* *Best Practice:* **Favor `double` over `float`** unless space is highly critical. Never assume floating-point numbers are exact, and avoid using them for currency or financial transactions.

---

## 9. Boolean Basics (`bool`)

Boolean variables hold only two possible values: `true` (stored internally as integer `1`) and `false` (stored internally as integer `0`). They are considered integral types.

* **Logical NOT Operator (`!`):** Used to invert or flip a boolean state (e.g., `!true` evaluates to `false`).
* **Printing Booleans:** By default, `std::cout` outputs `1` for true and `0` for false. Output `std::boolalpha` to switch the stream to print literal text words `"true"` or `"false"`. Turn it off using `std::noboolalpha`.
* **Implicit Conversion Rules:** Any context allowing integer-to-boolean conversion treats an integer value of `0` as `false`, and **any other non-zero integer value as `true`**. Uniform brace initialization (`bool b {2};`) strictly disallows these narrowing conversions and generates a compile error.
* **Stream Input Constraints:** By default, `std::cin` only accepts numeric input (`0` or `1`) for booleans. Entering any other characters or strings causes `std::cin` to silently enter failure mode. To allow inputting literal lowercase strings `"true"` or `"false"`, you must stream into the input manipulator `std::cin >> std::boolalpha;`.

---

## 10. Character Types (`char`) & Escape Sequences

The `char` data type holds a single textual symbol (letter, number, punctuation, or whitespace) stored internally as a 1-byte integer mapped via the **ASCII standard** (values 0 to 127).

* **Literal Syntax Bounds:** Character literals are wrapped strictly in **single quotes** (e.g., `'a'`, which maps to ASCII integer code point 97). Double quotes indicate a C-style string literal, which is completely distinct.
* **Whitespace Streaming Elimination:** Standard `std::cin >> ch;` extractions discard and skip all leading whitespace characters. To pull raw, un-skipped whitespace characters (like spaces or tabs) from the stream buffer into a character variable, use `std::cin.get(ch);` instead.
* **Multicharacter Literals Warning:** Avoid packing multiple characters inside single quotes (e.g., `'56'` or accidental forward-slashing errors like `'/n'`). This builds an implementation-defined literal that outputs unexpected massive integer garbage numbers.
* **Unicode Support Extensions:** `char16_t` and `char32_t` (added in C++11) and `char8_t` (added in C++20) provide structural data tracking for UTF-16, UTF-32, and UTF-8 Unicode standards respectively. `wchar_t` is platform-dependent and should be avoided outside of legacy native Windows API interactions.

### Key Escape Sequences Table
Escape sequences start with a backslash `\` inside a literal sequence to inject non-standard device signals or symbols:

| Sequence | Structural Meaning | Sequence | Structural Meaning |
| :--- | :--- | :--- | :--- |
| `\n` | Moves cursor to next line (Newline). | `\t` | Embeds a horizontal tab space. |
| `\'` | Prints a single quote symbol. | `\"` | Prints a double quote symbol. |
| `\\` | Prints a single literal backslash. | `\a` | Triggers a system audio alert beep. |

---

## 11. Type Conversions & Explicit Casting (`static_cast`)

C++ allows implicit type conversions (coercion) to dynamically alter type states without changing the foundational variables supplying the data. Conversion utilizes the source value as input to build a brand new, isolated **temporary object** of the target type.

* **Safe vs. Unsafe Implicit Pathing:** Safe conversions always preserve values natively (e.g., `int` to `double`). Unsafe implicit pathing (such as streaming a fractional `double` to an `int` variable parameter) truncates the decimal component silently, generating compilation warnings or fatal list-initialization syntax errors.
* **`static_cast` Syntax Mechanics:** Explicitly instructs the compiler to force a value translation, asserting full responsibility for data truncation or signs changes while suppressing standard implicit security warnings.

```cpp
int x { 10 };
int y { 4 };

// Explicit conversion forces floating-point division to prevent integer rounding loss
std::cout << static_cast<double>(x) / y; // Returns double value 2.5 safely

char ch { 97 };
std::cout << static_cast<int>(ch);      // Forces printing the underlying ASCII code (97) instead of 'a'
