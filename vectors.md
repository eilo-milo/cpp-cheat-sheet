# C++ Dynamic Arrays & Containers: `std::vector` (Chapter 16)

A detailed, comprehensive C++ reference guide based strictly on Chapter 16 of *LearnCPP*, covering **containers, `std::vector`, list constructors, element access, indexing sign issues, passing/returning vectors, move semantics, loops, `constexpr` interactions, stack behavior, and `std::vector<bool>`**.

---

## 1. Introduction to Containers & Arrays

A **container** is a data type that provides storage for a collection of unnamed elements. 

### Key Characteristics
* **Unnamed Elements:** The container object itself has an identifier, but its elements do not. This allows containers to scale to thousands or millions of elements without requiring unique variable names.
* **Homogeneous:** In C++, containers hold elements that are all of the same type.
* **Length vs. Size:** The number of elements in a container is its **length** (or count). In C++, "size" often refers to both element count (`size()`) and memory footprint in bytes (`sizeof`), so "length" is preferred when discussing element counts.
* **Array:** A container that stores a sequence of values contiguously in memory (adjacent memory locations with no gaps), allowing fast random access to any element.

### C++ Array Types

| Array Type | Nature | Resizable? | Best Used For |
| :--- | :--- | :--- | :--- |
| **`std::vector`** | Dynamic Array | Yes (at runtime) | General-purpose array needs |
| **`std::array`** | Fixed-size Array | No | Small, fixed-length arrays (supports `constexpr`) |
| **C-style Array** | Built-in Array | No | Legacy/C-compatibility code (avoid in modern C++) |

---

## 2. Fundamental `std::vector` Operations & Construction

`std::vector` is a class template defined in the `<vector>` header. It manages a dynamic array whose elements are allocated contiguously on the heap.

```cpp
#include <iostream>
#include <vector>

int main()
{
    // 1. Value initialization (empty vector, length 0)
    std::vector<int> empty {};

    // 2. Explicit list initialization (4 int elements)
    std::vector<int> primes { 2, 3, 5, 7 };

    // 3. Class Template Argument Deduction (CTAD - C++17)
    std::vector vowels { 'a', 'e', 'i', 'o', 'u' }; // Deduces std::vector<char>

    // 4. Constructing a vector of a specific initial length
    // MUST use direct initialization (parentheses), NOT braces!
    std::vector<int> data(10); // Creates 10 int elements, each value-initialized to 0
}
```

### The List Constructor Precedence Rule
Containers feature a **list constructor** that accepts an initializer list (`std::initializer_list`).

> **Rule:** When constructing an object using non-empty braces `{}`:
> * If the braced list can match a **list constructor**, the list constructor is **always preferred** over other constructors.
> * Therefore, `std::vector<int> v{ 10 };` creates a vector of **length 1** containing the element `10`.
> * Conversely, `std::vector<int> v(10);` calls the explicit single-argument length constructor, creating a vector of **length 10** filled with `0`s.

```cpp
std::vector<int> v1 { 10 }; // Length 1,  element is 10
std::vector<int> v2 ( 10 ); // Length 10, elements are 0
```

> **Best Practice:** When constructing a container with element values, use list initialization `{}`. When constructing a container with a specific initial length, use direct initialization `()`.

---

## 3. Element Access & Bounds Checking

C++ arrays use zero-based indexing ($0$ to $N-1$ for an array of length $N$). Indices represent an offset distance from the first element.

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector prime { 2, 3, 5, 7, 11 };

    // Unchecked Access: Fast, but passing an out-of-bounds index causes Undefined Behavior
    std::cout << prime[0] << '\n'; // 2 (first element)
    std::cout << prime[4] << '\n'; // 11 (last element, index N-1)

    // Checked Access: Performs runtime bounds checking
    // Throws std::out_of_range exception if index is invalid
    std::cout << prime.at(2) << '\n'; // 5
}
```

> **Warning:** For an array of length $N$, index $N$ is **one-past-the-end** and out of bounds. Accessing `array[N]` results in Undefined Behavior.

---

## 4. Vector Length, `size_type`, and the Signed/Unsigned Problem

### The Signed/Unsigned Design Flaw
The C++ standard library container classes use an **unsigned integral type** for lengths and indices. This nested type alias is named `size_type` (which defaults to `std::size_t`).

Because implicit conversion of a runtime signed integer to an unsigned integer is a **narrowing conversion**, using a signed integer as a non-`constexpr` index for `operator[]` will trigger compiler warnings.

### Obtaining Container Length

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector prime { 2, 3, 5, 7, 11 };

    // 1. size() Member Function (Returns unsigned std::vector<int>::size_type / std::size_t)
    std::size_t len1 { prime.size() };

    // 2. std::size() Non-member function (C++17 - Returns unsigned std::size_t)
    std::size_t len2 { std::size(prime) };

    // 3. std::ssize() Non-member function (C++20 - Preferred for signed code!)
    // Returns length as a signed type (usually std::ptrdiff_t)
    auto len3 { std::ssize(prime) }; // Signed length!
    int len4 { static_cast<int>(std::ssize(prime)) };
}
```

