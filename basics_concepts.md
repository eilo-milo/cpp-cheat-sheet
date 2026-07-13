# C++ Core Foundations & Best Practices

A distilled, highly scannable cheat sheet of essential C++ concepts, syntax rules, and modern best practices.

---

## 1. Core Structure & Syntax

* **Statements:** The smallest independent unit of computation. Most statements in C++ must end with a semicolon (`;`).
* **The `main()` Function:** Every executable C++ program must have exactly one `main()` function. Execution happens sequentially from top to bottom. It typically terminates by returning `0` to indicate success.
* **Whitespace Independence:** C++ generally ignores whitespace (spaces, tabs, newlines) outside of literal text. Use this freedom to format code for maximum human readability.

---

## 2. Variables & Initialization

An **object** is a region of storage in memory that holds a value. A **variable** is simply an object that has a name (identifier).

### Initialization Methods

| Type | Syntax Example | Notes |
| :--- | :--- | :--- |
| **Direct-list initialization** | `int x { 5 };` | **Preferred.** Modern style. Strictly disallows dangerous narrowing conversions (e.g., silently dropping a decimal). |
| **Value initialization** | `int x {};` | **Preferred for temporary variables.** Safely zeroes out the memory. Use when the value is about to be overwritten (e.g., by `std::cin`). |
| **Copy initialization** | `int x = 5;` | Inherited from C. Drops fractional parts silently without warning. |
| **Direct initialization** | `int x ( 5 );` | Older style, occasionally used for explicit casting or object type configurations. |

### Variable Best Practices
> ⚠️ **Always Initialize Your Variables:** Uninitialized variables inherit whatever "garbage" data happens to be left over in that memory address. Accessing them triggers **Undefined Behavior (UB)**, which causes unpredictable bugs and crashes.

* **One Definition Per Line:** Define each variable on its own separate line. Avoid the compressed syntax (like `int a, b = 5;`) as it frequently leaves the first variable uninitialized by mistake.
* **Suppressing Unused Warnings:** If a variable is intentionally left unused (e.g., a standard configuration constant), prefix it with the `[[maybe_unused]]` attribute to stop the compiler from throwing warnings or errors.

---

## 3. Input / Output (`iostream`)

* **`std::cout`:** Character output stream. Uses the insertion operator `<<` to push data to the console.
* **`std::cin`:** Character input stream. Uses the extraction operator `>>` to read data from the keyboard into a variable.
* **Buffering:** C++ batches I/O requests into a memory buffer to minimize slow hardware interactions. The buffer flushes (transfers data to the device) periodically automatically.

### I/O Best Practices
* **Prefer `\n` Over `std::endl`:** The `std::endl` manipulator forces a manual buffer flush along with a newline, which can severely slow down performance. Use the literal character `\n` instead.
* **Pre-Initialize Input Objects:** Always cleanly value-initialize a variable before extraction:
  ```cpp
  int userInput{}; 
  std::cin >> userInput;
  ```

---

## 4. Code Quality & Formatting Guidelines

### Purpose-Driven Comments
1. **File/Library Level:** Use to describe **what** the code or library accomplishes.
2. **Function Level:** Use to describe **how** the logic achieves its goal overall.
3. **Statement Level:** Use exclusively to explain **why** a specific choice or workaround was made. Avoid stating *what* a line does (e.g., `x = 0; // set x to 0` is a bad comment).

### Identifier Naming Conventions
* **Variables and Functions:** Start with a lowercase letter. Use `camelCase` or `snake_case` consistently.
* **User-Defined Types:** Classes, structs, and enums should start with an uppercase letter (`MyCustomClass`).
* **Length Proportionality:** The physical length of an identifier should match its scope. Use short names (like `i`) for trivial local blocks or loop scopes, and descriptive long names for items visible globally.
* **Avoid Leading Underscores:** Never start identifiers with `_` as they are frequently reserved for compilers and system libraries.

### Layout & Formatting
* **Line Limitations:** Keep lines under 80 characters. Wrap long expressions manually.
* **Operator Wrapping:** When breaking a lengthy statement across multiple lines, start the next line with the operator rather than leaving it at the end of the previous line:
  ```cpp
  std::cout << partialValue1
      << partialValue2
      << finalValue;
  ```
* **Consistency:** When editing an existing project, always abandon personal preferences to match the existing codebase style perfectly.

---

## 5. Key Terms & Concepts

* **Literal:** A raw, fixed value hardcoded directly into the source file (e.g., `3.14`, `'A'`, `"Text"`).
* **Side Effect:** An observable change made to the program state beyond returning a value (e.g., altering a variable with `=` or modifying the terminal output with `<<`).
* **Expression:** A sequence of values, identifiers, and operators that evaluates down to a single outcome value.
* **Expression Statement:** An expression turned into a valid standalone statement by appending a semicolon (e.g., `x = 5;`). The result of the expression evaluation is safely discarded once the side effects are realized.
