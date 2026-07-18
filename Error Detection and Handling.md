# C++ Software Testing, Error Handling & Assertions

A comprehensive reference guide for unit testing, input validation via std::cin, error handling strategies, and defensive constraints.

---

## 1. The Software Testing Challenge

Software testing (software validation) is the formal process of determining whether or not the software actually works as expected. 

* **The Combinatorial Explosion:** For all but the simplest programs, explicitly testing every single combination of inputs is computationally impossible. For instance, a simple function comparing two 4-byte integers requires $18,446,744,073,709,551,616$ (~18 quintillion) unique input combinations to test exhaustively.
* **The Smart Testing Reduction:** Instead of brute-forcing billions of runs, programs can be verified with high confidence by identifying and testing unique deterministic execution paths and edge categories instead.
* **Proactive Testing Need:** Just because a program compiles and works for one specific set of inputs does not guarantee it will function correctly under all conditions.

---

## 2. Unit Testing Isolation Strategy

* **Unit Testing:** Testing a small component or distinct piece of your code (such as a function or class) completely in isolation to prove its correctness before integration.
* **The Component Analogy:** Similar to building a car, checking every part individually before final assembly ensures that errors are located immediately inside the small code segment modified since the last compile. This minimizes ripple effects where small initial bugs force massive, sweeping architectural redesigns later.
* *Best Practice:* Write programs in small, well-defined units, compile frequently, and proactively test your code as you write.

### Test Preservation & Automation Frameworks
* **Informal Testing:** Writing temporary code inside `main()` to verify output, then erasing it after passing.
* **Test Preservation:** Retaining test routines within specialized helper verification functions (e.g., `testVowel()`) so they can be re-run safely whenever old code is updated.
* **Automated Testing:** Coding test routines to compare actual values against documented expected answers internally, eliminating the need for manual console verification.
* **Unit Testing Frameworks:** Specialized third-party software systems explicitly designed to streamline writing, managing, and executing complex test suites.
* **Integration Testing:** Retesting previously isolated units together after they have been integrated into the larger application to verify structural compatibility.

---

## 3. Code Coverage Metrics

Code coverage measures how much of the program's source text is actively executed during a test suite.

### 1. Statement Coverage
* Refers to the net percentage of individual statements executed by the test routines.
* While reaching 100% statement coverage is a baseline goal, it is fundamentally **insufficient** to prove structural correctness.

### 2. Branch Coverage
* Refers to the percentage of conditional branches executed, where each path is counted separately.
* An `if` statement contains two branches (the path when the condition evaluates to true, and the path when it evaluates to false, even if an explicit `else` block is omitted).
* *Best Practice:* Always target **100% branch coverage** for your control logic paths.

### 3. Loop Coverage (The 0, 1, 2 Test)
* Dictates that any loop structure must be explicitly tested under three distinct scenarios: executing **0 iterations**, **1 iteration**, and **2 iterations**.
* If a loop processes the 2-iteration edge case correctly, it will reliably function for all iterations greater than 2 ($n > 2$).
* *Best Practice:* Systematically utilize the **0, 1, 2 test** to guarantee loop iteration safety.

---

## 4. Input Category Testing

When testing functions that receive parameters or user input, verify performance across distinct behavioral categories:

* **Integers:** Test negative values, zero, positive values, and check for arithmetic overflow bounds if relevant.
* **Floating-Point Numbers:** Test precision limits using values slightly larger or smaller than expected (e.g., `0.1` and `-0.1`, or `0.7` and `-0.7`).
* **Strings:** Test empty strings (`""`), pure alphanumeric text, strings with mixed whitespace (leading, trailing, or inner), and strings comprised entirely of whitespace characters.
* **Pointers:** Always ensure your function securely handles passing a `nullptr` safely.

---

## 5. Defensive Programming & Error Handling

Defensive programming is the practice of anticipating all possible ways software can be misused by end-users or other developers to prevent malicious or accidental system failure. Assumption errors typically manifest when failing to check successful function execution, malformed external input formatting, or semantically invalid arguments.

### The 4 General Error Handling Strategies
1. **Handle within the Function:** Recover from the error silently inside the local scope where it occurred. This involves looping to retry operations (e.g., waiting for internet re-connectivity or demanding valid user input), or canceling the operation safely.
2. **Pass Back to the Caller:** Return success/failure signals to the parent function. This can be achieved by updating a `void` function to return a `bool` (`true` on success, `false` on failure), or using a unique **sentinel value** (a value with special meaning, such as returning `0.0` from a reciprocal function to signal an input error).
3. **Throw an Exception:** Throw an explicit error object that progressively traverses up the call stack until it is either caught by a matching handling block or forces termination in `main()`.
4. **Halt the Program:** For fatal, non-recoverable errors, terminate immediately via exit utilities.

