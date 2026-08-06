# 7.1–7.14 — Compound Statements, Scope, Linkage, & Namespaces

A comprehensive reference guide covering compound statements (blocks), local and global scope, storage duration, internal vs. external linkage, namespace mechanics, inline variables, and static duration entities.

---

## 1. Compound Statements (Blocks)

A **compound statement** (also called a **block**) is a group of zero or more statements enclosed within curly braces `{}` that the compiler treats as a single statement.

* **Usage:** Blocks can be used anywhere a single statement is allowed. No trailing semicolon `;` is needed after the closing brace `}`.
* **Nesting:** Blocks can be nested inside other blocks. 

```cpp
int main()
{ // Outer block (nesting level 1)
    int value {};

    { // Inner block (nesting level 2)
        value = 5;
    } // End inner block

    return 0;
} // End outer block
```

> 💡 **Best Practice:** Keep the nesting depth of your functions to **3 or less**. If a function requires deeper nesting levels, refactor the nested blocks into separate sub-functions.

---

## 2. Local Variables: Scope, Lifetime, & Linkage

Local variables are declared inside a function body or block. They possess specific automatic runtime properties:

* **Block Scope:** A local variable is in scope from its point of definition to the end of its defining block.
* **Automatic Storage Duration:** Local variables are instantiated at their point of definition and destroyed when the enclosing block exits.
* **No Linkage:** Identifiers declared locally have no linkage; every declaration inside a distinct block refers to an entirely separate memory entity.

```cpp
int main()
{
    int x { 5 }; // x enters scope and is instantiated here

    {
        int y { 7 }; // y enters scope here
        // Both x and y are visible here
    } // y goes out of scope and is destroyed here

    // y cannot be accessed here
    return 0;
} // x goes out of scope and is destroyed here
```

> 💡 **Best Practice:** Define variables in the most limited existing scope possible. Avoid creating temporary blocks solely to limit a variable's scope—refactor to a separate function instead.

---

## 3. Variable Shadowing (Name Hiding)

**Variable shadowing** occurs when a variable declared within a nested scope shares the exact same identifier as a variable in an outer scope. The inner identifier temporarily "hides" the outer variable.

```cpp
#include <iostream>

int g_value { 5 }; // Global variable

int main()
{
    int apples { 5 }; // Outer block variable

    {
        int apples { 10 }; // Shadows outer block 'apples'
        std::cout << apples << '\n'; // Prints 10
    }

    std::cout << apples << '\n'; // Prints 5

    int g_value { 7 }; // Shadows global variable 'g_value'
    std::cout << g_value << '\n';   // Prints local 7
    std::cout << ::g_value << '\n'; // Global scope resolution operator (::) accesses global 5

    return 0;
}
```

> ⚠️ **Best Practice:** Avoid variable shadowing completely. Prefix global variables with `g_` to prevent accidental shadowing.

---

## 4. User-Defined Namespaces

Namespaces allow developers to group identifiers under explicit scope domains, eliminating naming collisions in large projects.

```cpp
#include <iostream>

namespace Foo
{
    int doSomething(int x, int y) { return x + y; }
}

namespace Goo
{
    int doSomething(int x, int y) { return x - y; }
}

int main()
{
    // Scope Resolution Operator (::) explicitly identifies the target namespace
    std::cout << Foo::doSomething(4, 3) << '\n'; // Outputs 7
    std::cout << Goo::doSomething(4, 3) << '\n'; // Outputs 1
    return 0;
}
```

### Advanced Namespace Mechanics

* **C++17 Nested Syntax:**
  ```cpp
  namespace Foo::Goo {
      void print() {} // Equivalent to nesting namespace Goo inside Foo
  }
  ```
* **Namespace Aliases:** Allows temporary shortening of long, deeply nested namespace paths:
  ```cpp
  namespace Active = Foo::Goo;
  Active::print();
  ```

---

## 5. Internal vs. External Linkage

Linkage determines whether multiple declarations of an identifier in different translation units refer to the exact same entity.

### Internal Linkage (`static`)
Identifiers with internal linkage can only be seen and used within the single translation unit (`.cpp` file) where they are defined.

