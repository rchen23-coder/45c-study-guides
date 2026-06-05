# Practice Quiz 02 — Classes and Operator Overloading

**Covers:** Guide 02 (Classes and Strings slides — classes vs structs, ctor/dtor, `this`, init lists, `const`, operator overloading, `<<` / `>>`, static C-string helpers, TDD methodology).

**Questions:** 15

---

## Questions

### Question 1
What does this print?
```cpp
#include <iostream>
class Coins {
    int q, d, n, p;
public:
    Coins(int q = 0, int d = 0, int n = 0, int p = 0)
      : q(q), d(d), n(n), p(p) {}
    int total() const { return q*25 + d*10 + n*5 + p; }
};
int main() {
    Coins c(2, 1, 0, 3);
    std::cout << c.total();
}
```
- A. 6
- B. 63
- C. 50
- D. compile error — `q(q)` is ambiguous
- E. none of the other choices

### Question 2
SELECT ALL true about the operator overload pattern:
```cpp
std::ostream& operator<<(std::ostream& out, const T& obj);
```
- A. It must be a non-member function.
- B. It must be a `const` member function.
- C. Returning `out` lets `cout << a << b` chain.
- D. Passing `obj` by `const&` avoids copying.
- E. `out` is taken by reference so we share the caller's stream.

### Question 3
Which constructor below is the **copy constructor** for `class String`?
- A. `String();`
- B. `String(const char* s);`
- C. `String(const String& other);`
- D. `String(String&& other);`
- E. `String& operator=(const String& other);`

### Question 4
What does this print?
```cpp
struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {}
};
int main() {
    Point p{3, 4};
    p.x = 10;
    std::cout << p.x << ' ' << p.y;
}
```
- A. compile error — `Point` data members are private in a `struct`
- B. `3 4`
- C. `10 4`
- D. `10 0`
- E. none of the other choices

### Question 5
SELECT ALL statements that are true about `const` member functions.
- A. They may not modify any non-`mutable` data member of `*this`.
- B. They may be called on `const` objects.
- C. They may call other non-`const` member functions of `*this`.
- D. They may be called on non-`const` objects.
- E. They are declared by writing `const` after the parameter list: `int size() const;`.

### Question 6
What is the output?
```cpp
class Counter {
    int n = 0;
public:
    Counter& operator++() { ++n; return *this; }
    int value() const { return n; }
};
int main() {
    Counter c;
    ++(++c);
    std::cout << c.value();
}
```
- A. 0
- B. 1
- C. 2
- D. 3
- E. compile error

### Question 7
Which of the following operators **must** be implemented as a member function (not a free function)?
- A. `operator+`
- B. `operator==`
- C. `operator<<`
- D. `operator=`
- E. `operator!=`

### Question 8
SELECT ALL true about the following class:
```cpp
class Coins {
public:
    Coins(int q, int d, int n, int p);
    bool operator==(const Coins& other) const = default;
    int total() const;
};
```
- A. The compiler will generate member-wise equality for `operator==`.
- B. `Coins(0,0,0,0) == Coins(0,0,0,0)` evaluates to `true`.
- C. `operator==` is allowed to modify `*this` because it is `const`.
- D. `total()` cannot be called on a `const Coins` object.
- E. Two `Coins` with the same `total()` are guaranteed equal under the defaulted `==`.

### Question 9
Consider this `String` skeleton with a fixed buffer:
```cpp
constexpr int MAXLEN = 1024;
class String {
    char buf[MAXLEN];
public:
    explicit String(const char* s = "") { /* copy s into buf */ }
    char& operator[](int i);
};
```
What is the return type of `operator[]` and why?
- A. `char` — so `s[i]` can be modified.
- B. `char&` — so `s[i] = 'A';` works on the left side.
- C. `const char&` — to forbid modification.
- D. `char*` — pointer arithmetic.
- E. `void` — operators have no return.

### Question 10
What is the output?
```cpp
#include <iostream>
class C {
public:
    C(int)      { std::cout << "A"; }
    C(const C&) { std::cout << "B"; }
    ~C()        { std::cout << "C"; }
};
int main() {
    C x(1);
    C y = x;
}
```
- A. `ABCC`
- B. `AABBCC`
- C. `ABACC`
- D. `AAACC`
- E. none of the other choices

