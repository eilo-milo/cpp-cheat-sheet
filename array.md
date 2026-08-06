# C++ Fixed-Size & Multidimensional Arrays: `std::array` & C-Style Arrays (Chapter 17)

A comprehensive reference guide based on **Chapter 17 of *LearnCPP***, covering `std::array`, C-style arrays, pointer decay, pointer arithmetic, `std::reference_wrapper`, multidimensional arrays, and `std::mdspan`.

---

## 1. Fixed-Size Arrays vs. Dynamic Arrays

Arrays fall into two categories based on how their memory and length are managed:

```
                  +-----------------------------------+
                  |            Array Types            |
                  +-----------------------------------+
                                    |
            +-----------------------+-----------------------+
            |                                               |
  [ Dynamic Arrays ]                              [ Fixed-Size Arrays ]
  - Length set/changed at runtime                 - Length fixed at compile-time
  - Allocated on the Heap                         - Allocated on the Stack / Data Segment
  - Example: std::vector                          - Examples: std::array, C-Style Arrays
```

### Why Use `std::array` Over `std::vector`?
1. **Performance:** `std::array` incurs zero dynamic memory allocation overhead. Elements reside directly on the stack or inline within the owning object.
2. **`constexpr` Support:** `std::vector` supports `constexpr` only in very limited contexts. `std::array` is fully `constexpr`-compatible, enabling compile-time lookup tables, precomputed calculations, and static assertions.

> **Best Practice:** Use `std::array` for `constexpr` arrays or fixed-length stack data. Use `std::vector` for non-`constexpr` arrays whose length is determined at runtime.

---

## 2. Defining & Initializing `std::array`

`std::array` is a fixed-size container defined in `<array>`. It requires two template arguments:
1. **Element Type:** `T`
2. **Array Length:** `N` (a non-type template parameter of type `std::size_t`)

```cpp
#include <array>
#include <iostream>

int main()
{
    // The length MUST be a constant expression (compile-time evaluation)
    constexpr int len { 5 };
    std::array<int, len> a1 {}; // Value-initialized: All elements are 0 (PREFERRED)

    std::array<int, 5> a2;      // Default-initialized: Fundamental types left UNINITIALIZED (Garbage memory)

    // Aggregate Initialization
    std::array<int, 5> prime { 2, 3, 5, 7, 11 }; // Fully initialized
    std::array<int, 5> partial { 1, 2 };          // partial[0]=1, partial[1]=2; remaining elements value-initialized to 0
}
```

### Zero-Length `std::array`
`std::array<T, 0>` is a valid special-case class with no data members (`arr.empty() == true`). Accessing its elements via `operator[]` or `front()` results in **undefined behavior**.

### CTAD & `std::to_array`

```cpp
#include <array>

int main()
{
    // CTAD (Class Template Argument Deduction - C++17)
    constexpr std::array a1 { 1, 2, 3, 4, 5 }; // Deduces std::array<int, 5>
    constexpr std::array a2 { 1.1, 2.2 };      // Deduces std::array<double, 2>

    // Specifying element type while deducing size via std::to_array (C++20)
    constexpr auto shortArray { std::to_array<short>({ 9, 7, 5, 3, 1 }) }; // std::array<short, 5>
}
```

> **Best Practice:** Prefer CTAD to deduce both type and length for `std::array`. When an explicit non-deducible type is required (e.g., `short`), use `std::to_array` (C++20).

---

## 3. Length, Indexing, and Bounds Checking

`std::array` uses `std::size_t` for its non-type template parameter and defines a nested alias `size_type = std::size_t`.

```cpp
#include <array>
#include <iostream>

int main()
{
    constexpr std::array prime { 2, 3, 5, 7, 11 };

    // 1. Getting the length (all return constexpr std::size_t or std::ptrdiff_t)
    constexpr std::size_t l1 { prime.size() };  // Member function
    constexpr std::size_t l2 { std::size(prime) }; // C++17 Non-member function
    constexpr auto l3        { std::ssize(prime) };// C++20 Signed length (std::ptrdiff_t)

    // 2. Element Access
    std::cout << prime[2];        // Fast, unchecked access (Index 2 -> 5)
    std::cout << prime.at(2);     // Runtime bounds checking (throws std::out_of_range)

    // 3. Compile-Time Bounds Checking via std::get<N>()
    std::cout << std::get<3>(prime); // Valid: Returns 7 at compile-time
    // std::cout << std::get<9>(prime); // COMPILE ERROR: Index 9 out of bounds (static_assert failure)
}
```

