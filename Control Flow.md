# C++ Control Flow & Program Paths

A complete reference guide for conditional branching, loops, jumps, and program halts in C++.

---

## 1. Introduction to Control Flow

By default, a program is a **straight-line program** executing statements sequentially from the top of `main()` to the bottom. The path the CPU takes is called the **execution path**.

Control flow statements allow changing this sequential path through **branching** (jumping to non-sequential code).

<pre><code>mermaid
graph TD
    A[Control Flow Categories] --> B[Conditionals: if, else, switch]
    A --> C[Loops: while, do-while, for]
    A --> D[Jumps: goto, break, continue]
    A --> E[Halts: std::exit, std::abort]
</code></pre>

---

## 2. Conditional Statements: `if` & `else`

### Implicit Blocks & Scope Pitfalls
If braces `{}` are omitted after an `if` or `else`, the compiler implicitly creates a single-statement block. 
* **The Scope Trap:** Declaring variables inside an implicit block results in a compilation error because the variable is destroyed immediately at the end of that single line.

<pre><code>// ❌ WILL NOT COMPILE
if (true)
    int x{ 5 }; // x is destroyed here!
else
    int x{ 6 };
std::cout << x; // Error: x is out of scope
</code></pre>

### `if-else` vs. `if-if`
* Use **`if-else`** chaining when you only want the code after the *first true condition* to execute.
* Use **`if-if`** sequences when you want *multiple independent conditions* to check and execute.
* **Refactoring Guard Rails:** When every condition block ends with a `return` statement, you can safely drop all `else` keywords to keep the logic flat and clean.

---

## 3. Nested `if` Statements & The Dangling `else`

A **dangling else** occurs when an `else` statement is ambiguous in nested conditions.

> ⚖️ **The Resolution Rule:** In C++, an `else` statement is always paired with the *last unmatched `if` statement* residing within the same block scope.

### The Misleading Layout Example
<pre><code>if (x >= 0) 
    if (x <= 20) 
        std::cout << "Between 0 and 20\n";
else // ⚠️ Attached to the INNER if, NOT the outer one!
    std::cout << "Negative\n"; 
</code></pre>

**Best Practices:**
* Enclose inner `if` statements inside explicit braces `{}` to force proper `else` attachment.
* **Flattening:** Prefer restructuring code using logical operators (`&&`, `||`) or early returns to eliminate deep nesting.

---

## 4. Compile-Time Branching: `if constexpr` (C++17)

When an `if` condition relies on a compile-time constant expression, evaluating it at runtime is wasteful. `if constexpr` evaluates the condition at compile-time and **substitutes** the entire block with only the valid branch inside the compiled binary.

```cpp
#include <iostream>

int main() {
    constexpr double gravity{ 9.8 };

    if constexpr (gravity == 9.8) // Evaluated at compile-time
        std::cout << "Gravity is normal.\n"; // Only this line is compiled
    else
        std::cout << "We are not on Earth.\n"; // Completely optimized away
        
    return 0;
}
