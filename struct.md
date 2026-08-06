# C++ Program-Defined Types: Structs & Class Templates (Chapter 13)

A detailed, single-block C++ reference guide based strictly on Chapter 13 of LearnCPP, covering **structs, aggregate initialization, member selection, memory alignment, class templates, CTAD, and alias templates** using modern C++ best practices.

---

## 1. Introduction to Structs & Data Members

A **struct** (short for structure) is a program-defined compound data type that lets you bundle multiple related variables (data members) into a single unit.

```cpp
#include <iostream>

// Definition of a program-defined struct type named Employee
struct Employee
{
    int id {};         // Data member (value-initialized to 0)
    int age {};        // Data member
    double wage {};    // Data member
}; // Struct definitions must end with a semicolon

int main()
{
    Employee joe {};   // Instantiates an Employee object named 'joe'
    joe.id = 14;       // Access members using the member selection operator (.)
    joe.age = 32;
    joe.wage = 60000.0;

    std::cout << "Joe's ID: " << joe.id << '\n';
}
```

---

## 2. Default Member Initialization & Struct Initialization

### Default Member Initializers
Data members are not initialized by default. Providing explicit initializers inside the struct definition ensures members are initialized even if an object is instantiated without an initializer list.

```cpp
struct Fraction
{
    int numerator { 0 };   // Default member initializer
    int denominator { 1 }; // Default member initializer
};

int main()
{
    Fraction f1;          // Default initialization: f1.numerator = 0, f1.denominator = 1
    Fraction f2 {};       // Value initialization (Preferred): f2.numerator = 0, f2.denominator = 1
    Fraction f3 { 5, 8 }; // Explicit values override default member initializers
}
```

### Aggregate Initialization Modes
A struct containing only data members (no user-declared constructors, private members, or virtual functions) is a C++ **aggregate**. Aggregates use memberwise aggregate initialization.

```cpp
struct Point3d
{
    double x {};
    double y {};
    double z {};
};

int main()
{
    // 1. Direct-list initialization (Preferred)
    Point3d p1 { 1.0, 2.0, 3.0 };

    // 2. Value initialization (Value-initializes missing or all members)
    Point3d p2 {}; // x = 0.0, y = 0.0, z = 0.0

    // 3. Designated initializers (C++20) - Explicitly map members by name
    Point3d p3 { .x { 1.0 }, .z { 3.0 } }; // p3.y is value-initialized to 0.0
}
```

> **Warning:** Designated initializers **must specify members in order of declaration** in the struct.

---

## 3. Function Parameters & Returning Structs

Passing structs by reference prevents making expensive copies. Functions can also return structs by value to return multiple values.

```cpp
#include <iostream>

struct Vector2D
{
    double x { 0.0 };
    double y { 0.0 };
};

// IN Parameter: Pass by const reference (avoids making copies)
void printVector(const Vector2D& v)
{
    std::cout << '(' << v.x << ", " << v.y << ")\n";
}

// IN-OUT Parameter: Pass by non-const reference (modifies original object)
void scaleVector(Vector2D& v, double factor)
{
    v.x *= factor;
    v.y *= factor;
}

// Returning Structs by Value (Returns a temporary object)
Vector2D createZeroVector()
{
    return {}; // Deduces Vector2D from return type and value-initializes
}

int main()
{
    Vector2D v { 3.0, 4.0 };
    scaleVector(v, 2.0); // Modifies v in-place
    printVector(v);      // Prints (6, 8)

    // Passing a temporary struct directly as an rvalue
    printVector(Vector2D { 1.0, 1.0 });
}
```

---

## 4. Member Selection with Pointers & References

* Use the **dot operator (`.`)** for struct objects and references to structs.
* Use the **arrow operator (`->`)** for pointers to structs (`ptr->member` is equivalent to `(*ptr).member`).

```cpp
#include <iostream>

struct Employee
{
    int id {};
    double wage {};
};

int main()
{
    Employee joe { 1, 50000.0 };

    // Member selection on References
    Employee& ref { joe };
    ref.wage = 55000.0; // Use dot operator (.)

    // Member selection on Pointers
    Employee* ptr { &joe };
    ptr->wage = 60000.0; // Use arrow operator (->) instead of (*ptr).wage

    std::cout << joe.wage << '\n'; // Prints 60000
}
```

---

## 5. Struct Data Ownership

Structs should own the data they contain to avoid dangling references. Data members should be owning types (`std::string`) rather than viewing types (`std::string_view`).

