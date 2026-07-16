# C++ Functions & Multi-File Architecture

A comprehensive reference for function mechanics, memory scopes, the preprocessor, and header management in C++.

---

## 1. Function Structure & Sequential Calls

A function is a reusable sequence of statements designed to perform a distinct job. Functions split programs into modular, maintainable, and testable chunks.

* **Invocation Mechanics:** When a function call occurs, the CPU suspends the current function (the **caller**), "bookmarks" the execution point, jumps to the called function (the **callee**), runs its body, and returns to the bookmark.
* **Nested Functions Trap:** C++ **does not support nested functions** (defining a function inside another function body is strictly illegal).

---

## 2. Value-Returning Functions

A function is **value-returning** if its return type is anything other than `void`.

* **Return by Value:** Evaluating a return expression creates a temporary object containing a copy of the data, which is safely passed back to the caller.
* **The Critical UB Warning:** Failing to provide a `return` statement in a value-returning function triggers **Undefined Behavior (UB)**. 

### Exception for `main()`
The `main()` function is the *only* value-returning function that implicitly returns `0` if a `return` statement is omitted. However, explicitly returning `0` is best practice for visual consistency.

---

## 3. Void Functions (Non-Value Returning)

* **Behavior:** Used when a function is executed purely for its side effects or console operations.
* **Return Rules:** A `void` function automatically exits at its closing brace. Placing an empty `return;` statement at the very end is redundant and should be omitted.
* **Expression Restriction:** `void` functions cannot be called inside contexts that require an active value (e.g., `std::cout << printHi();` is a compile-time error).

---

## 4. Parameters vs. Arguments

* **Function Parameter:** A local variable declared in the function header, waiting to be initialized by the caller.
* **Function Argument:** The actual value or expression passed into the function call.
* **Pass by Value:** When called, arguments are copied into parameter variables using copy initialization.

### Unreferenced & Unnamed Parameters
If a parameter is structurally required but unused in the function body, **omit its identifier name** to suppress compiler warnings.

    // ✅ Clean and warning-free
    void doSomething(int /*count*/) {
        // Parameter exists for compatibility but has no name
    }

---

## 5. Local Variables: Lifetime & Scope

| Term | Category | Definition | Enforced At |
| :--- | :--- | :--- | :--- |
| **Scope** | Compile-time | Determines where an identifier can be seen and accessed in source text. Local variables have **block scope**. | Compilation |
| **Lifetime** | Runtime | The time window between an object's memory allocation (birth) and deallocation (death). | Execution |

* **Functional Isolation:** Local variables inside different functions can share identical names without any conflict because their block scopes do not overlap.
* *Best Practice:* Define your local variables as close to their first active use as reasonable to keep code highly readable and optimized.

---

## 6. Forward Declarations & The One Definition Rule (ODR)

The C++ compiler compiles code sequentially from top to bottom. If a function call appears before its definition, compilation fails (`identifier not found`).

* **Forward Declaration:** Informs the compiler of an identifier's existence and type mapping before its actual implementation. Written via a **function prototype** (header terminated by a semicolon).
* *Best Practice:* Always keep parameter names inside your forward declarations for documentation clarity: `int add(int x, int y);`.

### The One Definition Rule (ODR) Summary
1. **Within a File:** Each function, variable, or type can have exactly one definition per scope.
2. **Within a Program:** Each function or variable can have exactly one definition across the whole project binary.
3. **Exemptions:** Types, templates, and inline variables can be defined identically across different files.

---

## 7. The Preprocessor & Alternative Compilation Gating

Before compilation, the **preprocessor** transforms source text in-memory, producing a **translation unit**.

### Preprocessor Directives (Start with `#`, end with a newline)
* **`#include`:** Replaces the directive line recursively with the entire literal text of the target file.
* **`#define` Macros:** Establishes a text-substitution rule. 
  * *Best Practice:* Avoid object-like macros with substitution text (e.g., `#define GRAVITY 9.8`). Use `constexpr` variables instead.
* **Conditional Compilation (`#ifdef`, `#ifndef`, `#endif`):** Instructs the preprocessor to include or strip out text blocks depending on whether a macro identifier is defined.
* **Commenting Out Code Blocks:** Use `#if 0` to safely block compilation of a source block, even if it contains non-nestable multi-line comments `/* ... */`. Switch to `#if 1` to instantly re-enable it.

---

## 8. Header Files (`.h`) & Proper Architecture

Header files propagate forward declarations into multiple source files consistently, resolving modular code visibility across a multi-file system.

### Core File Structure
* **Source files (`.cpp`)** contain actual executable functionality and definitions.
* **Header files (`.h`)** contain declarations telling other files how to interface with those definitions.
* *Best Practice:* **Never put standard function or variable definitions in header files.** If a header containing a definition is included in more than one `.cpp` file, the linker will crash with a duplicate definition ODR error.

### Inclusion Syntax Blueprint
* **Angled Brackets (`#include <iostream>`):** Searches compiler/system include directories. Used for library/OS headers.
* **Double Quotes (`#include "add.h"`):** Searches the current directory first, then standard paths. Used for files you wrote.
* *Best Practice:* Source files should always `#include` their paired header file to allow the compiler to cross-check types and flag structural errors early. **Never include `.cpp` files.**

### Header Inclusion Ordering Rule
To catch missing dependency includes automatically, sort your `#include` list alphabetically within these strict tier blocks:
1. The paired header file for this specific code file (`"add.h"`)
2. Other internal headers from your same project (`"mymath.h"`)
3. Third-party library headers (`<boost/tuple.hpp>`)
4. Standard library headers (`<iostream>`)

---

## 9. Header Guards & `#pragma once`

If a header is included multiple times within a single translation unit (e.g., transitively via other headers), it causes duplicate declaration errors.

### Traditional Header Guards
Wrap the entire header content in conditional compile directives using a unique project-specific identifier naming macro:

    #ifndef UNIQUE_PROJECT_PATH_FILENAME_H
    #define UNIQUE_PROJECT_PATH_FILENAME_H

    // All declarations and prototypes go here
    int add(int x, int y);

    #endif

### `#pragma once`
A modern, cleaner alternative supported by virtually all modern compilers. It requests that the compiler handle deduplication automatically:

    #pragma once
    
    // All declarations go here
    int add(int x, int y);

* *Note:* Traditional guards are fully standardized and safer if identical files exist across different directory paths, while `#pragma once` is less error-prone and cleaner to write.
