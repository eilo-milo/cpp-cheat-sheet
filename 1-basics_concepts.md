# C++ Basics Cheat Sheet

## Program Structure & Execution
Every C++ program requires a `main()` function. Statements execute sequentially from top to bottom and must end with a semicolon `;`.

```cpp
#include <iostream> // Preprocessor directive for I/O

int main() {
    // Statements go here
    return 0; // 0 indicates successful execution
}
```

---

## Variables & Initialization
An object is a region of memory; a variable is a named object. **Always initialize your variables upon creation** to avoid Undefined Behavior (UB) and garbage values.

| Type | Syntax | Notes |
| :--- | :--- | :--- |
| **Default** | `int a;` | **Avoid.** Leaves variable uninitialized (garbage value). |
| **Copy** | `int b = 5;` | Traditional C-style. May allow narrowing conversions. |
| **Direct** | `int c(6);` | Traditional C++ style. |
| **Direct-List** | `int d { 7 };` | **Best Practice.** Prevents narrowing conversions (e.g., `int x {4.5};` is a compiler error). |
| **Value (Zero)** | `int e {};` | **Best Practice.** Initializes to zero. |

**Variable Best Practices:**
* Define one variable per line.
* Use `[[maybe_unused]]` (C++17) before a variable definition if you define it but intentionally do not use it to suppress compiler warnings.

---

## Console Input & Output (I/O)
Requires `#include <iostream>`. 

| Object / Operator | Syntax | Description |
| :--- | :--- | :--- |
| **Output** | `std::cout << x;` | Prints data to the console. |
| **Input** | `std::cin >> x;` | Reads space/newline separated data from the keyboard into a variable. |
| **Newline** | `\n` | **Best Practice.** Moves cursor to the next line without flushing the buffer. |
| **End Line** | `std::endl` | **Avoid.** Outputs a newline *and* flushes the buffer (which is slow). |

---

## Identifiers & Naming Rules
Identifiers are the names you give to variables, functions, and types. 

**Strict Rules:**
* Must contain only letters, numbers, and underscores.
* Cannot start with a number.
* Cannot be a reserved C++ keyword (e.g., `int`, `return`, `class`).
* Case-sensitive (`value` is different from `Value`).

**Naming Best Practices:**
* Start variables and functions with a lowercase letter.
* Use `camelCase` (e.g., `myVariableName`) or `snake_case` (e.g., `my_variable_name`).
* Do not start names with underscores (often reserved by the compiler/OS).
* Make names descriptive based on their lifespan (e.g., use `totalScore` instead of `s`).
* If modifying existing code, strictly adopt its existing naming conventions over your own.

---

## Formatting & Whitespace
C++ is whitespace-independent. Spaces, tabs, and newlines are generally ignored by the compiler outside of string literals.

**Formatting Best Practices:**
* Use an automatic code formatter in your IDE.
* Keep lines under 80 characters to improve readability.
* If breaking a long line containing an operator (like `<<` or `+`), put the operator at the **beginning** of the new line, not the end of the previous one.

```cpp
std::cout << "This is a very long string that we are splitting "
    << "across multiple lines.\n";
```

---

## Comments
Comments are ignored by the compiler and are for human readability.

* `//` Single-line comment.
* `/* ... */` Multi-line comment (Warning: Cannot be nested inside another multi-line comment).

**Commenting Best Practices:**
* At the library/program/function level: Describe **What** it does.
* Inside the library/program/function: Describe **How** it works.
* At the statement level: Describe **Why** the code is doing something. Avoid explaining *what* the code does unless it is overly complex.

---

## Literals, Operators & Expressions
* **Literal:** A fixed value inserted directly into source code (e.g., `5`, `"Hello"`).
* **Operator:** A symbol denoting an operation (e.g., `+`, `-`, `=`, `<<`).
* **Expression:** A combination of literals, variables, operators, and functions that evaluates to a single value.
* **Side Effect:** An observable effect of an operator or function beyond just producing a return value (e.g., changing a variable's value with `=`, or printing to the screen with `<<`).
* **Expression Statement:** An expression followed by a semicolon (e.g., `x = 5;`), which executes the expression and discards the resulting evaluated value.