---

## 4. Passing & Returning `std::array`

Because `std::array` stores its data inline without indirection, **passing it by value creates a full deep copy**. Always pass `std::array` by `const` reference unless a copy is explicitly required.

### Accepting Arrays of Any Length via Templates

```cpp
#include <array>
#include <iostream>

// Template parameterizes both Element Type (T) and Array Length (N)
template <typename T, std::size_t N>
void printArray(const std::array<T, N>& arr)
{
    static_assert(N > 0, "Array must not be empty!");
    for (const auto& elem : arr)
        std::cout << elem << ' ';
    std::cout << '\n';
}

// C++20 Auto Non-Type Template Parameter syntax
template <typename T, auto N>
void printArrayCpp20(const std::array<T, N>& arr)
{
    std::cout << "Length is: " << N << '\n';
}

int main()
{
    std::array a1 { 1, 2, 3 };
    std::array a2 { 1.1, 2.2, 3.3, 4.4 };

    printArray(a1); // Instantiates printArray<int, 3>
    printArray(a2); // Instantiates printArray<double, 4>
}
```

### Returning a `std::array`

```cpp
#include <array>

// 1. Return by value (Efficient for small, cheap-to-copy element types)
template <typename T, std::size_t N>
std::array<T, N> createArray(T defaultValue)
{
    std::array<T, N> arr{};
    arr.fill(defaultValue);
    return arr; // Copy elision / NRVO will eliminate copies in release builds
}

// 2. Out-parameter (For large arrays or expensive-to-copy element types)
template <typename T, std::size_t N>
void fillArray(std::array<T, N>& outArr, T value)
{
    outArr.fill(value);
}
```

---

## 5. `std::array` of Structs & Brace Elision

Initializing a `std::array` of aggregate structs can require explicit brace handling due to how C++ models member arrays internally.

```cpp
#include <array>
#include <iostream>

struct House
{
    int number{};
    int stories{};
    int roomsPerStory{};
};

int main()
{
    // Option A: Explicit element construction (Single Outer Braces)
    constexpr std::array houses1 {
        House{ 13, 1, 7 },
        House{ 14, 2, 5 }
    };

    // Option B: Double Braces for Inner C-Style Array Member (Brace Elision explicit override)
    // Necessary when passing raw brace lists without the class name
    constexpr std::array<House, 2> houses2 {{
        { 13, 1, 7 },
        { 14, 2, 5 }
    }};

    std::cout << houses1[0].number << '\n';
}
```

---

## 6. Arrays of References via `std::reference_wrapper`

In C++, **references are not objects**, meaning you cannot create a raw array of references (`std::array<int&, 3>` is invalid). To store rebindable or non-owning references in a container, use `std::reference_wrapper` from `<functional>`.

```cpp
#include <array>
#include <functional> // for std::reference_wrapper, std::ref, std::cref
#include <iostream>

int main()
{
    int x { 1 }, y { 2 }, z { 3 };

    // Array of reference wrappers (C++17 CTAD)
    std::array<std::reference_wrapper<int>, 3> arr { x, y, z };

    // Modifying the underlying object requires .get()
    arr[1].get() = 20; // Modifies 'y' directly

    std::cout << y << '\n'; // Prints 20

    // Using std::ref and std::cref helper functions
    auto arr2 { std::array{ std::ref(x), std::ref(y) } };
}
```

---

## 7. Enum-Indexed Arrays & String Lookup Tables

Using unscoped enumerations or enum classes as indices creates self-documenting, type-safe lookup tables.

```cpp
#include <array>
#include <iostream>
#include <string_view>

namespace Color
{
    enum Type
    {
        black,
        red,
        blue,
        max_colors // Count enumerator
    };

    using namespace std::string_view_literals;
    
    // Map enum values directly to string names
    constexpr std::array names { "black"sv, "red"sv, "blue"sv };

    // Ensure array length stays in sync with enum count
    static_assert(std::size(names) == max_colors, "Color names out of sync!");
}

constexpr std::string_view getColorName(Color::Type color)
{
    // Cast enum to size_t to safely index
    return Color::names[static_cast<std::size_t>(color)];
}

int main()
{
    std::cout << getColorName(Color::red) << '\n'; // Prints "red"
}
```

