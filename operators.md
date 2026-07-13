# C++ Operators: Core Concepts & Gotchas

A quick reference for operator precedence, math quirks, and logical evaluation in C++.

---

## 1. Precedence vs. Associativity

* **Precedence:** Determines which operators are grouped first (e.g., `*` happens before `+`).
* **Associativity:** Breaks ties when operators have the *same* precedence. 
    * *Left-to-Right:* `7 - 4 - 1` evaluates as `(7 - 4) - 1`.
    * *Right-to-Left:* `x = y = 5` evaluates as `x = (y = 5)`.
* **Order of Evaluation is Unspecified:** C++ does *not* promise whether the left or right side of an operator (like `+`) or function arguments evaluate first. 
    * > ⚠️ **Never write code that depends on evaluation order** (e.g., `add(x, ++x)` is undefined behavior).

---

## 2. Arithmetic & Math Quirks

* **Integer Division:** If both operands are integers, the fraction is completely dropped, not rounded (e.g., `7 / 4` is `1`).
* **Floating-Point Division:** If at least one operand is a float/double, you get a decimal result (e.g., `7.0 / 4` is `1.75`). Use `static_cast<double>(x)` to force floating-point math on integer variables.
* **Remainder/Modulo (`%`):** Only works on integers. It takes the sign of the *first* operand. (e.g., `-6 % 4` is `-2`). 
    * *Best Practice:* Always check parity against `0` (e.g., `x % 2 != 0` for odd numbers) to avoid negative number bugs.
* **No Exponent Operator:** C++ does not use `^` for exponents (that is Bitwise XOR). Use `std::pow()` from `<cmath>`.

---

## 3. Increment (`++`) & Decrement (`--`)

* **Prefix (`++x`):** Increments `x`, then returns the *new* value. **(Preferred for performance and predictability).**
* **Postfix (`x++`):** Makes a copy of `x`, increments the original `x`, and returns the *old* copy. 

---

## 4. The Conditional (Ternary) Operator (`?:`)

* **Syntax:** `condition ? execute_if_true : execute_if_false;`
* Acts as an inline `if-else`. 
* *Gotcha:* It has very low precedence. Always wrap it in parentheses if using it inside a larger expression: 
  
      std::cout << ((x > y) ? x : y);

---

## 5. Relational & Logical Operators

* **Floating-Point Comparisons:** NEVER use `==` or `!=` to compare calculated floating-point numbers due to invisible rounding errors. Use an "epsilon" function to check if they are "close enough."
* **Logical NOT (`!`):** Extremely high precedence. `!x > y` means `(!x) > y`. Write `!(x > y)` instead.
* **Short-Circuit Evaluation:**
    * **AND (`&&`):** If the left side is `false`, the right side is *never* evaluated.
    * **OR (`||`):** If the left side is `true`, the right side is *never* evaluated.
    * > ⚠️ **Warning:** Never put an operation with a side effect (like `++x`) on the right side of `&&` or `||`, because it might not execute.
