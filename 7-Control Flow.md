# C++ Control Flow & Program Paths

A complete reference guide for conditional branching, loops, jumps, and program halts in C++.

---

## 1. Introduction to Control Flow

By default, a program is a **straight-line program** executing statements sequentially from the top of `main()` to the bottom. The path the CPU takes is called the **execution path**.

Control flow statements allow changing this sequential path through **branching** (jumping to non-sequential code).

* **Conditionals:** if, else, switch
* **Loops:** while, do-while, for
* **Jumps:** goto, break, continue
* **Halts:** std::exit, std::abort

---

## 2. Conditional Statements: `if` & `else`

### Implicit Blocks & Scope Pitfalls
If braces `{}` are omitted after an `if` or `else`, the compiler implicitly creates a single-statement block. 

* **The Scope Trap:** Declaring variables inside an implicit block results in a compilation error because the variable is destroyed immediately at the end of that single line.

    // ❌ WILL NOT COMPILE
    if (true)
        int x{ 5 }; // x is destroyed here!
    else
        int x{ 6 };
    std::cout << x; // Error: x is out of scope

### `if-else` vs. `if-if`
* Use **`if-else`** chaining when you only want the code after the *first true condition* to execute.
* Use **`if-if`** sequences when you want *multiple independent conditions* to check and execute.
* **Refactoring Guard Rails:** When every condition block ends with a `return` statement, you can safely drop all `else` keywords to keep the logic flat and clean.

---

## 3. Nested `if` Statements & The Dangling `else`

A **dangling else** occurs when an `else` statement is ambiguous in nested conditions.

> ⚖️ **The Resolution Rule:** In C++, an `else` statement is always paired with the *last unmatched `if` statement* residing within the same block scope.

### The Misleading Layout Example
    if (x >= 0) 
        if (x <= 20) 
            std::cout << "Between 0 and 20\n";
    else // ⚠️ Attached to the INNER if, NOT the outer one!
        std::cout << "Negative\n"; 

**Best Practices:**
* Enclose inner `if` statements inside explicit braces `{}` to force proper `else` attachment.
* **Flattening:** Prefer restructuring code using logical operators (`&&`, `||`) or early returns to eliminate deep nesting.

---

## 4. Compile-Time Branching: `if constexpr` (C++17)

When an `if` condition relies on a compile-time constant expression, evaluating it at runtime is wasteful. `if constexpr` evaluates the condition at compile-time and **substitutes** the entire block with only the valid branch inside the compiled binary.

    #include <iostream>

    int main() {
        constexpr double gravity{ 9.8 };

        if constexpr (gravity == 9.8) // Evaluated at compile-time
            std::cout << "Gravity is normal.\n"; // Only this line is compiled
        else
            std::cout << "We are not on Earth.\n"; // Completely optimized away
            
        return 0;
    }

* **Strict Rule:** Code blocks inside `if constexpr` branches must be well-formed syntax-wise, and they require explicit braces.

---

## 5. The `switch` Statement

A `switch` statement evaluates an expression and jumps directly to a matching `case` label. It is heavily optimized via **Jump Tables** under the hood.

### Rules and Limitations
* The switch condition **must evaluate to an integral or enumerated type** (Floating-points and `std::string` are strictly banned).
* All `case` labels must evaluate to unique compile-time constant expressions.
* **Fallthrough Warning:** Execution flows sequentially past `case` boundaries unless blocked by a `break` or `return`. 

### Intentional Fallthrough (`[[fallthrough]]`)
If fallthrough is deliberate (e.g., sharing logic), use the C++17 `[[fallthrough]]` attribute on a null statement to suppress compiler warnings:

    switch (mode) {
        case 1:
            std::cout << "Low Power Mode\n";
            [[fallthrough]]; // Suppresses safe diagnostics
        case 2:
            std::cout << "Running Standard Tasks\n";
            break;
    }

### Formatting & Variable Scope inside `switch`
* **Label Layout:** Conventionally, `case` labels should not be indented from the main `switch` bracket scope.
* **Variable Definition:** Variables declared inside a `case` are scoped to the *entire* switch block. However, **initializing** variables inside a case is strictly illegal if subsequent cases exist (as a jump could bypass the initialization).
* *Best Practice:* If a case statement requires initialized local variables, wrap the case logic in an explicit block `{}`:

    case 1: {
        int localVal{ 42 }; // Safe: scoped only to this block
        std::cout << localVal;
        break;
    }

---

## 6. Loops (الحلقات التكرارية)

### 1. `while` Loops
Evaluates the condition *upfront*. If false initially, the body never executes.
* **The Semicolon Trap:** Placing a semicolon after the condition (`while (cond);`) creates an accidental infinite loop executing an empty null statement.
* **Loop Variables:** Counter variables should **always be signed integers**. Unsigned counters counting downwards will underflow past `0` to a massive number, causing a critical infinite loop.

### 2. `do-while` Loops
Evaluates the condition at the *bottom*, guaranteeing the block executes **at least once**.
* **Scope Constraint:** Variables checked in the `while` condition at the bottom must be declared *outside* the preceding `do` block.
* *Best Practice:* Favor standard `while` loops over `do-while` unless a one-pass guarantee is inherently cleaner.

### 3. `for` Loops
The golden standard for loops controlled by an obvious counter variable.

    for (init-statement; condition; end-expression)
        statement;

* **Execution Blueprint:** 1. `init-statement` (Runs once, defines loop-scoped variables).
  2. `condition` (Checked before *every* iteration).
  3. `Loop body` (Executes if condition is true).
  4. `end-expression` (Executes *after* the body, modifying the counter before looping back to step 2).
* *Best Practice:* Avoid inequality (`!=`) in numeric counter loops. Prefer less-than (`<`) or less-than-or-equal-to (`<=`) so the loop terminates safely even if the counter accidentally skips values.

---

## 7. Jumps: `goto`, `break`, & `continue`

* **`goto` / Statement Labels:** Unconditional jumps that utilize **function scope** (labels are visible throughout the entire function even before declaration). *Banned in standard practice due to spaghetti code generation.*
* **`break`:** Terminates the *innermost enclosing loop or switch block immediately*. Execution picks up right after the loop block.
* **`continue`:** Jumps straight to the bottom of the current loop iteration. 
  * In a `for` loop, the `end-expression` (e.g., `++i`) **still executes** after a `continue`.
  * In a `while` loop, the modifier line is skipped, heavily risking an accidental infinite loop.

---

## 8. Program Halts: Hard Terminations

Halts terminate the application immediately without returning naturally through the remaining call stack logic.

* **`std::exit(status_code)`:** (`<cstdlib>`) Terminates normally. Cleans up static/global storage objects and handles file cache flushing.
* **`std::atexit(cleanup_function)`:** Registers safe functions to fire automatically right before `std::exit()` concludes. Cleanup tasks must take no arguments and return `void`.
* **`std::abort()`:** Triggers abnormal program termination instantly without executing any static or global cleanups.

> ⚠️ **The Critical Local Variable Leak:** **None of the halt functions clean up local stack variables.** Destructors for active local objects in the current scope or higher up the call stack will never fire, making explicit halts unsafe. 

*Best Practice:* Avoid explicit halts entirely; manage critical errors via C++ exceptions or return safely from `main()`.