---

## 8. C-Style Arrays & Array Decay

C-style arrays (`T name[N]`) are inherited from C. They lack safety, do not know their own length after decay, and should generally be avoided in modern C++.

### Array Decay Mechanics
Whenever a C-style array is evaluated in an expression (including being passed to a function), it implicitly **decays into a pointer** to its first element (`&arr[0]`), losing its compile-time size information.

```
Visualizing Array Decay:

  c-array: [ 9 ] [ 7 ] [ 5 ] [ 3 ] [ 1 ]   Type: int[5] (Knows size = 5)
             ^
             |
  decayed: [ * ]                           Type: int*   (Lost size information!)
```

```cpp
#include <iostream>

// Parameter 'arr[]' or 'arr[1000]' is automatically rewritten by compiler to 'const int*'
void printDecayedArray(const int arr[]) // Same as: const int* arr
{
    // sizeof(arr) returns the size of a POINTER (e.g., 8 bytes on 64-bit), NOT the array!
    std::cout << "Sizeof in function: " << sizeof(arr) << '\n'; 
}

int main()
{
    int arr[5]{ 9, 7, 5, 3, 1 };

    std::cout << "Sizeof in main: " << sizeof(arr) << '\n'; // Prints 20 (5 ints * 4 bytes)
    printDecayedArray(arr); // Array decays to int* when passed
}
```

> **Exceptions to Decay:** A C-style array does **not** decay when used with `sizeof()`, `typeid()`, taking its address (`&arr`), or when passed by reference (`const int(&arr)[5]`).

---

## 9. Pointer Arithmetic & Subscripting Math

Array subscripting (`arr[i]`) is implemented in terms of **pointer arithmetic**.

```cpp
ptr[n]  <===>  * (ptr + n)
```

```cpp
#include <iostream>

int main()
{
    const int arr[]{ 9, 7, 5, 3, 1 };
    const int* ptr{ arr }; // Decays to pointer to arr[0]

    // Pointer Arithmetic
    std::cout << *ptr << '\n';       // Prints 9 (arr[0])
    std::cout << *(ptr + 2) << '\n'; // Moves forward 2 * sizeof(int) bytes -> Prints 5 (arr[2])
    std::cout << ptr[2] << '\n';     // Equivalent shorthand for *(ptr + 2) -> Prints 5

    // Relative Indexing with Negative Subscripts
    const int* midPtr { &arr[3] };   // Points to element 3 (value 3)
    std::cout << midPtr[-1] << '\n'; // Moves BACKWARD 1 element -> Prints 5 (arr[2])
}
```

### Traversing via Pointer Boundaries (`begin` & `end`)

```cpp
#include <iostream>

int main()
{
    constexpr int arr[]{ 9, 7, 5, 3, 1 };

    const int* begin{ arr };                      // Points to first element
    const int* end  { arr + std::size(arr) };     // Points to ONE-PAST-THE-END

    // Standard pointer traversal loop (basis of iterators)
    for (; begin != end; ++begin)
    {
        std::cout << *begin << ' '; // Dereference pointer to access value
    }
}
```

---

## 10. C-Style Strings (`char[]`)

A **C-style string** is a C-style array of `char` (or `const char`) ending with a **null-terminator character** (`'\0'`, ASCII 0).

```cpp
#include <iostream>
#include <cstring> // for std::strlen

int main()
{
    // Array length is 6 (5 text characters + 1 null-terminator '\0')
    const char str[]{ "Hello" };

    // std::strlen counts characters UP TO (excluding) the null-terminator
    std::cout << "String length: " << std::strlen(str) << '\n'; // Prints 5
    std::cout << "Array size:   " << sizeof(str) << '\n';     // Prints 6

    // Safe input reading in C++20
    char buffer[255]{};
    std::cin.getline(buffer, std::size(buffer)); // Prevents buffer overflow
}
```

> **Best Practice:** Avoid non-`const` C-style string arrays. Use `std::string` for mutable text, and `constexpr std::string_view` for string symbolic constants.

---

## 11. Multidimensional Arrays

Multidimensional arrays model grids, matrices, or spatial volumes.