### Safe Indexing Strategies

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector prime { 2, 3, 5, 7, 11 };
    int index { 3 }; // Signed int index

    // Strategy 1: Use a constexpr index (implicit conversion from constexpr signed to std::size_t is non-narrowing)
    constexpr int cIndex { 3 };
    std::cout << prime[cIndex] << '\n'; // No warning

    // Strategy 2: Index via data() pointer (returns C-style array pointer, allows signed indexing)
    std::cout << prime.data()[index] << '\n'; // Safe, no warning!

    // Strategy 3: Cast index explicitly
    std::cout << prime[static_cast<std::size_t>(index)] << '\n';
}
```

---

## 5. Passing & Returning `std::vector` (Copy vs. Move Semantics)

### Passing Vectors to Functions
Always pass `std::vector` by **`const` reference** to avoid making expensive deep copies of its dynamically allocated memory.

```cpp
#include <iostream>
#include <vector>

// Explicit Element Type
void printVector(const std::vector<int>& arr)
{
    std::cout << arr[0] << '\n';
}

// Function Template (Accepts vectors of ANY element type)
template <typename T>
void printVectorGeneric(const std::vector<T>& arr)
{
    std::cout << arr[0] << '\n';
}

// Abbreviated Function Template (C++20)
void printVectorAuto(const auto& arr)
{
    std::cout << arr[0] << '\n';
}
```

### Returning Vectors by Value & Move Semantics
While passing `std::vector` by value is expensive, **returning `std::vector` by value is highly efficient** because `std::vector` supports **move semantics**.

* **Copy Semantics:** Deep-copies all data members and heap allocations from a source object to a target object.
* **Move Semantics:** Transfers ownership of internal data storage (pointers) from an rvalue (temporary object about to be destroyed) to a target object. The transfer cost is trivial (just swapping pointers).

```cpp
#include <iostream>
#include <vector>

std::vector<int> generateData()
{
    std::vector data { 1, 2, 3, 4, 5 };
    return data; // Inexpensively MOVED out of the function (or copy-elided)
}

int main()
{
    std::vector myData { generateData() }; // Uses Move Construction! No deep copy occurs.
}
```

> **Best Practice:** Pass `std::vector` by `const std::vector<T>&`. Return `std::vector` **by value** when returning local vector instances or temporaries.

---

## 6. Iterating Over Arrays (Loops & Traversal)

### Index-based `for` Loops

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector scores { 84, 92, 76, 81, 56 };

    // Signed Loop Iteration (C++20 std::ssize)
    for (auto i{ std::ssize(scores) - 1 }; i >= 0; --i)
    {
        std::cout << scores.data()[i] << ' ';
    }
}
```

### Range-based `for` Loops (`for-each`)
Range-based `for` loops iterate through every element in a container safely without indices, eliminating off-by-one errors and sign conversion warnings.

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <ranges> // C++20

int main()
{
    std::vector<std::string> words { "apple", "banana", "cherry" };

    // Read-only access: Use const auto& to prevent expensive copies
    for (const auto& word : words)
    {
        std::cout << word << ' ';
    }

    // Modifying original elements: Use auto&
    for (auto& word : words)
    {
        word += "!";
    }

    // Reverse Traversal (C++20)
    for (const auto& word : std::views::reverse(words))
    {
        std::cout << word << ' ';
    }
}
```

> **Best Practice for Range-Based `for` Loops:**
> * Use `auto` for fundamental / cheap-to-copy types when modifying copies.
> * Use `auto&` when modifying original elements.
> * Use `const auto&` for viewing elements (prevents expensive copies).

---

## 7. Indexing with Enumerations

Unscoped enumerations implicitly convert to integral types, making them excellent symbolic names for array indices.

```cpp
#include <iostream>
#include <vector>
#include <cassert>

namespace Student
{
    enum Names : unsigned int // Unsigned underlying type prevents sign warnings
    {
        kenny,
        kyle,
        stan,
        butters,
        cartman,
        max_students // Count enumerator
    };
}

