# C++ Program-Defined Types: Structs & Class Templates (Chapter 13)

A practical, detailed C++ reference guide focused on **structs, member access, aggregate initialization, parameter modes, padding, class templates, CTAD, and alias templates**, based on modern C++ best practices.

---

## 1. Introduction to Structs & Data Members

A **struct** (short for structure) is a program-defined compound data type that bundles multiple related variables (data members) into a single type.

```cpp
#include <iostream>
#include <string>

// Definition of a program-defined type 'Employee'
struct Employee
{
    int id {};         // Member variable (value-initialized to 0)
    int age {};        // Member variable
    double wage {};    // Member variable
}; // Must end with a semicolon!

int main()
{
    Employee joe {};   // Instantiates an Employee object (value-initialized)
    joe.id = 14;       // Access members using member selection operator (.)
    joe.age = 32;
    joe.wage = 60000.0;

    std::cout << "Joe's ID: " << joe.id << '\n';
}
```

---

## 2. Default Member Initialization & Struct Initialization

### Default Member Initializers
Provide default values directly inside the struct definition to prevent uninitialized data members.

```cpp
struct Fraction
{
    int numerator { 0 };  // Default member initializer
    int denominator { 1 };
};

int main()
{
    Fraction f1;    // Default initialized: f1.numerator = 0, f1.denominator = 1
    Fraction f2 {}; // Value initialized (Preferred): f2.numerator = 0, f2.denominator = 1
}
```

### Aggregate Initialization Modes
A struct with only data members (no user-declared constructors, private members, or virtual functions) is an **aggregate**.

```cpp
struct Point3d
{
    double x {};
    double y {};
    double z {};
};

int main()
{
    // 1. List initialization (Preferred)
    Point3d p1 { 1.0, 2.0, 3.0 };

    // 2. Value initialization (Sets all uninitialized members to 0/default)
    Point3d p2 {}; // x = 0.0, y = 0.0, z = 0.0

    // 3. Designated Initializers (C++20) - Explicitly map initializers to members
    Point3d p3 { .x { 1.0 }, .z { 3.0 } }; // p3.y is value-initialized to 0.0
}
```

> **Warning:** Designated initializers **must match the order of declaration** in the struct definition.

---

## 3. Function Parameters & Returning Structs

### Parameter Direction Modes with Structs

```cpp
#include <iostream>

struct Vector2D
{
    double x {};
    double y {};
};

// IN Parameter: Pass by const reference (prevents expensive copies)
void printVector(const Vector2D& v)
{
    std::cout << '(' << v.x << ", " << v.y << ")\n";
}

// IN-OUT Parameter: Pass by non-const reference (modifies caller's object)
void scaleVector(Vector2D& v, double factor)
{
    v.x *= factor;
    v.y *= factor;
}

// Returning Structs by Value (Returns unnamed temporary)
Vector2D createZeroVector()
{
    return { 0.0, 0.0 }; // Deduces Vector2D from return type
}

int main()
{
    Vector2D v { 3.0, 4.0 };
    scaleVector(v, 2.0); // Modifies v in-place
    printVector(v);      // Prints (6, 8)

    // Passing temporary struct directly
    printVector(Vector2D { 1.0, 1.0 });
}
```

---

## 4. Member Selection with Pointers & References

* Use operator `.` for **objects** and **references**.
* Use operator `->` (arrow operator) for **pointers**.

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

    // Reference
    Employee& ref { joe };
    ref.wage = 55000.0; // Use dot operator on reference

    // Pointer
    Employee* ptr { &joe };
    ptr->wage = 60000.0; // Equivalent to (*ptr).wage = 60000.0

    std::cout << joe.wage << '\n'; // Prints 60000
}
```

---

## 5. Struct Memory Alignment & Padding

The size of a struct can be larger than the sum of its member sizes due to **compiler padding** added for data structure alignment.

```cpp
#include <iostream>

// Unoptimized ordering (12 bytes on 64-bit due to padding)
struct Unpadded
{
    short a {}; // 2 bytes + 2 bytes padding
    int b {};   // 4 bytes
    short c {}; // 2 bytes + 2 bytes padding
};

// Optimized ordering (8 bytes)
struct Optimized
{
    int b {};   // 4 bytes
    short a {}; // 2 bytes
    short c {}; // 2 bytes
};

int main()
{
    std::cout << "Unpadded size: " << sizeof(Unpadded) << '\n';   // Prints 12
    std::cout << "Optimized size: " << sizeof(Optimized) << '\n'; // Prints 8
}
```

> **Best Practice:** Declare data members in **decreasing order of size** to minimize padding overhead.

---

## 6. Overloading Operator `<<` for Struct Output

Overload `operator<<` to allow formatted output of a struct using `std::cout`.

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
    return out; // Return ostream reference to allow chaining
}

int main()
{
    Point p { 5, 10 };
    std::cout << "Point: " << p << '\n'; // Output: Point: (5, 10)
}
```

---

## 7. Class Templates (`template <typename T>`)

Class templates allow instantiation of aggregate types for any data type without duplicating code.

```cpp
#include <iostream>

// Define a Class Template
template <typename T, typename U>
struct Pair
{
    T first {};
    U second {};
};

// Function Template taking a Class Template
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

## 8. Class Template Argument Deduction (CTAD) & Deduction Guides

Starting in **C++17**, the compiler can deduce template type arguments from initializers.

```cpp
#include <utility> // For std::pair

template <typename T, typename U>
struct CustomPair
{
    T first {};
    U second {};
};

// C++17 Deduction Guide (Required for custom aggregates in C++17, automatic in C++20)
template <typename T, typename U>
CustomPair(T, U) -> CustomPair<T, U>;

int main()
{
    // Explicit template arguments
    std::pair<int, double> p1 { 1, 2.3 };

    // CTAD (C++17+)
    std::pair p2 { 1, 2.3 };            // Deduces std::pair<int, double>
    CustomPair p3 { "Hello", 100 };     // Deduces CustomPair<const char*, int>
}
```

---

## 9. Alias Templates (`using`)

Create type aliases for template classes while keeping template parameters flexible.

```cpp
#include <iostream>

template <typename T>
struct Point
{
    T x {};
    T y {};
};

// Alias Template (Must be placed in global scope)
template <typename T>
using Coord = Point<T>;

int main()
{
    Coord<int> p1 { 10, 20 };       // Expands to Point<int>
    Coord<double> p2 { 1.5, 2.5 };  // Expands to Point<double>

    std::cout << p1.x << ", " << p2.x << '\n';
}
```

---

## Summary Best Practices

1. **Default Values:** Always provide default member initializers for all members inside the struct definition.
2. **Value Initialization:** Prefer `Type obj {};` over `Type obj;` to guarantee full member value initialization.
3. **Pass by Const Reference:** Pass structs by `const T&` to eliminate unnecessary copy overhead.
4. **Member Ordering:** Declare larger data members first to decrease padding.
5. **Pointer Member Selection:** Always use `->` when accessing members via pointers.
