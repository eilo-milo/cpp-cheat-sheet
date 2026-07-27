# 8.13 / 8.14 — Pseudo-Random Number Generation & Mersenne Twister

Computers are deterministic by design, making them incapable of generating *truly* random numbers through pure software. Instead, modern programs simulate randomness using algorithms known as **Pseudo-Random Number Generators (PRNGs)**.

---

## 1. Core PRNG Theory & Concepts

* **Algorithm:** A finite sequence of reusable instructions designed to solve a problem or calculate a result.
* **State & Stateful Algorithms:** An algorithm is **stateful** if it retains information across calls. The stored values are referred to as its **state**.
* **Deterministic Behavior:** Given the exact same initial state (input), a PRNG will always generate the exact same sequence of numbers.
* **PRNG Sequence Generation:**
  1. The current state is modified via mathematical operations.
  2. The new state is used to generate the next number in the sequence.

---

## 2. Seeding & Underseeding

A **seed** is the initial value (or set of values) used to set the starting state of a PRNG. 

* **The Seed Rule:** Because PRNGs are deterministic, supplying the same seed value results in the exact same sequence of pseudo-random numbers.
* **Underseeding:** Occurs when a PRNG is initialized with fewer bits of quality seed data than its internal state requires.
  * *Example:* `std::mt19937` has an internal state size of **19,937 bits** (624 32-bit integers). Seeding it with a single 32-bit integer severely underseeds the generator, reducing the quality of generated randomness.
* **`std::seed_seq` (Seed Sequence):** A helper type introduced to distribute seed data evenly across a PRNG's state array. Passing multiple random seeds to `std::seed_seq` helps mitigate underseeding issues.

---

## 3. What Makes a Good PRNG?

1. **Distribution Uniformity:** Numbers across the output range should be generated with equal probability (verifiable via a histogram).
2. **Unpredictability:** Examining previous outputs should not allow someone to determine future values.
3. **Good Dimensional Distribution:** Generates low, high, odd, and even numbers randomly across the spectrum.
4. **High Periodicity:** The **period** is the length of the sequence before a PRNG starts repeating itself. Good PRNGs maintain a massive period across all seed values.
5. **Computational Efficiency:** Minimal execution time and memory footprint.

---

## 4. Randomization in C++ (`<random>`)

C++ provides random generation mechanisms inside the standard `<random>` header. 

### Comparison of C++ PRNG Engines

| Type Name | Family | State Size | Quality | Recommendation |
| :--- | :--- | :--- | :--- | :--- |
| `std::minstd_rand` | Linear Congruential (LCG) | 4 bytes | Awful | **Do Not Use** |
| `std::mt19937` / `_64` | Mersenne Twister | 2500 bytes | Decent | **Recommended** (Default Choice) |
| `std::ranlux24` | Subtract and Carry | 196 bytes | Good | Avoid (Very Slow) |
| `std::default_random_engine` | Implementation Defined | Varies | Unknown | **Do Not Use** |
| `rand()` (from `<cstdlib>`) | C-Style LCG | 4 bytes | Awful | **Do Not Use** |

> ⚠️ **Note on Mersenne Twister:** While `std::mt19937` is the standard choice for games and general applications, its output becomes predictable after observing 624 generated numbers. **Do not use `std::mt19937` for cryptographic or security purposes**.

---

## 5. Modern Seeding Techniques

### Method 1: Using `std::random_device` + `std::seed_seq` (Recommended)
`std::random_device` requests non-deterministic random data directly from the operating system.

```cpp
#include <iostream>
#include <random>

int main()
{
    std::random_device rd{};
    
    // Pass 8 random integers from std::random_device to seed_seq for full-state mixing
    std::seed_seq ss{ rd(), rd(), rd(), rd(), rd(), rd(), rd(), rd() }; 
    
    std::mt19937 mt{ ss }; // Initialize Mersenne Twister with seed_seq

    std::uniform_int_distribution die6{ 1, 6 }; // Uniform distribution between 1 and 6

    for (int count{ 1 }; count <= 10; ++count)
    {
        std::cout << die6(mt) << '\t';
    }
    std::cout << '\n';

    return 0;
}
```

### Method 2: System Clock (Alternative / Fallback)
Uses system ticks from `<chrono>` to vary initial state across executions.

```cpp
#include <chrono>
#include <iostream>
#include <random>

int main()
{
    std::mt19937 mt{ static_cast<std::mt19937::result_type>(
        std::chrono::steady_clock::now().time_since_epoch().count()
    ) };

    std::uniform_int_distribution die6{ 1, 6 };
    std::cout << die6(mt) << '\n';

    return 0;
}
```

> 💡 **Best Practices:**
> * **Seed Only Once:** Initialize your PRNG once at program startup. Re-seeding repeatedly (e.g., inside a loop or function) degrades randomness and performance.
> * **Debugging Tip:** To reproduce bugs dependent on random events, temporarily seed your PRNG with a fixed constant integer (e.g., `std::mt19937 mt{ 5 };`).

---

## 6. Global Production Helper: `Random.h`

To eliminate parameter-passing overhead and prevent re-initialization bugs across multi-file projects, use this self-seeding, header-only `Random` namespace:

### `Random.h`
```cpp
#ifndef RANDOM_MT_H
#define RANDOM_MT_H

#include <chrono>
#include <random>

namespace Random
{
    // Returns a fully-seeded Mersenne Twister instance
    inline std::mt19937 generate()
    {
        std::random_device rd{};

        // Mix clock state with OS-provided random entropy
        std::seed_seq ss{
            static_cast<std::seed_seq::result_type>(std::chrono::steady_clock::now().time_since_epoch().count()),
            rd(), rd(), rd(), rd(), rd(), rd(), rd()
        };

        return std::mt19937{ ss };
    }

    // Single global PRNG instance shared safely across files
    inline std::mt19937 mt{ generate() };

    // Generate a random int between [min, max] (inclusive)
    inline int get(int min, int max)
    {
        return std::uniform_int_distribution{ min, max }(mt);
    }

    // Template overload for matching type bounds (e.g., size_t, long, etc.)
    template <typename T>
    T get(T min, T max)
    {
        return std::uniform_int_distribution<T>{ min, max }(mt);
    }

    // Template overload for heterogeneous types or explicit return typing
    template <typename R, typename S, typename T>
    R get(S min, T max)
    {
        return get<R>(static_cast<R>(min), static_cast<R>(max));
    }
}

#endif
```

### Usage Example (`main.cpp`)
```cpp
#include "Random.h"
#include <iostream>

int main()
{
    // 1. Basic int bounds [1, 6]
    std::cout << Random::get(1, 6) << '\n';

    // 2. Unsigned integral bounds
    std::cout << Random::get(1u, 6u) << '\n';

    // 3. Explicit return type specialization
    std::cout << Random::get<std::size_t>(0, 100) << '\n';

    return 0;
}
```