### C-Style Multidimensional Arrays (Row-Major Order)
C++ stores multidimensional arrays in **row-major order** (elements of the same row are adjacent in memory).

```cpp
#include <iostream>

int main()
{
    // 3 rows, 4 columns: arr[Row][Column]
    int grid[3][4] {
        { 1,  2,  3,  4 },  // Row 0
        { 5,  6,  7,  8 },  // Row 1
        { 9, 10, 11, 12 }   // Row 2
    };

    // Traversal (Outer loop = rows, Inner loop = columns)
    for (std::size_t row{ 0 }; row < 3; ++row)
    {
        for (std::size_t col{ 0 }; col < 4; ++col)
        {
            std::cout << grid[row][col] << '\t';
        }
        std::cout << '\n';
    }
}
```

### Multidimensional `std::array` via Alias Templates

```cpp
#include <array>
#include <iostream>

// Alias Template for a clean 2D std::array declaration syntax
template <typename T, std::size_t Row, std::size_t Col>
using Array2d = std::array<std::array<T, Col>, Row>;

int main()
{
    // Create a 3-row, 4-column 2D array
    Array2d<int, 3, 4> grid {{
        { 1, 2, 3, 4 },
        { 5, 6, 7, 8 },
        { 9, 10, 11, 12 }
    }};

    std::cout << grid[1][2] << '\n'; // Access Row 1, Column 2 -> Prints 7
}
```

---

## 12. Flattening 2D Arrays & `std::mdspan` (C++23)

Storing multidimensional data in nested arrays can cause memory fragmentation or verbose syntax. **Flattening** maps a 2D coordinate $(R, C)$ into a single 1D array index:

$$\text{Index} = (\text{Row} \times \text{Total Columns}) + \text{Column}$$

```
2D Grid View (3 x 4):           Flat 1D Array (Length = 12):
Row 0: [ 1][ 2][ 3][ 4] ---->  [ 1][ 2][ 3][ 4][ 5][ 6][ 7][ 8][ 9][10][11][12]
Row 1: [ 5][ 6][ 7][ 8] -------^
Row 2: [ 9][10][11][12] ----------------------^
```

### C++23 `std::mdspan`
`std::mdspan` is a non-owning, non-const, multidimensional view over a 1D contiguous sequence of elements.

```cpp
#include <array>
#include <iostream>
#include <mdspan> // C++23

template <typename T, std::size_t Row, std::size_t Col>
using FlatArray2d = std::array<T, * Col Row>;

int main()
{
    // Flat 1D storage
    FlatArray2d<int, 3, 4> flatData {
        1,  2,  3,  4,
        5,  6,  7,  8,
        9, 10, 11, 12
    };

    // Construct a 2D view (3 rows, 4 columns) over the 1D pointer data
    std::mdspan view { flatData.data(), 3, 4 };

    // Query dimensions
    std::cout << "Rows: " << view.extents().extent(0) << '\n';
    std::cout << "Cols: " << view.extents().extent(1) << '\n';

    // Access elements using C++23 multidimensional subscript operator [row, col]
    std::cout << "Element at (1, 2): " << view[1, 2] << '\n'; // Prints 7

    // Modify underlying data through the view
    view[1, 2] = 77;
    std::cout << "Modified flatData[6]: " << flatData[6] << '\n'; // Prints 77
}
```

---

## Summary Best Practices

1. **`std::array` Selection:** Prefer `std::array` over `std::vector` whenever the array length is known at compile time and `constexpr` evaluation or zero dynamic allocation is desired.
2. **Class Template Argument Deduction:** Use CTAD (`std::array a { 1, 2, 3 }`) or `std::to_array` (C++20) to avoid redundantly specifying types and lengths.
3. **Array Passing:** Always pass `std::array` by `const` reference (`const std::array<T, N>&`) to avoid deep copies.
4. **Bounds Checking:** Use `std::get<N>(arr)` for compile-time checked indexing on `constexpr` arrays.
5. **C-Style Arrays:** Avoid C-style arrays due to implicit pointer decay and loss of length information. Use `constexpr std::string_view` for string constants and `std::array` for numeric data.
6. **Multidimensional Data:** Flatten 2D/3D grids into a single 1D `std::array` and use `std::mdspan` (C++23) to provide multidimensional index operations (`view[row, col]`).
