# ICS 45C — C++ Foundations
> Covers the C++ Tour and Arrays slide decks + the introductory homeworks (HW1, HW2 basics)

HW1 and HW2 are intentionally short here: they are introductory. Skim this guide and move on to the per-topic guides for the heavy material.

---

## Table of Contents
1. [Program Structure](#1-program-structure)
2. [Built-in Types and Literals](#2-built-in-types-and-literals)
3. [Variables, constexpr, and Storage](#3-variables-constexpr-and-storage)
4. [Operators and Expressions](#4-operators-and-expressions)
5. [Input and Output Streams](#5-input-and-output-streams)
6. [Control Flow](#6-control-flow)
7. [Functions](#7-functions)
8. [References (Parameter Modes)](#8-references-parameter-modes)
9. [Fixed C-style Arrays](#9-fixed-c-style-arrays)
10. [`char` as Index](#10-char-as-index)
11. [A First Class: `Stack` (HW1)](#11-a-first-class-stack-hw1)
12. [File Organization (`.hpp` / `.cpp`)](#12-file-organization-hpp--cpp)
13. [Compilation Pipeline](#13-compilation-pipeline)
14. [HW1 / HW2 Pattern Quick Reference](#14-hw1--hw2-pattern-quick-reference)
15. [Key Rules to Remember](#15-key-rules-to-remember)

---

## 1. Program Structure

Every C++ program begins at `main()`. A minimal program:

```cpp
#include <iostream>          // brings declarations for cin/cout
int main() {                 // entry point
    int i = 4;               // local variable, initialized to 4
    int sq = i * i;          // sq = 16
    std::cout << sq << '\n'; // print to terminal, newline
    return 0;                // exit status (0 = success)
}
```

- `#include <X>` pulls a standard header into the file.
- `std::cout` is the standard output stream. `<<` is the **insertion** operator.
- Statements end in `;`. Code blocks live inside `{ ... }`.

---

## 2. Built-in Types and Literals

| Type | Examples | Notes |
|---|---|---|
| `int` | `-1`, `0`, `42` | whole numbers |
| `double` | `0.0`, `3.14`, `-1.5` | floating point (note the decimal) |
| `char` | `'A'`, `'9'`, `'\n'` | **single** quotes, stored as 1 byte |
| `bool` | `true`, `false` | logical |
| `std::string` | `"hello"`, `"World"` | **double** quotes, a class from `<string>` |
| C array | `int a[5] = {1,2,3,4,5}` | fixed size known at compile time |

Single quotes vs double quotes is a common bug: `'A'` is one character (an `int`-like value), `"A"` is a string of length 1 plus a null terminator.

---

## 3. Variables, constexpr, and Storage

```cpp
int numberOfStudents = 30;       // initialize at declaration
int automobileVelocity{0};       // brace initialization (preferred for safety)
int foo;                         // uninitialized — value is garbage!

constexpr double PI = 3.1415926; // compile-time constant, cannot be changed
constexpr int STK_MAX = 1000;    // used for fixed-size arrays in HW1
```

- A `constexpr` is a value the compiler can fix at compile time. Use it for array sizes and "magic" numbers.
- An uninitialized local variable has an indeterminate value — always initialize.

### Three storage areas

| Area | What lives there | Lifetime |
|---|---|---|
| **Static storage** (data segment) | globals, `static` variables, string literals | whole program |
| **Stack** | function parameters, local variables | scope (`{ ... }`) |
| **Free store / Heap** | objects created with `new` / `new[]` | until `delete` / `delete[]` |

In HW1 every object lives on the stack. Heap allocation appears in HW4+.

---

## 4. Operators and Expressions

| Group | Operators | Notes |
|---|---|---|
| Arithmetic | `+ - * /` `%` | `%` is integer modulus |
| Assignment | `= += -= *= /= %=` | modify state |
| Increment/Decrement | `++i  i++  --i  i--` | prefix vs postfix matters in expressions |
| Comparison | `== != < > <= >=` | returns `bool` |
| Logical | `&&  \|\|  !` | short-circuit |

Precedence (high → low): unary `++ -- + -` → `* / %` → binary `+ -` → comparisons → `= += ...`. **When in doubt, add parentheses.** There is no `**`; use `x * x` or `std::pow(x, 2)`.

---

## 5. Input and Output Streams

`std::cout` is `std::ostream`. `std::cin` is `std::istream`. `<<` writes, `>>` reads:

```cpp
std::cout << "Hello, " << name << '\n';   // chains left-to-right
int age;
std::cout << "Enter your age: ";
std::cin >> age;                           // reads one whitespace-separated token
```

`std::cin >> name` reads one **word** (stops at whitespace). For an entire line use:

```cpp
std::string line;
std::getline(std::cin, line);              // reads up to '\n'
```

### File streams (`#include <fstream>`)

```cpp
std::ifstream in("input.txt");             // open for reading
std::ofstream out("output.txt");           // create/overwrite for writing
std::string line;
while (std::getline(in, line))             // returns false at EOF
    out << line << '\n';
in.close();  out.close();                  // optional; auto on scope exit
```

**Important convention used across HW2-HW9:** functions take `std::istream&` / `std::ostream&` parameters so the same code works with `cin`, files, or string streams.

---

## 6. Control Flow

```cpp
if (i == 0) { ... } else if (i > 0) { ... } else { ... }

for (int i = 0; i < 10; ++i) {             // classic for
    std::cout << i;
}

for (auto c : line) {                      // range-for over each element
    std::cout << c;
}

int i = 10;
while (i > 0) { --i; }                     // while loop

do { --i; } while (i > 0);                 // do-while runs body at least once
```

- `break;` exits the nearest loop.
- `continue;` jumps to the next iteration.
- `switch` (covered in Linked List guide) handles discrete `int`/`char`/`enum` values.

---

## 7. Functions

```cpp
// Declaration (a.k.a. prototype) — interface, usually in .hpp
int square(int n);

// Definition — implementation, usually in .cpp
int square(int n) {
    return n * n;
}

int main() {
    int result = square(4);                // 16
}
```

**Default values for parameters:**

```cpp
double to_centigrade(double f = 32.0) {    // f defaults to 32.0
    return 5.0 * (f - 32.0) / 9.0;
}
to_centigrade();      // uses default
to_centigrade(6.0);   // overrides default
```

A function must be **declared** before it is called. If only declared (not defined), the linker errors out (`undefined reference`).

---

## 8. References (Parameter Modes)

A **reference** (`T&`) is an alias for an existing object. The function can read and modify the original.

```cpp
void increment(int& x) {                   // x is a reference to caller's variable
    x = x + 1;                              // modifies the caller's variable
}

int main() {
    int z = 10;
    increment(z); increment(z);
    std::cout << z;                         // prints 12
}
```

| Mode | Syntax | What happens | When to use |
|---|---|---|---|
| Pass by value | `void f(T x)` | function gets a **copy** | small types (int, char) or you don't need to modify caller |
| Pass by reference | `void f(T& x)` | function shares the caller's object — can modify it | need to modify, or copy is expensive |
| Pass by const reference | `void f(const T& x)` | shares, but cannot modify | large objects, read-only |

`const T&` is the idiom for "pass cheaply but don't change it" — you'll see this on every HW from HW2 on.

---

## 9. Fixed C-style Arrays

A C array is a fixed-size contiguous block of elements. The size must be known at compile time (a `constexpr`).

```cpp
constexpr int N = 5;
int a[N] = {0};                            // size 5, all zero
int a2[] = {1, 2, 3};                      // size deduced (3)
a[0] = 42;                                  // index access, no bounds check
int x = a[N - 1];                           // last element
// a[N] or a[-1]  →  undefined behavior! No runtime error, but corrupt memory.
```

Arrays do **not** know their own length when passed to a function — you must pass the length too:

```cpp
void print(int a[], int len) {              // a[] is really int*
    for (int i = 0; i < len; ++i)           // must use len, not "a.size()"
        std::cout << a[i] << ' ';
}
```

Range-for **does** work directly on a stack array (because the compiler still knows its size at the call site):

```cpp
int A[5] = {1,2,3,4,5};
for (auto x : A) std::cout << x;            // works on local arrays only
```

---

## 10. `char` as Index

A `char` is a 1-byte integer holding an ASCII code. `'A'` is 65, `'B'` is 66, ..., `'Z'` is 90. You can convert freely between `int` and `char`:

```cpp
char c = 'C';
int  i = c;                                 // i = 67
char back = static_cast<char>(i);           // 'C'
```

**Used in HW1 letter_count:** to count each letter, translate the char to an array index in `[0, 25]`:

```cpp
int counts[26] = {0};                       // 26 zeros, one slot per letter

int char_to_index(char ch) {
    return std::toupper(ch) - 'A';          // 'A'/'a' → 0, ..., 'Z'/'z' → 25
}

char index_to_char(int i) {
    return 'A' + i;                         // 0 → 'A', ..., 25 → 'Z'
}

void count(std::string s, int counts[]) {
    for (int i = 0; i < (int)s.size(); ++i)
        if (std::isalpha(s[i]))
            ++counts[char_to_index(s[i])];
}
```

Gotcha: never write `++counts['A']` — that would step on memory at index 65, way outside `counts[26]`.

---

## 11. A First Class: `Stack` (HW1)

HW1 introduces the **class** concept by wrapping a fixed-size buffer of `char`. Classes are covered in depth in Guide 02; here is just the shape.

```cpp
constexpr int STK_MAX = 1000;

class Stack {
    int  _top;                              // PRIVATE by default
    char buf[STK_MAX];
public:
    Stack() { _top = -1; }                  // constructor: starts empty

    void push(char c) {                     // add to top
        if (isFull()) { /* error */ return; }
        buf[++_top] = c;
    }

    char pop() {                            // remove and return top
        if (isEmpty()) return '@';          // sentinel for empty
        return buf[_top--];
    }

    char top()    { return isEmpty() ? '@' : buf[_top]; }
    bool isEmpty(){ return _top == -1; }
    bool isFull() { return _top == STK_MAX - 1; }
};
```

Key ideas already appearing here:
- **Data members are private**; clients use only public methods.
- The **constructor** (`Stack()`) initializes a fresh object.
- The buffer is a fixed-size C array — a known capacity (`STK_MAX`). Real `std::stack` grows.
- Reference parameters allow helper functions to modify the stack:

```cpp
void push_all(Stack& stk, std::string line) {       // & means reference
    for (char c : line) stk.push(c);
}
```

---

## 12. File Organization (`.hpp` / `.cpp`)

C++ projects split modules into two files:

| File | Contains |
|---|---|
| `module.hpp` (header) | declarations: class interface, function prototypes, `constexpr` constants |
| `module.cpp` (source) | definitions: method/function bodies |

A `.cpp` file `#include`s its `.hpp`. Other files that need the interface also include the `.hpp` — **not** the `.cpp`.

```cpp
// coins.hpp
#ifndef COINS_HPP                          // include guard
#define COINS_HPP
class Coins {
public:
    Coins(int q, int d, int n, int p);     // declaration only
    int total_value_in_cents() const;
private:
    int quarters, dimes, nickels, pennies;
};
#endif

// coins.cpp
#include "coins.hpp"
Coins::Coins(int q, int d, int n, int p)   // qualify with Coins::
  : quarters(q), dimes(d), nickels(n), pennies(p) {}

int Coins::total_value_in_cents() const {
    return quarters*25 + dimes*10 + nickels*5 + pennies;
}
```

The `#ifndef / #define / #endif` "include guard" prevents the header from being included twice in the same compilation unit. Every `.hpp` in the homework set uses this pattern.

---

## 13. Compilation Pipeline

Four phases turn `.cpp` → running program:

| Phase | What it does | Common error |
|---|---|---|
| **Preprocessor** | expands `#include`, `#define`, `#ifdef` | missing header file |
| **Compiler** | one pass over each `.cpp`, produces `.o` object files | "undeclared function" — fix by adding `#include` or forward declaration |
| **Linker** | combines `.o` files + libraries into one executable | "undefined reference" — declared but never defined |
| **Loader** | OS loads executable into memory, calls `main()` | usually invisible |

The build system (CMake in this course) automates compilation and linking. A failed link with "multiply defined function" usually means the same function body appears in two `.cpp` files (or a function body is defined in a `.hpp` without `inline`).

---

## 14. HW1 / HW2 Pattern Quick Reference

### HW1: `letter_count` — array indexed by character

```cpp
int counts[26] = {0};                       // 26 zeros
std::string s;
while (std::cin >> s)                       // read word by word until EOF
    for (char c : s)
        if (std::isalpha(c))
            ++counts[std::toupper(c) - 'A'];

for (int i = 0; i < 26; ++i)
    std::cout << char('A' + i) << ' ' << counts[i] << '\n';
```

### HW1: `Stack` of `char` with fixed buffer

```cpp
Stack stk;
std::string line;
while (std::cin >> line) {
    for (char c : line) stk.push(c);        // fill stack
    while (!stk.isEmpty())                  // drain (prints reversed)
        std::cout << stk.pop();
    std::cout << '\n';
}
```

### HW2: `Coins` — class with `operator==`, `operator<<`

```cpp
class Coins {
public:
    Coins(int q, int d, int n, int p);
    int total_value_in_cents() const;
    void print(std::ostream& out) const;
    bool operator==(const Coins& other) const = default;  // member-wise compare
private:
    int quarters, dimes, nickels, pennies;
};

std::ostream& operator<<(std::ostream& out, const Coins& c) {
    c.print(out);                            // delegate to member function
    return out;                              // allow chaining
}
```

`operator==(...) const = default` asks the compiler to generate member-wise equality. We'll see why a custom `==` is sometimes needed in later guides.

### HW2: `word_count` — first taste of STL

```cpp
#include <map>
#include <set>
std::set<std::string>      stopwords;       // unique sorted words to ignore
std::map<std::string, int> counts;          // word → count

std::string word;
while (in >> word) {
    if (!stopwords.contains(word))           // C++20
        ++counts[word];                      // map[key]++ is the counting idiom
}
for (const auto& [w, n] : counts)            // structured-binding range-for
    out << w << ' ' << n << '\n';
```

Both `set` and `map` keep their elements sorted. Iterating a `map` gives pairs of (key, value).

---

## 15. Key Rules to Remember

- **Single quotes for `char`, double quotes for `std::string`.** Different types entirely.
- **Initialize every variable** before reading it. `int x;` followed by `cout << x;` prints garbage.
- **A reference (`T&`) is an alias** for an existing object — modifications persist after the function returns.
- **`const T&`** = read-only sharing — use for any non-trivial type you don't need to modify.
- **C arrays don't carry their length.** Either use `constexpr` size, pass `(arr, len)`, or use `std::vector` (covered in the STL guide).
- **`char` is an integer.** Convert to an array index by subtracting `'A'`.
- **Stack vs heap matters later.** In HW1/HW2 everything is stack. In HW4+ you'll manage heap memory manually.
- **`.hpp` = interface, `.cpp` = implementation.** Always use include guards.
- **Functions that take streams should take `std::istream&` / `std::ostream&`** — never `ifstream` / `ofstream` directly. This is how the course's testable interfaces are structured.

Next guide: **02 — Classes and Operator Overloading** picks up from `Coins` and builds out the HW3 `String` class.
