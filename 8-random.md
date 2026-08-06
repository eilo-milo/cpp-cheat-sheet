# C++ Cheat Sheet: Random Number Generation & `<random>`

A reference guide covering random number theory, Pseudo-Random Number Generators (PRNGs), proper seeding mechanics, distributions, and production-ready C++ code patterns.

---

## 1. Randomness Theory & PRNG Concepts

### Overview
Computers are deterministic systems that cannot generate truly random numbers through software alone. Instead, programs simulate randomness using algorithms.

- **PRNG (Pseudo-Random Number Generator):** A deterministic algorithm that calculates a sequence of numbers simulating random properties based on an internal state.
- **State:** The variable(s) maintained across calls that store the current position in the sequence.
- **Seed:** The initial value(s) used to set the PRNG's state.

> **Key Rule:** Given the same initial seed, a PRNG will **always** produce the exact same sequence of numbers.

### Properties of a Good PRNG
1. **Distribution Uniformity:** Every number in the target range has an equal probability of occurring.
2. **Non-Predictability:** Future values cannot easily be deduced from prior outputs.
3. **High Dimensional Distribution:** Generates low, high, odd, and even values evenly across time.
4. **Long Period:** Generates a vast sequence before repeating its state loop (e.g., $2^{19937}-1$ for Mersenne Twister).
5. **Performance & Efficiency:** Minimal memory footprint and fast state transitions.

---

## 2. Standard C++ PRNG Engines (`<random>`)

C++ provides several engine families in the `<random>` header.

| Engine | Family | Period | State Size | Quality | Recommendation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `std::mt19937` | Mersenne Twister (32-bit) | $2^{19937}-1$ | ~2500 bytes | Decent | **Preferred Standard Choice** |
| `std::mt19937_64` | Mersenne Twister (64-bit) | $2^{19937}-1$ | ~2500 bytes | Decent | **Preferred for 64-bit bounds** |
| `std::default_random_engine` | Implementation-defined | Varies | Varies | Varies | **Avoid** (Unpredictable behavior) |
| `std::minstd_rand` | Linear Congruential | $2^{31}$ | 4 bytes | Poor | **Avoid** |
| `rand()` | Legacy C LCG | $2^{31}$ | 4 bytes | Awful | **Avoid** (Legacy C only) |

> **Security Note:** `std::mt19937` is **not cryptographically secure**. Its state can be predicted after observing 624 outputs. For cryptographic applications, use specialized libraries (e.g., ChaCha20).

---

## 3. Seeding Best Practices & `std::seed_seq`

### Underseeding Problem
`std::mt19937` requires 19,937 bits (~2.5 KB) of internal state. Initializing it with a single 32-bit or 64-bit integer significantly **underseeds** the generator, resulting in compromised output quality (e.g., certain initial values will never occur).

### Proper Seeding Pattern
Use `std::seed_seq` initialized with multiple entropy sources (system clock + multiple `std::random_device` reads) to populate the state evenly.

```cpp
#include <iostream>
#include <random>
#include <chrono>

int main() {
    std::random_device rd{};

    // Gather entropy from both clock ticks and OS random device
    std::seed_seq ss{
        static_cast<std::seed_seq::result_type>(
            std::chrono::steady_clock::now().time_since_epoch().count()
        ),
        rd(), rd(), rd(), rd(), rd(), rd(), rd()
    };

    // Correctly seeded 32-bit Mersenne Twister
    std::mt19937 mt{ ss };

    // Produce random output
    std::cout << mt() << '\n';
}
```

> **Rules of Thumb for Seeding:**
> 1. **Only seed once** per generator during application/component initialization. Never reseed inside a loop or function.
> 2. Do not use `std::random_device` as your primary generator (it can be slow or exhaust OS entropy pools). Use it solely to seed PRNG engines.

---

## 4. Distributions

PRNG engines produce raw unsigned integers spanning their entire native range. **Distributions** map these raw values uniformly into a target range $[X, Y]$.

### Common Uniform Distributions

```cpp
#include <iostream>
#include <random>

int main() {
    std::mt19937 mt{ 1337 }; // Example engine (fixed seed for demonstration)

    // 1. Uniform Integer Distribution [min, max] inclusive
    std::uniform_int_distribution<int> die6{ 1, 6 };
    std::cout << "Dice Roll: " << die6(mt) << '\n';

    // 2. Uniform Real (Floating-Point) Distribution [min, max)
    std::uniform_real_distribution<double> percent{ 0.0, 1.0 };
    std::cout << "Probability: " << percent(mt) << '\n';
}
```

---

## 5. Production Pattern: Header-Only `Random.h`

To avoid creating and seeding PRNG engines repeatedly across functions or files, use a thread-safe / header-only global helper in a dedicated namespace.

### `Random.h` Implementation (C++17+)

```cpp
#ifndef RANDOM_MT_H
#define RANDOM_MT_H

#include <chrono>
#include <random>
#include <type_traits>

namespace Random
{
    // Factory function to instantiate a fully seeded std::mt19937 engine
    inline std::mt19937 generate()
    {
        std::random_device rd{};
        std::seed_seq ss{
            static_cast<std::seed_seq::result_type>(
                std::chrono::steady_clock::now().time_since_epoch().count()
            ),
            rd(), rd(), rd(), rd(), rd(), rd(), rd()
        };
        return std::mt19937{ ss };
    }

    // Inline global Mersenne Twister engine instance (ODR-safe in C++17)
    inline std::mt19937 mt{ generate() };

    // Generate random integer between [min, max] inclusive
    inline int get(int min, int max)
    {
        return std::uniform_int_distribution<int>{min, max}(mt);
    }

    // Template overloads for arbitrary integral types (short, long, size_t, etc.)
    template <typename T>
    T get(T min, T max)
    {
        return std::uniform_int_distribution<T>{min, max}(mt);
    }

    // Overload supporting mixed parameter types with explicit return type selection
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
#include <cstddef>

int main() {
    // Basic integer calls
    int roll { Random::get(1, 6) };             // int in [1, 6]
    auto uVal { Random::get(1u, 100u) };        // unsigned int in [1, 100]
    
    // Explicit return type template parameter
    auto index { Random::get<std::size_t>(0, 10) }; // std::size_t in [0, 10]

    // Access engine directly with custom distribution
    std::uniform_real_distribution<double> dist{ 0.0, 1.0 };
    double chance = dist(Random::mt);

    std::cout << "Roll: " << roll << ", Chance: " << chance << '\n';
}
```

---

## 6. Debugging Random Programs

Because PRNG outputs vary on every execution, bugs can be non-deterministic and difficult to reproduce.

```cpp
// DEBUGGING STRATEGY:
// Replace dynamic entropy seeding with a fixed constant seed (e.g., 5 or 42)
// std::mt19937 mt{ 5 }; 
```
* Using a **fixed seed** ensures the execution path and generated numbers remain 100% deterministic, enabling step-by-step debugger tracing. Restore dynamic seeding once the bug is fixed.