int main()
{
    std::vector<int> testScores(Student::max_students); // Vector of length 5

    testScores[Student::stan] = 76; // Self-documenting index access

    // Assert that the array matches the enum count
    assert(testScores.size() == Student::max_students);
}
```

---

## 8. Vector Resizing, Capacity, and Stack Behavior

### Length vs. Capacity

```
Vector Storage in Memory:
+-------------------+-------------------+-------------------+-------------------+-------------------+
|  Elem 0 (In Use)  |  Elem 1 (In Use)  |  Elem 2 (In Use)  | Reserved Unused   | Reserved Unused   |
+-------------------+-------------------+-------------------+-------------------+-------------------+
|<------------------ Length = 3 --------------------------->|
|<--------------------------------- Capacity = 5 -------------------------------------------------->|
```

* **Length (`size()`):** How many elements are currently in active use.
* **Capacity (`capacity()`):** How many elements the vector has allocated memory for in storage.

### Reallocation Mechanics
When elements are inserted beyond current capacity, the vector performs **reallocation**:
1. Allocates a new, larger memory block (usually doubling capacity or multiplying by $1.5\times$).
2. Copies/moves existing elements into the new block.
3. Deallocates the old memory block.

Because reallocation is computationally expensive, separating length and capacity reduces allocation overhead.

### Managing Storage: `resize()` vs. `reserve()` vs. `shrink_to_fit()`

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v { 1, 2, 3 }; // Length = 3, Capacity = 3

    // resize(): Changes LENGTH (and capacity if necessary). New elements are value-initialized.
    v.resize(5); // Length = 5 (adds two 0s), Capacity = 5

    // reserve(): Changes CAPACITY ONLY. Length remains unchanged.
    v.reserve(20); // Length = 5, Capacity = 20 (No elements created!)

    // shrink_to_fit(): Requests compiler to reduce capacity to match length.
    v.shrink_to_fit(); // Length = 5, Capacity = 5
}
```

> **Best Practice:** Use `reserve()` before pushing many elements into a vector if you know the approximate number of items in advance.

---

## 9. Stack Behavior (`push_back` and `emplace_back`)

`std::vector` can act as a **Last-In, First-Out (LIFO) Stack**.

```cpp
#include <iostream>
#include <string>
#include <vector>

struct Foo
{
    std::string name {};
    int id {};

    Foo(std::string_view n, int i) : name{ n }, id{ i } {}
};

int main()
{
    std::vector<int> stack {};

    // Push Operations
    stack.push_back(10); // Adds element to end, increments length, reallocates if full
    stack.push_back(20);

    // Peek Top Element
    std::cout << "Top: " << stack.back() << '\n'; // 20

    // Pop Operation
    stack.pop_back(); // Removes last element, decrements length (Capacity unchanged)

    // push_back vs emplace_back
    std::vector<Foo> fooVec {};

    Foo f { "Bar", 1 };
    fooVec.push_back(f); // Pushes existing object

    // Constructing temporary in-place: emplace_back forwards arguments directly to constructor
    // Avoids constructing a temporary object and making an extra copy/move!
    fooVec.emplace_back("Baz", 2); 
}
```

> **Best Practice:** 
> * Prefer `emplace_back()` when constructing a new temporary object directly inside the vector.
> * Prefer `push_back()` when inserting an object that already exists.

---

## 10. The `std::vector<bool>` Caveat

`std::vector<bool>` is a specialized class template optimization that attempts to pack 8 `bool` values into a single byte of memory.

### Why to Avoid `std::vector<bool>`
1. **Not a True C++ Container:** It stores a collection of bit references, not actual `bool` objects contiguous in memory.
2. **Cannot Return `bool&`:** Expression `v[0]` returns a temporary proxy object (`std::vector<bool>::reference`), not a real `bool&`. This breaks generic template functions that expect `T&`.
3. **Inconsistent Performance:** Highly variable across compiler implementations.

```cpp
// AVOID:
// std::vector<bool> flags { true, false, true };

// PREFERRED ALTERNATIVES:
// 1. If length is known at compile time:
// std::bitset<8> flags { 0b1010'1010 };

// 2. If dynamic resizable vector of booleans is required:
// std::vector<char> flags { 1, 0, 1 }; // Fully standard container compatible
```

> **Best Practice:** Avoid `std::vector<bool>`. Use `std::bitset` for compile-time bitflags, or `std::vector<char>` / `std::vector<uint8_t>` for a dynamic container of boolean values.

---

## Summary Best Practices

1. **Initialization:** Use list initialization (`{}`) for explicit element values; use direct initialization (`()`) when specifying initial element capacity/length.
2. **Passing Vectors:** Always pass vectors by `const std::vector<T>&`. Return vectors **by value** (leveraging move semantics).
3. **Looping:** Prefer range-based `for` loops (`const auto&`) over index loops to prevent signed/unsigned warnings and off-by-one errors.
4. **Reserving Memory:** Call `.reserve(N)` before executing a loop with repeated `.push_back()` or `.emplace_back()` calls.
5. **Constructing In-Place:** Use `.emplace_back()` when adding new temporary elements to avoid redundant copies.
6. **Avoid `std::vector<bool>`:** Use `constexpr std::bitset` or `std::vector<char>` instead.