---

## 6. Stream Extraction Mechanics & `std::cin` Validation

The extraction operator `operator>>` processes stream inputs via a structured text pipeline:
1. Discards all leading whitespace characters (spaces, tabs, newlines) currently waiting in the input buffer.
2. If the buffer is empty, it pauses execution and waits for user entry, subsequently discarding leading whitespace again.
3. Extracts consecutive characters until encountering a newline (`\n`) or a character that is invalid for the target variable type.

### Extraction Results
* **Success:** If characters are successfully extracted, they are converted and assigned to the variable.
* **Failure:** If no characters can be extracted, the target variable is assigned `0` (since C++11), and `std::cin` enters **failure mode**, forcing all future extraction requests to silently fail until cleared.

### Stream Recovery Boilerplate
To safely restore a failed `std::cin` stream back to normal operation, you must execute three tasks sequentially:

    if (!std::cin) // Detects if the previous stream extraction failed or overflowed
    {
        if (std::cin.eof()) // Checks if the stream was permanently closed by a user EOF request
        {
            std::exit(0);   // Shut down the program cleanly
        }

        std::cin.clear();   // 1. Clear stream flags and restore 'normal' operation mode
        ignoreLine();       // 2. Clear out the malformed characters that caused the failure
    }

---

## 7. Stream Parsing Utilities

Wrap these robust stream manipulation blocks inside your source files to manage input tracking safely:

### 1. Clear Current Input Buffer Line
Removes all remaining buffered characters up to and including the trailing newline to prevent extraneous characters from bleeding into subsequent input requests.

    #include <limits>
    #include <iostream>

    void ignoreLine() {
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    }

### 2. Track Unextracted Characters
Peeks into the stream to determine if the user appended extra unextracted data on the same input line (treating extraneous input as a strict validation failure case).

    bool hasUnextractedInput() {
        return !std::cin.eof() && std::cin.peek() != '\n';
    }

---

## 8. Runtime Assertions (`assert`)

An assertion is an expression containing a development assumption that must evaluate to `true` unless a bug exists in the program. 

* **Behavior:** If the expression evaluates to `true`, it does nothing. If it evaluates to `false`, an error diagnostic containing the failed expression, filename, and line number is printed to `std::cerr`, and the program terminates immediately via `std::abort()`.
* **Purpose:** Asserts are development and debugging tools used to catch programming errors early, enforce preconditions (bouncer patterns), and flag unimplemented code blocks. 
* **Header:** Implemented via the `assert` preprocessor macro within the `<cassert>` header file.

### The Descriptive Assert Trick
Since plain flags inside an assert fail with vague messages, append a logical AND string literal to output readable contextual information automatically:

    assert(found && "Car could not be found in the system database");

### Compiling Out Asserts via `NDEBUG`
Assertions introduce a minor performance check cost and should never trigger in a production release. Defining the preprocessor macro `NDEBUG` before including `<cassert>` completely disables the macro, compiling them out of the binary.

> ⚠️ **The Side-Effect Pitfall:** Never place active operational expressions inside an assertion (e.g., `assert(g_mode = 2);`). When `NDEBUG` is defined, the entire expression is stripped out, meaning your logic will **never execute** in release builds.

---

## 9. Compile-Time Assertions (`static_assert`)

A `static_assert` is an assertion checked natively at compile-time rather than during execution. If the condition evaluates to false, it generates a compilation error, completely halting the build process.

* **Syntax:** `static_assert(constexpr_condition, diagnostic_message);`.
* **Keywords:** `static_assert` is a core language keyword, meaning **no header file needs to be included** to use it.
* **Features:** Because it is checked by the compiler, the condition *must* be a valid constant expression. It can reside anywhere (even in the global namespace) and incurs **zero runtime cost**. The diagnostic message parameter is optional since C++17.

### Usage Comparison Summary
* Use `static_assert` whenever a condition can be validated at compile-time (e.g., verifying type memory sizes via `sizeof`).
* Use runtime `assert` to capture program bugs, developer misuse, or impossible conditions during active testing.
* Use explicit **Error Handling** loops, exceptions, or returns to handle normal, expected environment failures safely inside production release builds.
