# C++ Scopes, Linkages & Namespaces

A comprehensive reference for blocks, custom namespaces, linkage mechanics, static duration, and using-statements.

---

## 1. Compound Statements (Blocks) & Nesting

A compound statement (block) is a group of zero or more statements enclosed in curly braces `{}` and treated by the compiler as a single statement.

* **Nesting Levels:** Blocks can be nested inside other blocks. The nesting level (nesting depth) is the maximum number of nested blocks you can be inside at any point in a function.
* **The Standard Requirement:** The C++ standard dictates that compilers should support at least 256 levels of nesting.
* *Best Practice:* Keep the nesting level of your functions to **3 or less**. If a function requires more depth, it is a prime candidate for refactoring into separate sub-functions.

---

## 2. User-Defined Namespaces

Namespaces provide custom scope regions to group identifiers and prevent naming collisions.

* **Scope Resolution Operator (`::`):** Tells the compiler that the identifier on the right should be looked up within the scope of the operand on the left.
* **Global Lookup (`::identifier`):** Using the scope resolution operator with a blank left-hand side explicitly instructs the compiler to resolve the identifier from the global namespace.
* **Implicit Lookup Cascading:** If an identifier inside a namespace lacks scope resolution, the compiler checks the current namespace first, then outer containing namespaces in sequence, and checks the global namespace last.
* **Forward Declarations:** Forward declarations for namespaced functions must reside within an identical namespace block across files.
* *Best Practice:* Start your program-defined namespace names with a **Capital letter** to remain consistent with the C++ Core Guidelines and avoid collisions with system libraries. **Never add custom code to the `std` namespace.**

---

## 3. Storage Duration & Linkage Mechanics

Every C++ variable possesses traits that dictate its compile-time visibility (**Scope**) and runtime lifespan (**Duration**).

### Linkage Overview
Linkage determines whether multiple declarations of an identifier refer to the same object instance or distinct objects across scopes or files.

### 1. No Linkage (Local Variables)
* Local variables and function parameters have block scope and **automatic storage duration** (instantiated at definition, destroyed at the end of the block).
* Each declaration inside a distinct scope block refers to a totally unique entity.

### 2. Internal Linkage (File Boundary)
An identifier with internal linkage is accessible only within a single translation unit (.cpp file).
* **Variables:** Const and constexpr globals have internal linkage by default. Non-const globals can be forced to have internal linkage using the `static` keyword.
* **Functions:** Functions can be limited to internal linkage by marking the declaration with `static`.
* Duplicate internal definitions across different files do not violate the One Definition Rule (ODR).

### 3. External Linkage (Global Boundary)
An identifier with external linkage is visible to the linker and can be shared across the entire program.
* **Functions:** Functions have external linkage by default.
* **Variables:** Non-const globals have external linkage by default. Const and constexpr variables can be given external linkage using the `extern` keyword.
* **Forward Declarations:** To access an external global variable defined in another file, you must write a forward declaration using the `extern` keyword *without an initializer*.

    // Variable Forward Declaration
    extern int g_x; 

* *Best Practice:* Only use `extern` for variable forward declarations or const global definitions. Do not use `extern` on non-const global definitions with an initializer.

---

## 4. Why Non-Const Global Variables Are Evil

Non-constant global variables have static duration (live until the program ends) and external linkage. They should be avoided entirely in clean architectures.

* **Unpredictable State:** Any function call can covertly modify a global variable's value, rendering the overall program state unpredictable and difficult to trace.
* **De-modularization:** Global variables tie functions tightly to a specific global environment, ruining reusability and isolated unit testing.
* **Static Initialization Order Fiasco:** The initialization order of static objects across *different* translation units is completely ambiguous. If an external global variable in `a.cpp` relies on a global variable in `b.cpp` for its initialization, there is a 50% chance of reading an uninitialized zero-state value.
* *Best Practice:* Prefer local variables. If a global constant is necessary, wrap it inside a user-defined namespace as an `inline constexpr` variable (C++17) in a header file.

---

## 5. Inline Functions & Variables (C++17)

* **Inline Expansion:** A compiler optimization where a function call is replaced directly by the function's internal body instructions to completely eliminate function call overhead.
* **The `inline` Keyword (Modern Definition):** In modern C++, `inline` means **multiple identical definitions are allowed across translation units**. The linker will automatically de-duplicate them into a single canonical target definition without throwing an ODR violation.
* **Inline Variables (C++17):** Allows defining constexpr or const variables directly inside a header file so they are shared efficiently across multiple files without ODR clashes.
* *Best Practice:* Do not use `inline` as a hint to request inline expansion. Only use it when defining functions or variables inside header files.

---

## 6. Static Local Variables

Applying the `static` keyword to a local variable changes its duration from automatic to **static duration**.

* **Lifespan Preservation:** A static local variable is instantiated once (the first time its definition line is hit) and **retains its value across multiple function calls**, surviving past its block scope boundary until the program finishes entirely.
* **Zero Initialization:** If not explicitly initialized, static variables are guaranteed to zero-initialize by default at program startup.
* *Best Practice:* Const static local variables are excellent for avoiding expensive re-initialization costs (e.g., loading values from databases). However, **avoid non-const static local variables** because they obscure function predictability and lock internal state away, making functions non-reusable.

---

## 7. Using-Statements & Namespace Pollution

### 1. Using-Declarations (`using std::cout;`)
Creates an unqualified alias for a single explicit identifier. Safe and acceptable inside source (`.cpp`) files after all include lines.

### 2. Using-Directives (`using namespace std;`)
Imports **all** identifiers from a target namespace into the current scope unqualified. 
* **The Collision Vulnerability:** Avoid `using namespace std;` at the top of your files. It heavily risks naming collisions with library functions or future standard library updates. It also hides the structural source of a function from the code reader.

### Scope and Restrictions
* Using-statements adhere to normal block scoping rules when declared inside a block.
* **Header Rule:** **Never place using-statements in header files** or before `#include` lines. Doing so pollutes the global scope of every file that includes that header, introducing dangerous order-dependent compilation bugs.

---

## 8. Unnamed & Inline Namespaces

### Unnamed (Anonymous) Namespaces
A namespace defined without an identifier name. 

* All content declared within an unnamed namespace automatically receives **internal linkage**, making them completely invisible to the linker outside that file.
* It is a cleaner alternative to marking multiple individual global declarations with the `static` keyword.

### Inline Namespaces (C++17 Versioning)
A namespace declared with the `inline` modifier. 

* Anything declared inside an inline namespace is treated as if it belongs directly to the parent namespace. 
* Primarily used for seamless **API versioning**. By shifting the `inline` keyword from a `V1` block to a `V2` block, newer programs automatically receive the upgraded function defaults, while older applications can still explicitly request `V1::function()` behavior.
