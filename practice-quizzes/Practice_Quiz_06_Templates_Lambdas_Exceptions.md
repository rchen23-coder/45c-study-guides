# Practice Quiz 06 — Templates, Lambdas, Exceptions, Namespaces

**Covers:** Guide 06 (Topic 7 slides — lambdas + captures, exceptions + std hierarchy, function & class templates, specialization, member templates, type aliases, namespaces).

**Questions:** 15

---

## Questions

### Question 1
What is the type of `f` here?
```cpp
auto f = [](int x, int y) { return x * y; };
```
- A. `int(int, int)`
- B. `std::function<int(int,int)>`
- C. A unique unnamed compiler-generated class type (a closure type).
- D. `int*`
- E. `int` (the result type)

### Question 2
SELECT ALL true about lambda captures.
- A. `[=]` captures all referenced enclosing variables by value.
- B. `[&]` captures all referenced enclosing variables by reference.
- C. `[x]` captures just `x` by value.
- D. `[&x]` captures just `x` by reference.
- E. `[this]` is required to call methods on the enclosing object inside the lambda.

### Question 3
What does this print?
```cpp
#include <iostream>
int main() {
    int x = 5;
    auto f = [x]() mutable { x += 1; return x; };
    std::cout << f() << ' ' << f() << ' ' << x;
}
```
- A. `6 6 5`
- B. `6 7 5`
- C. `6 7 7`
- D. `6 6 6`
- E. none of the other choices

### Question 4
SELECT ALL exceptions that derive from `std::logic_error`.
- A. `std::out_of_range`
- B. `std::domain_error`
- C. `std::runtime_error`
- D. `std::invalid_argument`
- E. `std::bad_alloc`

### Question 5
What is the output?
```cpp
#include <iostream>
#include <stdexcept>
int divide(int a, int b) {
    if (b == 0) throw std::invalid_argument("zero");
    return a / b;
}
int main() {
    try {
        int x = divide(10, 0);
        std::cout << x;
    } catch (const std::invalid_argument& e) {
        std::cout << "caught: " << e.what();
    } catch (...) {
        std::cout << "other";
    }
}
```
- A. `0`
- B. `caught: zero`
- C. `other`
- D. nothing (program terminates)
- E. compile error

### Question 6
What does this print?
```cpp
template <typename T>
T min(T a, T b) { return a < b ? a : b; }

int main() {
    std::cout << min(3, 4) << ' ' << min(3.5, 2.5);
}
```
- A. `3 2.5`
- B. `4 3.5`
- C. compile error — `min` ambiguous
- D. `3 2`
- E. none of the other choices

### Question 7
SELECT ALL true about a function declared `void f() noexcept`.
- A. `f` promises not to throw any exception visible to the caller.
- B. If an exception escapes `f`, `std::terminate` is called.
- C. `noexcept` prevents `f` from throwing internally.
- D. Moving from `f`'s return value can be optimized by some standard containers.
- E. `noexcept` may appear before the function name as in `void noexcept f();`.

### Question 8
What does this print?
```cpp
template <typename T>
class Box {
    T val;
public:
    explicit Box(T v) : val(v) {}
    T get() const { return val; }
};
int main() {
    Box<int> a{42};
    Box<std::string> b{"hi"};
    std::cout << a.get() << ' ' << b.get();
}
```
- A. `42 hi`
- B. compile error — Box can't hold both int and string
- C. `0 hi`
- D. `42` followed by an empty string
- E. none of the other choices

### Question 9
SELECT ALL true about a **member template** like:
```cpp
template <typename T>
class Array {
public:
    template <typename Fn>
    void fill_with_fn(Fn fn);
};
```
- A. `Fn` is deduced from the argument passed at the call site.
- B. The same `Array<int>` can be filled with different lambdas at different calls.
- C. `Fn` must be a function pointer; lambdas don't work.
- D. The member template is itself a class — you instantiate it before use.
- E. `fill_with_fn` could accept any callable: lambda, function pointer, or `std::function`.

### Question 10
What does this print?
```cpp
#include <iostream>
template <typename T>
T add(T a, T b) { return a + b; }
template <>
const char* add(const char* a, const char* b) { return "specialized"; }

int main() {
    std::cout << add(1, 2) << ' ' << add("X", "Y");
}
```
- A. `3 specialized`
- B. `3 XY`
- C. `12 specialized`
- D. compile error
- E. none of the other choices

