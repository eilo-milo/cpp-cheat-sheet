# C++ Operators & Compound Expressions

A quick reference for evaluation rules, arithmetic behavior, and logical operations in C++.

---

## 1. Precedence, Associativity & Evaluation Order

To evaluate a compound expression, the compiler parses the grouping of operands with operators at compile-time.

* **Operator Precedence:** Operators with higher precedence (lower number, e.g., level 5 `*` vs level 6 `+`) grab their operands first.
* **Operator Associativity:** If adjacent operators have the same precedence, associativity determines grouping direction (Left-to-Right `L->R` or Right-to-Left `R->L`).
* **Order of Evaluation:** Precedence and associativity **do not** dictate the execution order of operands or function arguments. Operands can be evaluated in *any* order by the compiler, leading to potential bugs if side effects are mixed.

### Order of Evaluation Pitfall

<pre><code>// ⚠️ AMBIGUOUS EXPRESSION (Undefined Behavior)
// GCC evaluates right-to-left; Clang evaluates left-to-right.
printCalculation(getValue(), getValue(), getValue()); 

// ✅ UNAMBIGUOUS SOLUTION
int a{ getValue() }; 
int b{ getValue() }; 
int c{ getValue() }; 
printCalculation(a, b, c); // Safe and deterministic
</code></pre>

**Best Practices:**
* Use parentheses to make non-trivial compound expressions explicit (Rule of thumb: Parenthesize everything except standard `+`, `-`, `*`, `/`).
* Expressions with a single assignment operator (and no commas) do not need the right-hand side parenthesized: `x = y + z + w;`.
* **Never** use a variable that has a side effect applied to it more than once in a single statement.

---

## 2. Arithmetic Operators

### Modifying vs. Non-Modifying Operators
* **Non-Modifying:** Calculate and return a value without altering operands (e.g., standard `+`, `-`, `*`, `/`).
* **Modifying:** Alter the left operand permanently (e.g., `=`, `+=`, `*=`, `++`, `--`).

### Division and Remainder (`%`)
* **Floating-Point Mode:** If either operand is a float/double, fractional parts are kept.
* **Integer Mode:** If both operands are integers, fractions are dropped entirely (Truncated: `7 / 4` is `1`).
* **Type Forcing:** Use `static_cast<double>(int_var)` on one operand to force floating-point division.
* **Division by Zero:** Integer division by `0` crashes the program (UB). Floating-point division by `0.0` yields `Inf` or `NaN` on IEEE754 systems.
* **The Remainder Operator (`%`):** Returns the remainder of integer division. The result always takes the **sign of the first operand** (`x`).
  * *Best Practice:* Always compare the result of a remainder operation against `0` (e.g., `(x % 2) != 0` to check for odd numbers safely, avoiding negative sign bugs).

### Exponentiation
C++ does not have an exponent operator (`^` is Bitwise XOR). Use `std::pow(base, exp)` from `<cmath>` for doubles, or write a dedicated integer loop function to prevent rounding/precision drift.

---

## 3. Increment & Decrement Operators

Favor **Prefix** versions over Postfix versions as a baseline habit.

| Form | Syntax | Behavior | Performance |
| :--- | :--- | :--- | :--- |
| **Prefix Increment** | `++x` | Increments `x`, then returns `x`. | Highly Performant |
| **Prefix Decrement** | `--x` | Decrements `x`, then returns `x`. | Highly Performant |
| **Postfix Increment** | `x++` | Copies `x`, increments original `x`, returns copy. | Slower (Creates Temporary) |
| **Postfix Decrement** | `x--` | Copies `x`, decrements original `x`, returns copy. | Slower (Creates Temporary) |

---

## 4. The Comma Operator (`,`)

Allows evaluating multiple expressions where one is expected. Evaluates left, evaluates right, returns right.
* *Best Practice:* **Avoid using the comma operator entirely**, except inside `for` loops. It has the lowest precedence and causes severe readability issues.
* Do not confuse the comma operator with the *separator comma* used in function arguments or definitions.

---

## 5. The Conditional Ternary Operator (`?:`)

C++'s only ternary operator, acting as a shortcut for a simple `if-else` value assignment.
* **Syntax:** `condition ? expression1 : expression2;`
* **Type Rule:** The types of `expression1` and `expression2` must match or be implicitly convertible by the compiler.
* **Parenthesization Best Practice:** Always parenthesize the entire ternary operation when embedding it inside a compound expression: `std::cout << ((x > y) ? x : y);`.

---

## 6. Relational Operators & Floating-Point Pitfalls

Relational operators (`<`, `>`, `<=`, `>=`, `==`, `!=`) return a boolean `true` or `false`.

> ⚠️ **Floating-Point Equality Warning:** **Never** use `==` or `!=` on computed floating-point numbers. Rounding errors accumulate, making exact comparisons fail.

### Safe Floating-Point Comparison
To check if two floats are "equal", see if they are close enough within an absolute and relative tolerance factor (Epsilon):

<pre><code>bool approximatelyEqualRel(double a, double b, double relEpsilon) {
    return (std::abs(a - b) <= (std::max(std::abs(a), std::abs(b)) * relEpsilon));
}

bool approximatelyEqualAbsRel(double a, double b, double absEpsilon, double relEpsilon) {
    if (std::abs(a - b) <= absEpsilon) return true; // Handles near-zero comparisons
    return approximatelyEqualRel(a, b, relEpsilon);
}
</code></pre>

---

## 7. Logical Operators

Used to combine multiple relational expressions.

### Operators and Precedence
1. `!` (Logical NOT) — **Highest Precedence.** *Always wrap sub-comparisons in parentheses:* `if (!(x > y))`.
2. `&&` (Logical AND)
3. `||` (Logical OR) — **Lowest Precedence.**
* *Best Practice:* When mixing `&&` and `||`, explicitly parenthesize operations to ensure proper ordering: `(a && b) || (c && d)`.

### Short-Circuit Evaluation
Built-in `&&` and `||` evaluate from **left to right**. If the outcome is fully determined by the left operand, the right operand is completely skipped.
* `false && ...` -> right side is skipped.
* `true || ...` -> right side is skipped.
* *Warning:* Never place expressions with necessary side effects (like `++y`) on the right side of a logical operator.

### De Morgan's Laws
When distributing a logical negation over a compound condition, you must invert the inner logical operators:
* `!(x && y)` is equivalent to `!x || !y`
* `!(x || y)` is equivalent to `!x && !y`

### Logical XOR Shortcut
C++ does not have a logical XOR operator. If operands are confirmed `bool` variables, use inequality `!=` as a functional replacement:
* `if (a != b)` is equivalent to `a XOR b`.
