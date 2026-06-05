# ICS 45C — Templates, Lambdas, Exceptions, Namespaces
> Covers the Topic 7 slide deck + HW7 (template `Array<T>` and template `Matrix<T>`)

These four features are smaller in surface area than the previous guides, but they show up everywhere from HW7 onward. Templates make HW7's `Array` work for any type; lambdas and exceptions show up in HW7's `fill_with_fn` and `out_of_range` throws; namespaces wrap helpers (we already saw `namespace list` in HW5).

---

## Table of Contents
1. [Lambdas: Anonymous Local Functions](#1-lambdas-anonymous-local-functions)
2. [Lambda Captures](#2-lambda-captures)
3. [Exceptions: `try`/`catch`/`throw`](#3-exceptions-trycatchthrow)
4. [The Standard Exception Hierarchy](#4-the-standard-exception-hierarchy)
5. [Templates: Function Templates](#5-templates-function-templates)
6. [Template Specialization and Overload Resolution](#6-template-specialization-and-overload-resolution)
7. [Class Templates](#7-class-templates)
8. [Defining Template Methods](#8-defining-template-methods)
9. [Template Parameter Defaults & Member Templates](#9-template-parameter-defaults--member-templates)
10. [Type Aliases: `using` and `typedef`](#10-type-aliases-using-and-typedef)
11. [Namespaces (Recap and Detail)](#11-namespaces-recap-and-detail)
12. [The HW7 `Array<T>` and `Matrix<T>` Templates](#12-the-hw7-arrayt-and-matrixt-templates)
13. [HW7 Pattern Quick Reference](#13-hw7-pattern-quick-reference)
14. [Key Rules to Remember](#14-key-rules-to-remember)

---

## 1. Lambdas: Anonymous Local Functions

A **lambda** is a function written inline, often passed straight to an algorithm. Three pieces in order: capture list `[]`, parameter list `()`, body `{}`.

```cpp
auto sq = [](int x) { return x * x; };               // a lambda that squares its argument
std::cout << sq(4);                                   // 16

// Pass directly to an algorithm:
std::vector<int> v = {1,2,3,4,5};
std::for_each(v.begin(), v.end(),
              [](int x) { std::cout << x << ' '; });  // prints 1 2 3 4 5
```

The compiler invents a class on the fly. `sq` is an object of that class with an `operator()(int x) const`. **`auto`** is the only practical way to name a lambda type, because each lambda has a unique compiler-generated type.

Optional return-type syntax (only needed when the compiler can't deduce it):

```cpp
auto divide = [](double a, double b) -> double { return a / b; };
//                                     ^^^^^^^ trailing return type
```

---

## 2. Lambda Captures

The capture list lets a lambda see variables from the enclosing scope.

| Capture | Meaning |
|---|---|
| `[]` | capture nothing |
| `[&]` | capture all referenced variables **by reference** |
| `[=]` | capture all referenced variables **by copy** |
| `[x]` | capture `x` by copy |
| `[&x]` | capture `x` by reference |
| `[=, &m]` | copy everything else, but `m` by reference |
| `[this]` | capture `this` so the lambda can call methods of the enclosing class |

```cpp
int total = 0;
std::vector<int> v = {1,2,3,4};

std::for_each(v.begin(), v.end(),
              [&total](int x) { total += x; });       // captures total by reference
// total == 10
```

The HW7 `Array<T>::fill_with_fn` and HW8 `compute_grades.cpp` both capture by reference to mutate state:

```cpp
// HW8 compute_grades — capture out by reference so the lambda can write to it
std::for_each(b.students.cbegin(), b.students.cend(),
              [&out](const Student& s) { out << s << "\n"; });
```

**Beware capturing references to soon-dead variables.** If the lambda outlives them, you have a dangling reference. With `[=]` the variables are copied into the lambda, so no dangling.

---

## 3. Exceptions: `try`/`catch`/`throw`

Exceptions are the non-local way to report errors. The flow:

```cpp
// 1) Where the error is detected — throw
throw std::domain_error("Error: invalid percentage " + std::to_string(score));

// 2) Where the error is handled — catch
try {
    s.validate();
} catch (const std::domain_error& e) {                // catch by const&
    std::cerr << e.what() << '\n';                    // e.what() returns the message
}
```

What happens at `throw`:
1. The thrown object is created (here, `std::domain_error`).
2. The runtime unwinds the call stack, destroying every automatic object on the way (destructors run — RAII matters!).
3. It stops when it finds a `catch` whose type matches.
4. If no handler matches, `std::terminate()` is called and the program aborts.

Three rules of thumb:
- **Catch by `const T&`.** Catching by value slices polymorphic exception types; catching by `T&` works but is unconventional.
- **Throw exceptions, don't return error codes**, but only for **exceptional** events — not for "user typed a wrong menu option." Use exceptions for programmer errors and contract violations.
- **Destructors must not throw.** During stack unwinding two exceptions in flight will terminate the program.

The HW7 `Array<T>::operator[]` throws on out-of-bounds:

```cpp
T& Array<T>::operator[](int index) {
    if (!in_bounds(index))
        throw std::out_of_range{"out of bounds"};
    return buf[index];
}
```

The HW8 `Student::validate()` throws on bad input:

```cpp
void Student::validate() const {
    auto check = [](int score) {
        if (score < 0 || score > 100)
            throw std::domain_error("Error: invalid percentage" + std::to_string(score));
    };
    std::for_each(quiz.cbegin(), quiz.cend(), check);
    std::for_each(hw.cbegin(),   hw.cend(),   check);
    check(static_cast<int>(final_score));
}
```

---

## 4. The Standard Exception Hierarchy

All exceptions in `<stdexcept>` derive from `std::exception`:

```
exception
├── logic_error           — error caused by a bug in the program
│   ├── domain_error      — value outside expected domain (e.g., bad %)
│   ├── invalid_argument  — bad argument (e.g., bitset init not 0/1)
│   ├── length_error      — operation exceeds size limit
│   └── out_of_range      — index out of valid range (.at() throws this)
└── runtime_error         — error caused by something external/runtime
    ├── range_error       — value out of computable range
    ├── overflow_error    — arithmetic overflow
    └── underflow_error   — arithmetic underflow
```

Two extras outside `<stdexcept>`:
- `std::bad_alloc` — thrown by failing `new`.
- `std::ios::failure` — I/O error (only thrown if enabled on a stream).

Pick the type that names the error. HW7 uses `out_of_range` for bad indexing; HW8 uses `domain_error` for "score must be 0–100".

Use `catch (const std::exception& e)` as a last resort to catch anything you missed, then call `e.what()` for the message.

---

## 5. Templates: Function Templates

A **template** generates code from a pattern at compile time. The classic `min`:

```cpp
template <typename Type>                              // typename = "Type is a placeholder type"
Type min(Type a, Type b) {
    return a < b ? a : b;
}

int main() {
    int    x = min(10, 20);                            // instantiates min<int>
    double y = min(10.5, 20.8);                        // instantiates min<double>
    auto   s = min(std::string{"hi"}, std::string{"yo"});
}
```

When the compiler sees `min(10, 20)` it deduces `Type = int`, generates a function body, and compiles it. Each distinct `Type` produces a distinct version.

What can be a template parameter?

| Form | Use |
|---|---|
| `typename T` (or `class T`, same meaning) | a stand-in **type** |
| `int N` (non-type) | a compile-time **value**, e.g. array sizes |
| `template <typename> class Container` | a template itself |

**What can the body do?** Whatever the compiler can compile for that `T`. If the template uses `a < b`, then `T` must support `<`. The compiler error you get when something is missing is famously long.

---

## 6. Template Specialization and Overload Resolution

For some types the general implementation is wrong or inefficient. You can **specialize**:

```cpp
template <typename T> T min(T a, T b) { return a < b ? a : b; }

// Specialization: for char*, compare strings, not pointers
char* min(char* a, char* b) {
    return std::strcmp(a, b) < 0 ? a : b;
}
```

When calling `min(...)`, the compiler:
1. Looks for matching **non-template** functions. Picks one if there's an unambiguous best fit.
2. Otherwise, looks for matching **template** instantiations. Picks one if unambiguous.
3. Otherwise tries type conversions on the non-templates.

So the specialized non-template wins for `char*`; the template handles everything else.

A `const` formal can bind to either `const` or non-`const` actuals. A non-`const` formal binds only to non-`const`. A `const T&` formal can bind to a literal — useful for `f("hello")`.

---

## 7. Class Templates

A class template parameterizes a whole class:

```cpp
template <typename T>
class Stack {
    int  top;
    int  len;
    T*   buf;
public:
    explicit Stack(int capacity = 100)
      : top(0), len(capacity), buf(new T[capacity]) {}
    ~Stack()         { delete[] buf; }
    void push(T x)   { buf[top++] = x; }
    T    pop()       { return buf[--top]; }
    int  size() const{ return top; }
};

Stack<int>    s1;                                     // a stack of ints
Stack<double> s2;                                     // a stack of doubles
Stack<std::string> s3;                                // a stack of strings
```

Inside the class body, you can use `T` like an ordinary type. Outside it (in the implementation), every method has to be re-templated and qualified — see §8.

---

## 8. Defining Template Methods

If you keep method bodies inside the class declaration (HW7 does), there's no extra ceremony. If you split into `.hpp` (declarations) and `.cpp` (definitions), every method definition gets the template prefix and `Stack<T>::` qualification:

```cpp
// stack.hpp
template <typename T>
class Stack {
public:
    explicit Stack(int capacity = 100);
    void push(T x);
};

// stack.cpp  ← but actually, you can't really split a template like this; the
//              definitions must be visible at the point of instantiation.
template <typename T>
Stack<T>::Stack(int capacity)
  : top(0), len(capacity), buf(new T[capacity]) {}

template <typename T>
void Stack<T>::push(T x) {
    buf[top++] = x;
}
```

**Practical takeaway**: put template definitions in the header (or in a separate file `.tpp`/`.ipp` that the header includes). HW7's `array.hpp` and `matrix.hpp` define everything inline in the header for this reason.

---

## 9. Template Parameter Defaults & Member Templates

Template parameters can have defaults:

```cpp
template <typename CharT = char>
class basic_string {
    /* ... */
};
basic_string<>      s1;                               // basic_string<char>
basic_string<wchar_t> s2;
```

A **member function** of a non-template class can itself be a template. You'll see this used in HW7 `Array<T>::fill_with_fn`:

```cpp
template <typename T>
class Array {
public:
    template <typename Fn>                            // member template
    void fill_with_fn(Fn fn) {
        for (int i = 0; i < len; ++i)
            buf[i] = fn(i);                            // fn could be a lambda, function ptr, ...
    }
};
```

`Fn` is deduced from the argument. A lambda of any signature compatible with `(int) → T` works:

```cpp
Array<double> a(10);
a.fill_with_fn([](int i) { return i * 0.5; });        // fill with 0, 0.5, 1.0, ...
```

This is how HW7's `Matrix` constructor builds an `Array<Array<T>>`:

```cpp
template <typename T>
Matrix<T>::Matrix(int rows, int cols)
  : rows{rows}, cols{cols}, data{rows} {
    data.fill_with_fn([cols](int){ return Array<T>{cols}; });  // each row = new Array<T>
}
```

---

## 10. Type Aliases: `using` and `typedef`

A type alias gives a shorter name to a long type. Two equivalent syntaxes:

```cpp
typedef int* IntPointer;                              // C-style (legacy)
using   IntPointer = int*;                            // modern, preferred

using intStack    = Stack<int>;                       // alias a template instantiation
using doubleStack = Stack<double>;
```

`using` is strictly more powerful than `typedef` (it can be templated itself: `template<typename T> using Vec = std::vector<T>;`), so use `using` everywhere in new code.

---

## 11. Namespaces (Recap and Detail)

We covered namespaces in Guide 04 for the `list::` helpers. Three additional points:

- Namespaces are **cumulative**: a namespace defined in two files combines into one logical namespace.
- The standard library lives in `namespace std`. Everything: `std::cout`, `std::vector`, `std::ranges::sort`.
- **Avoid `using namespace std;` in headers.** It pulls the entire standard library into every file that includes you — name conflicts are likely.

```cpp
namespace list {
    Node* copy(Node* head);                            // declared in list.hpp
}
// list.cpp
namespace list {
    Node* copy(Node* head) { /* ... */ }               // definition adds to same namespace
}
```

Accessing names: `list::copy(head)` (qualified), `using list::copy; copy(head);` (single name), `using namespace list;` (the whole namespace, usually only inside a function).

---

## 12. The HW7 `Array<T>` and `Matrix<T>` Templates

HW7 combines everything: templates, copy-and-swap, friend functions, lambdas, exceptions, and template I/O operators.

### `Array<T>` — full skeleton with selected bodies

```cpp
#include <iomanip>
#include <iostream>
#include <sstream>
#include <utility>

template <typename T>
class Array {
public:
    Array()                : len{0}, buf{nullptr} {}
    explicit Array(int n)  : len{n}, buf{new T[n]} {}

    Array(const Array& other)                          // copy ctor — deep copy
      : len{other.len}, buf{new T[other.len]} {
        for (int i = 0; i < len; ++i) buf[i] = other.buf[i];
    }

    Array(Array&& other) noexcept                      // move ctor — steal via swap
      : len{0}, buf{nullptr} {
        swap(*this, other);
    }

    friend void swap(Array& a, Array& b) noexcept {    // friend swap — see below
        std::swap(a.len, b.len);
        std::swap(a.buf, b.buf);
    }

    Array& operator=(const Array& other) {             // copy-and-swap
        Array temp{other};
        swap(*this, temp);
        return *this;
    }
    Array& operator=(Array&& other) noexcept {         // move assign
        swap(*this, other);
        return *this;
    }

    ~Array() { delete[] buf; }

    int length() const { return len; }

    T& operator[](int i) {                              // throws on bad index
        if (!in_bounds(i)) throw std::out_of_range{"out of bounds"};
        return buf[i];
    }
    const T& operator[](int i) const {
        if (!in_bounds(i)) throw std::out_of_range{"out of bounds"};
        return buf[i];
    }

    void fill(const T& v) {                             // assign one value to all slots
        for (int i = 0; i < len; ++i) buf[i] = v;
    }

    template <typename Fn>
    void fill_with_fn(Fn fn) {                          // call fn(i) for each slot
        for (int i = 0; i < len; ++i) buf[i] = fn(i);
    }

private:
    int len;
    T*  buf;
    bool in_bounds(int i) const { return i >= 0 && i < len; }
};
```

A few language details worth pausing on:

| Feature | Meaning |
|---|---|
| `noexcept` on move ctor | promises this function won't throw — lets containers optimize and required for some library code |
| `friend void swap(Array&, Array&)` defined inside the class | a **free** function with access to private members; argument-dependent lookup finds it |
| Two `operator[]` overloads | one returns `T&` (for non-`const` Arrays), one returns `const T&` (for `const` Arrays) |
| Member-template `fill_with_fn` | accepts any callable, including lambdas |

### Stream operators for templates

Non-member `operator<<` and `operator>>` must themselves be templates:

```cpp
template <typename T>
std::ostream& operator<<(std::ostream& out, const Array<T>& a) {
    std::stringstream temp;
    temp << std::setprecision(2) << std::fixed << std::right;
    for (int i = 0; i < a.length(); ++i)
        temp << std::setw(8) << a[i];                  // calls Array<T>::operator[] const
    out << temp.str();
    return out;
}

template <typename T>
std::istream& operator>>(std::istream& in, Array<T>& a) {
    for (int i = 0; i < a.length(); ++i)
        in >> a[i];
    return in;
}
```

`std::setw(8)` from `<iomanip>` sets the next field's width to 8 characters; `std::setprecision(2) << std::fixed` formats doubles with two decimal places. The buffering through a `stringstream` (also `<sstream>`) ensures the manipulators apply uniformly to every element.

### `Matrix<T>` is `Array<Array<T>>`

```cpp
template <typename T>
class Matrix {
public:
    Matrix() : rows{0}, cols{0}, data{} {}

    Matrix(int rows, int cols)
      : rows{rows}, cols{cols},
        data{rows} {                                    // outer Array<Array<T>> of size rows
        data.fill_with_fn([cols](int){ return Array<T>{cols}; });
    }                                                    //  each inner Array has size cols

    Array<T>&       operator[](int r)       { return data[r]; }
    const Array<T>& operator[](int r) const { return data[r]; }

    int num_rows() const { return rows; }
    int num_cols() const { return cols; }

    void fill(const T& v) {
        for (int i = 0; i < rows; ++i) data[i].fill(v);
    }
    template <typename Fn>
    void fill_with_fn(Fn fn) {
        for (int i = 0; i < rows; ++i)
            for (int j = 0; j < cols; ++j) data[i][j] = fn(i, j);
    }

private:
    int             rows, cols;
    Array<Array<T>> data;                              // composition wins
};
```

A `Matrix<T>` follows the **Rule of Zero** (Guide 03): its only member is an `Array<Array<T>>`, whose copy/move/destruction is already correct. No special members needed.

### Matrix-vector multiplication is a free function template

```cpp
template <typename T>
Array<T> operator*(const Matrix<T>& mat, const Array<T>& arr) {
    if (mat.num_cols() != arr.length())
        throw std::string{"Dimensions do not line up for product!"};

    Array<T> result{mat.num_rows()};
    result.fill(0);
    for (int i = 0; i < mat.num_rows(); ++i)
        for (int j = 0; j < mat.num_cols(); ++j)
            result[i] += mat[i][j] * arr[j];
    return result;
}
```

(`throw std::string{}` is unusual — `std::runtime_error` would be more conventional. The HW7 reference does it this way.)

---

## 13. HW7 Pattern Quick Reference

### Copy-and-swap for a template

```cpp
template <typename T>
class Array {
public:
    Array(const Array& other)                          // 1. deep copy
      : len{other.len}, buf{new T[other.len]} {
        for (int i = 0; i < len; ++i) buf[i] = other.buf[i];
    }

    Array(Array&& other) noexcept : len{0}, buf{nullptr} {
        swap(*this, other);                            // 2. move = swap with empty
    }

    friend void swap(Array& a, Array& b) noexcept {    // 3. one swap rules them all
        std::swap(a.len, b.len);
        std::swap(a.buf, b.buf);
    }

    Array& operator=(const Array& other) {             // 4. by value would also work
        Array temp{other};
        swap(*this, temp);
        return *this;
    }
    Array& operator=(Array&& other) noexcept {
        swap(*this, other);
        return *this;
    }

    ~Array() { delete[] buf; }                         // 5. release
};
```

### Lambda → `fill_with_fn` → Matrix initialization

```cpp
Array<int> a(5);
a.fill_with_fn([](int i){ return i * i; });            // {0, 1, 4, 9, 16}

Matrix<double> rot(2, 2);                              // rotation matrix
double angle = 0.3;
rot.fill_with_fn([&](int i, int j) {
    if (i == 0 && j == 0)      return std::cos(angle);
    else if (i == 0 && j == 1) return -std::sin(angle);
    else if (i == 1 && j == 0) return std::sin(angle);
    else                       return std::cos(angle);
});
```

### Exception-aware indexing

```cpp
try {
    Array<int> a(5);
    a[10] = 0;                                          // out of bounds
} catch (const std::out_of_range& e) {
    std::cerr << e.what();                             // "out of bounds"
}
```

### Compose existing template types — Rule of Zero

```cpp
template <typename T>
class Matrix {
    Array<Array<T>> data;                              // delegates everything
};                                                      // no special members needed
```

---

## 14. Key Rules to Remember

- A **lambda** is `[captures](params) { body }`. Use `auto` to name it.
- **Capture by reference (`[&]`) only when the lambda outlives its references**, otherwise capture by copy (`[=]`).
- Inside a method, `[this]` captures the enclosing object pointer.
- **Throw exceptions for exceptional events**, not control flow. Common types: `std::out_of_range`, `std::domain_error`, `std::invalid_argument`.
- **Catch by `const T&`.** Print `e.what()` for the message.
- **Destructors must not throw.** RAII ensures they run during unwinding — they cannot throw a *second* exception.
- A **function template** writes one body and the compiler generates one per used type. `template <typename T>` introduces a type parameter.
- A **class template** parameterizes a whole class. `Stack<int>`, `Stack<std::string>` are distinct types.
- Template definitions usually live in the header (or in a `.tpp` included from the header). Putting bodies in `.cpp` and forgetting explicit instantiation is the classic mistake.
- **Member templates** (a templated method inside a non-template class) accept any callable — perfect for `fill_with_fn`.
- **`noexcept`** on move ctor/assignment / `swap` is more than documentation: the standard library uses it to choose between copy and move.
- **`friend void swap(T&, T&)`** defined in-class is the idiomatic pattern — ADL finds it, and it can touch private members.
- **Stream operators for templates are themselves templates**: `template <typename T> std::ostream& operator<<(std::ostream&, const Array<T>&)`.
- **Rule of Zero** wins whenever you can hold a standard container; HW7 `Matrix` does this with `Array<Array<T>>`.
- `using` over `typedef` everywhere in new code.
- **Namespaces are cumulative** across files. Don't `using namespace std;` in headers.

Next: **Guide 07 — The STL** covers HW8. After that, Guide 08 picks up custom containers, smart pointers, and ranges.