### Question 11
SELECT ALL true about C++ `namespace`s.
- A. The same namespace can be reopened in multiple files.
- B. `using namespace std;` is recommended at the top of headers for convenience.
- C. Nested namespaces (`namespace a::b { ... }`) are allowed.
- D. Members of a namespace can be referenced qualified (`a::x`) or via `using a::x;`.
- E. The standard library lives in `namespace std`.

### Question 12
What is the output?
```cpp
#include <iostream>
int main() {
    int total = 0;
    auto add = [&total](int x) { total += x; };
    for (int i = 1; i <= 4; ++i) add(i);
    std::cout << total;
}
```
- A. 0
- B. 4
- C. 10
- D. 24
- E. compile error

### Question 13
SELECT ALL true about exception handling.
- A. `catch (const std::exception& e)` will catch any exception derived from `std::exception`.
- B. Destructors should not throw — a second exception during unwinding calls `std::terminate`.
- C. If no `catch` matches a thrown exception, the program terminates.
- D. Throwing a `const char*` is equivalent to throwing a `std::string`.
- E. RAII-managed resources are released when an exception propagates through their scope.

### Question 14
What is the result?
```cpp
#include <iostream>
#include <stdexcept>
template <typename T>
T& at_or_throw(T* arr, int n, int i) {
    if (i < 0 || i >= n) throw std::out_of_range("bad");
    return arr[i];
}
int main() {
    int A[3] = {10, 20, 30};
    try { std::cout << at_or_throw(A, 3, 5); }
    catch (const std::exception& e) { std::cout << e.what(); }
}
```
- A. `10`
- B. `30`
- C. `bad`
- D. compile error — template can't throw
- E. none of the other choices

### Question 15
SELECT ALL true about `using` and `typedef`.
- A. `using NewName = OldName;` is the modern form.
- B. `typedef OldName NewName;` is the C-style form, still legal.
- C. Only `using` can take template parameters (`template <typename T> using Vec = std::vector<T>;`).
- D. `typedef` creates a new distinct type that is not interchangeable with the original.
- E. `using` and `typedef` make the new name an alias of the original type.

---

## Answer Key

1. **C** — Each lambda has a unique closure type the compiler invents.
2. **A, B, C, D, E** — All five are accurate; `[this]` is needed to call methods of the enclosing object.
3. **B** — `mutable` lets the lambda modify its by-value copy of `x`. First call: 6. Second call: the closure's `x` is now 6, so it returns 7. The enclosing `x` is untouched (still 5). Output: `6 7 5`.
4. **A, B, D** — `logic_error` subtree: `out_of_range`, `domain_error`, `invalid_argument`, `length_error`. (`runtime_error` is the sibling subtree. `bad_alloc` derives directly from `exception`, not `logic_error`.)
5. **B** — `divide(10, 0)` throws; first matching `catch` prints `caught: zero`.
6. **A** — Template instantiated as `min<int>` and `min<double>`; returns the smaller of each pair.
7. **A, B, D** — `noexcept` promises not to throw; escape triggers `terminate`; std lib uses it to choose move over copy. (C: `noexcept` doesn't *prevent* throwing — it promises. E: syntax is `void f() noexcept;`, not `void noexcept f();`.)
8. **A** — `Box<int>` and `Box<std::string>` are distinct instantiations; each stores and returns its own value.
9. **A, B, E** — Member templates deduce, can be called with different callables, and accept any callable. (C is false. D confuses class templates with member function templates.)
10. **A** — `add(1,2)` instantiates `add<int>` → 3. `add("X","Y")` matches the explicit specialization for `const char*` → `"specialized"`.
11. **A, C, D, E** — Standard namespace facts. (B: `using namespace std;` in a header pollutes every including file and is widely discouraged.)
12. **C** — Lambda captures `total` by reference; sums 1+2+3+4 = 10.
13. **A, B, C, E** — Standard exception facts. (D: types differ — catching `const char*&` won't catch `std::string` and vice versa.)
14. **C** — Index 5 is out of bounds; throws `out_of_range`; caught as base `exception&`; prints `bad`.
15. **A, B, C, E** — Both create aliases; only `using` can be templated. (D is false: `typedef` does NOT create a distinct type — it's also an alias.)