```cpp
#include <iostream>
#include <string>
#include <string_view>

struct Owner
{
    std::string name {}; // std::string is an owner (makes a copy)
};

struct Viewer
{
    std::string_view name {}; // std::string_view is a viewer (does not make a copy)
};

std::string getName()
{
    std::string name { "Alex" };
    return name; // Returns temporary std::string
}

int main()
{
    Owner o { getName() };  // Safe: o.name copies temporary string before it dies
    Viewer v { getName() }; // Danger: v.name views temporary string, leaving it dangling!
}
```

---

## 6. Struct Memory Alignment & Padding

The size of a struct can be larger than the sum of its individual data members because compilers add invisible bytes of **padding** for performance and memory alignment.

```cpp
#include <iostream>

// Unoptimized member order (Padding added after 'a' and 'c')
struct Unpadded
{
    short a {}; // 2 bytes + 2 bytes padding
    int b {};   // 4 bytes
    short c {}; // 2 bytes + 2 bytes padding
}; // sizeof(Unpadded) == 12

// Optimized member order (Minimizes padding)
struct Optimized
{
    int b {};   // 4 bytes
    short a {}; // 2 bytes
    short c {}; // 2 bytes
}; // sizeof(Optimized) == 8

int main()
{
    std::cout << "Unpadded size: " << sizeof(Unpadded) << '\n';   // Prints 12
    std::cout << "Optimized size: " << sizeof(Optimized) << '\n'; // Prints 8
}
```

> **Best Practice:** Declare struct members in **decreasing order of size** to minimize padding.

---

## 7. Overloading Operator `<<` for Structs

Overload `operator<<` to support printing structs directly with `std::cout`.

```cpp
#include <iostream>

struct Point
{
    int x {};
    int y {};
};

// Overload operator<<
std::ostream& operator<<(std::ostream& out, const Point& p)
{
    out << '(' << p.x << ", " << p.y << ')';
    return out; // Return ostream reference to allow operator chaining
}

int main()
{
    Point p { 5, 10 };
    std::cout << "Point: " << p << '\n'; // Outputs: Point: (5, 10)
}
```

---

## 8. Class Templates (`template <typename T>`)

Class templates serve as blueprints for instantiating struct or class types using different member data types.

```cpp
#include <iostream>

// Define a Class Template
template <typename T, typename U>
struct Pair
{
    T first {};
    U second {};
};

// Function Template taking a Class Template argument
template <typename T, typename U>
void printPair(const Pair<T, U>& p)
{
    std::cout << '[' << p.first << ", " << p.second << "]\n";
}

int main()
{
    Pair<int, double> p1 { 1, 2.5 };
    Pair<std::string, int> p2 { "Age", 20 };

    printPair(p1);
    printPair(p2);
}
```

---

## 9. Class Template Argument Deduction (CTAD) & Deduction Guides

Starting in **C++17**, the compiler can automatically deduce template type arguments from aggregate initializers.

```cpp
#include <utility> // For std::pair

template <typename T, typename U>
struct CustomPair
{
    T first {};
    U second {};
};

// C++17 Deduction Guide (Required for custom aggregates in C++17; automatic in C++20)
template <typename T, typename U>
CustomPair(T, U) -> CustomPair<T, U>;

int main()
{
    // Explicit template arguments
    std::pair<int, double> p1 { 1, 2.3 };

    // CTAD (C++17+)
    std::pair p2 { 1, 2.3 };          // Deduces std::pair<int, double>
    CustomPair p3 { "Hello", 100 };   // Deduces CustomPair<const char*, int>
}
```

> **Note:** CTAD cannot be used in non-static member initializations or function parameters.

---

## 10. Alias Templates (`using`)

Alias templates allow creating parameterized type aliases for class templates.

```cpp
#include <iostream>

template <typename T>
struct Point
{
    T x {};
    T y {};
};

// Alias Template (Must be defined in global scope)
template <typename T>
using Coord = Point<T>;

int main()
{
    Coord<int> p1 { 10, 20 };      // Instantiates Point<int>
    Coord<double> p2 { 1.5, 2.5 }; // Instantiates Point<double>

    std::cout << p1.x << ", " << p2.x << '\n';
}
```

---

## Summary Best Practices

1. **Default Initializers:** Always provide default member initializers for all data members inside the struct definition.
2. **Value Initialization:** Prefer `Type obj {};` over `Type obj;` to guarantee value initialization of all members.
3. **Pass by Const Reference:** Pass structs by `const T&` to avoid making expensive copies.
4. **Member Ordering:** Order data members from largest to smallest size to reduce memory padding.
5. **Pointer Member Selection:** Always use `->` instead of `(*ptr).` when accessing members via a pointer.
6. **Data Ownership:** Ensure data members are owning types (`std::string`) rather than viewing types (`std::string_view`).