```cpp
static int g_internal { 5 };  // Internal non-const global
const int g_constInternal { 1 }; // Const globals are internal by default
constexpr int g_constexprInternal { 2 }; // Constexpr globals are internal by default

static void internalFunction() {} // Function restricted to this file
```

### External Linkage (`extern`)
Identifiers with external linkage can be accessed across multiple translation units via forward declarations.

```cpp
// In a.cpp (Definition):
int g_x { 2 }; // Non-const globals are external by default
extern const double g_gravity { 9.8 }; // Explicitly external const

// In main.cpp (Forward Declaration / Access):
extern int g_x; 
extern const double g_gravity;
```

> ⚠️ **Best Practice:** Only use `extern` for global variable forward declarations or explicit `const` global definitions in `.cpp` files. Do not use `extern` on non-const variable definitions.

---

## 6. Sharing Global Constants Across Files

### The Modern C++17 Solution: `inline constexpr`
C++17 introduces **inline variables**, allowing a variable to be defined across multiple translation units without violating the One-Definition Rule (ODR). The linker deduplicates them into a single memory instance.

```cpp
// constants.h
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace Constants
{
    inline constexpr double pi { 3.14159 };
    inline constexpr double avogadro { 6.022e23 };
    inline constexpr double gravity { 9.8 };
}

#endif
```

> 💡 **Best Practice:** If your project supports C++17 or newer, prefer defining `inline constexpr` global variables inside a header file.

---

## 7. Static Local Variables

Applying `static` to a local variable converts its duration from **automatic** to **static** (it is initialized once and survives until the program terminates), while retaining its **block scope**.

```cpp
#include <iostream>

int generateID()
{
    static int s_itemID { 0 }; // Initialized only once on initial function call
    return s_itemID++; // Value persists across subsequent function calls
}

int main()
{
    std::cout << generateID() << '\n'; // Outputs 0
    std::cout << generateID() << '\n'; // Outputs 1
    return 0;
}
```

> ⚠️ **Best Practice:** Avoid non-const static local variables if they alter program flow or prevent a function from being cleanly reused/reset. `const` static local variables are recommended to avoid expensive object re-initialization.

---

## 8. Unnamed & Inline Namespaces

### Unnamed (Anonymous) Namespaces
All declarations inside an unnamed namespace are treated as if they have internal linkage and are automatically imported into the parent scope.

```cpp
namespace // Unnamed namespace
{
    void localHelper() {
        // Can only be accessed within this translation unit
    }
}
```

> 💡 **Best Practice:** Prefer unnamed namespaces over individual `static` declarations when restricting multiple function/type declarations to a single file. Never place unnamed namespaces in header files.

### Inline Namespaces (Versioning)
Used primarily to version library functions. Declarations inside an `inline` namespace are exposed directly to the parent scope.

```cpp
namespace Lib
{
    namespace V1 {
        void compute() { std::cout << "Legacy V1\n"; }
    }

    inline namespace V2 { // Default version
        void compute() { std::cout << "Current V2\n"; }
    }
}

int main()
{
    Lib::compute();     // Executes V2 (inline)
    Lib::V1::compute(); // Explicitly calls legacy V1
}
```

---

## 9. Comprehensive Summary Table

| Variable Type | Scope | Duration | Linkage | Syntax Example |
| :--- | :--- | :--- | :--- | :--- |
| **Local Variable** | Block | Automatic | None | `int x { 1 };` |
| **Static Local Variable** | Block | Static | None | `static int s_x { 1 };` |
| **Internal Global Variable** | Global | Static | Internal | `static int g_x { 1 };` |
| **External Global Variable** | Global | Static | External | `int g_x { 1 };` |
| **Inline Global Constant** | Global | Static | External | `inline constexpr int g_x { 1 };` |
| **Const Global Variable** | Global | Static | Internal | `constexpr int g_x { 1 };` |

---

## 10. Rules for `using` Statements

* **Using-Declaration:** Alias a single unqualified identifier (`using std::cout;`). Safe to use inside `.cpp` source files.
* **Using-Directive:** Imports an entire namespace unqualified (`using namespace std;`). **Avoid doing this** due to high risks of silent naming collisions and ambiguous symbol errors.

> 🛑 **Critical Rule:** Never place `using` statements (declarations or directives) in header files or before `#include` directives.