### Question 11
SELECT ALL true about non-member `operator+` for a `String` class:
```cpp
String operator+(const String& a, const String& b);
```
- A. Both operands may be string literals.
- B. The result is a new `String` returned by value.
- C. Returning by reference (`String&`) would be safe here.
- D. Implementing `+` does NOT automatically give you `+=`.
- E. Marking `operator+` as `const` is required.

### Question 12
What does this print?
```cpp
struct Person {
    std::string name;
    void greet() { std::cout << "Hi " << name; }
};
int main() {
    const Person p{"Sam"};
    p.greet();
}
```
- A. `Hi Sam`
- B. `Hi `
- C. compile error — `greet()` is not `const`
- D. compile error — `Person` has no constructor
- E. none of the other choices

### Question 13
Why might a static helper like `String::strncpy` be declared `static` rather than as a regular method?
- A. So it can be called as `String::strncpy(...)` without an instance.
- B. So it can be marked `const` and avoid `this`.
- C. Because the helper does not access any non-static members of the class.
- D. To make it run faster at compile time.
- E. So the compiler can inline it without needing the class definition.

### Question 14
SELECT ALL statements about TDD as taught in the Strings slides:
- A. Start by writing a nominal "happy path" test case.
- B. Identify a small helper function the public method can use.
- C. Write all public methods first, then tests at the end.
- D. Commit after each small green cycle.
- E. Public methods do error-checking; helpers stay general-purpose.

### Question 15
What does this print?
```cpp
#include <iostream>
class Box {
    int x;
public:
    explicit Box(int x) : x(x) { std::cout << "B"; }
    Box(const Box& o) : x(o.x) { std::cout << "C"; }
};
Box make() { return Box(7); }
int main() {
    Box a = make();
    std::cout << '|';
}
```
- A. `B|`
- B. `BC|`
- C. `BCC|`
- D. `CC|`
- E. depends on compiler (mandatory or non-mandatory copy elision)

---

## Answer Key

1. **B** — `2*25 + 1*10 + 0*5 + 3*1 = 50 + 10 + 0 + 3 = 63`. (The init-list `q(q)` is fine — the compiler resolves member-name vs parameter-name.)
2. **A, C, D, E** — `operator<<` cannot be a member of `ostream` from outside, so it's a non-member free function. (B is false: it isn't a member at all, so `const` doesn't apply.)
3. **C** — Copy ctor takes `const T&`. (D is move ctor; E is copy assignment.)
4. **C** — `struct` data is public by default, so `p.x = 10` is allowed. Print: `10 4`.
5. **A, B, D, E** — `const` methods can be called on either `const` or non-`const` objects. (C is false: a `const` method may NOT call non-`const` members of `*this`.)
6. **C** — Each `++` adds 1; the inner `++c` returns `c` by reference, then outer `++` adds another. `n` ends at 2.
7. **D** — `operator=`, `operator[]`, `operator()`, `operator->` MUST be member functions. The rest may be either.
8. **A, B** — `= default` generates member-wise equality, and identical inputs compare equal. (C: `const` forbids modification. D: `const` methods CAN be called on `const` objects. E: equality depends on each member, not just `total()`.)
9. **B** — Returning `char&` makes `s[i] = 'X';` legal; the index can sit on the left of `=`.
10. **A** — `C x(1);` prints `A`. `C y = x;` calls the copy ctor → `B`. Both destruct at scope end → `CC`. Total: `ABCC`.
11. **A, B, D** — `const&` binds to literals; result must be a new object (you can't return a reference to a local). `+` and `+=` are independent. (C: returning a reference would dangle. E: `const` on a non-member doesn't apply.)
12. **C** — `greet()` is non-`const` so it cannot be called on `const Person p`.
13. **A, C** — `static` means callable without an instance and not bound to `this`. (B: `const` is a non-`static` member specifier. D / E are not the reason.)
14. **A, B, D, E** — All four are the TDD discipline from the Strings slides. (C is the opposite of TDD.)
15. **A** — Mandatory copy elision since C++17 (a prvalue from `make()` initializes `a` directly). Just one `B` is printed. (Pre-C++17 you might see extra `C`s.)
