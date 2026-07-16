# C++ Bit Manipulation & Numeral Systems

A practical cheat sheet for std::bitset, bitwise operators, two's complement binary math, and bit masking.

---

## 1. Introduction to Bit Flags

In standard architectures, the smallest addressable unit of memory is 1 byte (8 bits). Storing a boolean value (which only needs 1 bit) in a full byte leaves 7 bits unused.

* **Bit Flags:** Treating each individual bit within an integer object as an independent, isolated boolean value.
* **Terminology:** * Bit holding `1` -> True / On / Set.
  * Bit holding `0` -> False / Off / Not Set.
  * Changing a bit's state -> Flipped / Inverted.
* *Best Practice:* Always use **unsigned integers** or `std::bitset` when performing bit manipulation to avoid unexpected compiler-specific signed behavior.

---

## 2. Bit Manipulation via `std::bitset` (`<bitset>`)

Bits are numbered from **right to left**, starting with index `0`. `std::bitset` provides clean member functions to query and modify these positions.

### Key Query & Modification Methods
* **`bits.test(pos)`:** Queries whether a specific bit is `0` or `1` (returns a bool).
* **`bits.set(pos)`:** Turns a bit **On** (forces it to `1`).
* **`bits.reset(pos)`:** Turns a bit **Off** (forces it to `0`).
* **`bits.flip(pos)`:** Flips a bit state (inverts `1` <-> `0`).

### Advanced Status Checks
* **`bits.size()`:** Returns the total number of bits contained in the bitset wrapper.
* **`bits.count()`:** Returns the exact number of bits currently set to `true`.
* **`bits.all()`:** Returns `true` if *every single bit* inside the object is set to `1`.
* **`bits.any()`:** Returns `true` if *at least one* bit inside the object is set to `1`.
* **`bits.none()`:** Returns `true` if *all bits* inside the object are completely `0`.

> ⚠️ **The Memory Surprise:** `std::bitset` is heavily optimized for speed, not memory compression. A `std::bitset<8>` typically pads its allocation size up to the nearest `sizeof(size_t)` (4 bytes on 32-bit systems, 8 bytes on 64-bit systems).

---

## 3. The 6 Bitwise Operators

Bitwise operators apply logical operations directly to each pair of underlying bits columns. They are non-modifying unless combined with assignment (`=`).

| Operator | Symbol | Evaluation Rule |
| :--- | :--- | :--- |
| **Bitwise NOT** | `~x` | Unary. Flips every individual bit (`0` becomes `1`, `1` becomes `0`). |
| **Bitwise AND** | `x & y` | Column bit is `1` **only if both** corresponding bits are `1`. |
| **Bitwise OR** | `x \| y` | Column bit is `1` **if any** of the corresponding bits are `1`. |
| **Bitwise XOR** | `x ^ y` | Column bit is `1` **only if** the paired bits are completely different. |
| **Left Shift** | `x << n` | Shifts bits left by `n` slots. Bounces highest bits off the end (lost). Fills new low slots with `0`. |
| **Right Shift** | `x >> n` | Shifts bits right by `n` slots. Bounces lowest bits off the end (lost). |

### Overloading Precedence Trap
Since `<<` and `>>` are also used by `std::cout` and `std::cin`, you **must wrap your bit-shifting operations in parentheses** when embedding them inside I/O streams:

    std::cout << (x << 1) << '\n'; // Safe bit-shifting output

### Narrow Integral Promotion Warning
Applying bitwise operations (especially `~` and `<<`) to types narrower than `int` (like `std::uint8_t`) triggers **automatic integral promotion** to a signed or unsigned `int`. 
* *Best Practice:* Avoid bit-shifting narrow integers when possible, or wrap the result in an explicit `static_cast` back to your narrow type to suppress safety warnings:

    c = static_cast<std::uint8_t>(~c);

---

## 4. Binary Math & Two's Complement Representation

### Converting Binary to Decimal
Multiply each binary digit position by its corresponding base-2 power value ($2^0=1, 2^1=2, 2^2=4, 2^3=8, \dots$), then sum up the components where a `1` resides.

### Two's Complement System (Signed Negation)
Modern systems encode negative whole numbers using **Two's Complement**. The leftmost bit acts as the sign marker (`0` = positive, `1` = negative).

To transform a positive decimal number into its negative binary equivalent:
1. Write down the positive value in standard binary form (e.g., `5` -> `0000 0101`).
2. **Invert all the bits** (Bitwise NOT: `1111 1010`).
3. **Add 1** to the inverted result (`1111 1011`). This sequence eliminates a duplicate "negative zero" representation.

---

## 5. Traditional Bit Masking

A **bit mask** is a predefined literal constant used to target, clear, or evaluate explicit bit allocations inside a flag variable.

### Constructing Bit Masks
Modern C++ (C++14+) enables binary literals directly. For older versions, utilize the left-shift operator (`1 << pos`):

    // C++14 Binary Literals
    constexpr std::uint8_t isHungry   { 0b0000'0001 }; // Bit 0
    constexpr std::uint8_t isSad      { 0b0000'0010 }; // Bit 1
    constexpr std::uint8_t isMad      { 0b0000'0100 }; // Bit 2
    
    // C++11 Left-Shift Equivalents
    constexpr std::uint8_t isHappy    { 1 << 3 };      // Bit 3 (0000 1000)
    constexpr std::uint8_t isLaughing { 1 << 4 };      // Bit 4 (0001 0000)

### Code Blueprint for Bit Mask Operations

* **Testing a Bit State:** Use Bitwise AND (`&`). Evaluates to non-zero if the bit is on.
  
    if (flags & isHappy) { /* Bit 3 is true */ }

* **Setting a Bit (Turning On):** Use Bitwise OR assignment (`|=`).
  
    flags |= isHappy; 
    flags |= (isHappy | isLaughing); // Turn on multiple flags simultaneously

* **Resetting a Bit (Turning Off):** Combine Bitwise AND assignment with a negated mask (`&= ~mask`).
  
    flags &= ~isHappy;
    flags &= ~(isHappy | isLaughing); // Clear multiple flags simultaneously

* **Flipping a Bit (Toggling):** Use Bitwise XOR assignment (`^=`).
  
    flags ^= isHappy; // Flips 0->1 or 1->0

---

## 6. Multi-Bit Mask Extraction (RGBA Color Blueprint)

Bit masks can encompass multiple bits consecutively to extract complex packed structures, such as reading distinct 8-bit channels from a single packed 32-bit pixel value.

    #include <cstdint>
    #include <iostream>

    int main() {
        constexpr std::uint32_t redMask   { 0xFF000000 };
        constexpr std::uint32_t greenMask { 0x00FF0000 };
        constexpr std::uint32_t blueMask  { 0x0000FF00 };
        constexpr std::uint32_t alphaMask { 0x000000FF };

        std::uint32_t pixel { 0xFF7F3300 }; // Hex color sample

        // Isolate the target bits via AND, then right-shift them into the lowest 8 bits
        std::uint8_t r { static_cast<std::uint8_t>((pixel & redMask) >> 24) };
        std::uint8_t g { static_cast<std::uint8_t>((pixel & greenMask) >> 16) };
        std::uint8_t b { static_cast<std::uint8_t>((pixel & blueMask) >> 8) };
        std::uint8_t a { static_cast<std::uint8_t>(pixel & alphaMask) };

        return 0;
    }
