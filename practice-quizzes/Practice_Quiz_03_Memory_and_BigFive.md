# Practice Quiz 03 — Memory Management and the Big Five

**Covers:** Guide 03 (Dyn Array slides — `new`/`delete`, pointers/references, Rule of Three/Five/Zero, copy-and-swap, move semantics, RAII, the HW4 `AllocationTracker`).

**Questions:** 15

---

## Questions

### Question 1
SELECT ALL true about `new` and `delete`.
- A. `new T[n]` returns a `T*` pointing at the first element.
- B. `delete p` is safe when `p == nullptr`.
- C. `delete p` is correct for memory acquired with `new T[n]`.
- D. `new T` allocates from the stack.
- E. If `new` cannot allocate, it throws `std::bad_alloc`.

### Question 2
Which line in `main` is a memory leak?
```cpp
int main() {
    int* a = new int{1};      // line A
    int* b = new int[5];      // line B
    int* c = a;               // line C
    a = nullptr;              // line D
    delete[] b;               // line E
}
```
- A. Line A
- B. The leak happens because line C makes `c` point at `a`'s memory and then line D drops `a`.
- C. The leak happens because the allocation from line A is never `delete`d.
- D. Line E
- E. There is no leak in this program.

### Question 3
What is the output?
```cpp
#include <iostream>
class R {
    int* p;
public:
    R(int v)            : p(new int{v}) { std::cout << "C"; }
    R(const R& o)       : p(new int{*o.p}) { std::cout << "K"; }
    ~R()                { delete p;  std::cout << "D"; }
};
int main() {
    R x(1);
    R y = x;
}
```
- A. `CKDD`
- B. `CCDD`
- C. `CKD`
- D. `CCKDD`
- E. none of the other choices

### Question 4
SELECT ALL true about the Rule of Three / Five.
- A. A class that manages a raw pointer must define copy ctor, copy assignment, and destructor.
- B. If you define a destructor, the compiler will NOT silently generate a move ctor.
- C. The Rule of Zero says you should always disable copy operations on resource-holding classes.
- D. Marking a move ctor `noexcept` enables certain standard-library optimizations.
- E. Defining a copy ctor disables the default destructor.

### Question 5
What is the most likely consequence of this code?
```cpp
class Buf {
    char* p;
public:
    Buf(int n) : p(new char[n]) {}
    ~Buf() { delete[] p; }
};
int main() {
    Buf a(100);
    Buf b = a;
}
```
- A. The compiler refuses to compile.
- B. The program runs cleanly.
- C. Double-`delete[]` of the same memory — undefined behavior (often a crash) at end of `main`.
- D. A memory leak.
- E. `b.p` ends up `nullptr`.

### Question 6
Which line completes a correct **move constructor** for the class below?
```cpp
class String {
    char* buf;
public:
    String(String&& s) { /* ??? */ }
    ~String() { delete[] buf; }
};
```
- A. `buf = new char[strlen(s.buf) + 1]; strcpy(buf, s.buf);`
- B. `buf = s.buf;`
- C. `buf = s.buf; s.buf = nullptr;`
- D. `delete[] buf; buf = s.buf; s.buf = nullptr;`
- E. `swap(*this, s);`

### Question 7
SELECT ALL statements that are true about the copy-and-swap idiom:
```cpp
String& String::operator=(String s) { swap(s); return *this; }
```
- A. The parameter `s` is a copy of the rhs, made via the copy ctor (or move ctor if the rhs is an rvalue).
- B. Self-assignment (`a = a`) is naturally safe.
- C. It requires the class to define both copy and move ctors separately.
- D. Exception safety: if construction of `s` throws, `*this` is untouched.
- E. The same `operator=` handles both copy and move assignment.

### Question 8
What does this print?
```cpp
#include <iostream>
class A {
    int* p;
public:
    A(int v) : p(new int{v}) {}
    A(A&& o) noexcept : p(o.p) { o.p = nullptr; }
    ~A() { delete p; std::cout << '-'; }
};
A make() { return A(42); }
int main() {
    A a = make();
    std::cout << '|';
}
```
- A. `-|-`
- B. `|-`
- C. `-|`
- D. `||`
- E. none of the other choices

### Question 9
What happens after `std::move(x)` is applied to a `String x`?
- A. `x` is destroyed immediately.
- B. `x` is cast to an rvalue reference; nothing else happens by itself.
- C. The compiler refuses to use `x` again until reassignment.
- D. The move constructor of `String` is automatically called on `x`.
- E. `x` is set to `nullptr`.

### Question 10
SELECT ALL true about RAII (Resource Acquisition Is Initialization).
- A. The destructor is the right place to release the resource.
- B. Stack-allocated objects automatically run their destructors when scope ends.
- C. If a function throws, RAII-managed objects in the stack frame are still cleaned up.
- D. RAII requires the resource to be heap memory.
- E. The Rule of Zero is a way to apply RAII by composing standard-library types.

### Question 11
Which of the following is the **only** correct way to release memory acquired with `new T[n]`?
- A. `delete p;`
- B. `free(p);`
- C. `delete[] p;`
- D. `delete &p;`
- E. `p = nullptr;`

### Question 12
What does this print?
```cpp
#include <iostream>
class V {
public:
    V()             { std::cout << "D"; }
    V(const V&)     { std::cout << "C"; }
    V(V&&) noexcept { std::cout << "M"; }
    ~V()            { std::cout << "X"; }
};
int main() {
    V a;
    V b = std::move(a);
}
```
- A. `DCXX`
- B. `DMXX`
- C. `DCMXX`
- D. `DMX`
- E. none of the other choices

### Question 13
The HW4 `AllocationTracker` reports a "double delete." Which most likely caused it?
- A. A leaking `new` that was never matched with `delete`.
- B. A move ctor that copied the pointer but forgot to null out the source.
- C. A class that uses only `std::string` and `std::vector` (Rule of Zero).
- D. A destructor that runs twice on the same object.
- E. Using `new[]` paired with `delete` (without `[]`).

### Question 14
SELECT ALL ways an `AllocationTracker` can detect a bug.
- A. Reports a leak when `delete[] buf` is missing from a destructor.
- B. Reports a double delete when two objects share the same pointer.
- C. Reports a mismatching delete when `new` is paired with `delete[]` or vice versa.
- D. Reports an exception when `new` throws `std::bad_alloc`.
- E. Reports the function name where the bad delete occurred.

### Question 15
Why is a **private** size constructor like `String(int length)` useful in HW4?
- A. To prevent users from constructing zero-length strings.
- B. So `reverse()` and `operator+` can allocate the right-sized buffer once, without going through `strdup`.
- C. To allow `String` to be constructed from another String's `buf` pointer.
- D. To enforce that all strings are the same length.
- E. To guarantee exception safety in the destructor.

---

## Answer Key

1. **A, B, E** — `new T[n]` yields a `T*`; `delete nullptr` is a no-op; failed `new` throws `bad_alloc`. (C is wrong: must use `delete[]` for `new[]`. D: `new` allocates from the free store / heap, not the stack.)
2. **C** — `a` is reassigned to `nullptr` at line D before being deleted; the original heap int is unreachable.
3. **A** — `R x(1)` → `C`; `R y = x` → `K` (copy ctor); both destruct at end → `DD`. Total: `CKDD`.
4. **A, B, D** — These are core Rule-of-Three/Five rules and the `noexcept` move optimization. (C: Rule of Zero means "don't write any of the five" — not "disable copy". E: copy ctor doesn't affect dtor generation.)
5. **C** — Default copy ctor copies the pointer, not the data. Both `Buf` destructors run, each calls `delete[]` on the same memory.
6. **C** — Move ctor steals the pointer **and** nulls the source so its destructor won't double-free. (D adds a needless `delete[] buf` — at construction `buf` is uninitialized garbage. A is a copy, not a move.)
7. **A, B, D, E** — One `operator=` taking by value handles both copy and move (the parameter is constructed via copy or move ctor, depending on rhs). Self-assignment safe because the copy happens first. (C: you DON'T need a separate move-assignment; the by-value parameter covers it.)
8. **B** — `make()` returns an `A`; RVO / move-construct into `a`; `cout << '|'` prints `|`; at scope end `~A()` prints `-`. Output: `|-`.
9. **B** — `std::move` is just an unconditional cast to rvalue reference. The move constructor isn't called until that rvalue is *used*.
10. **A, B, C, E** — The destructor releases; scope-exit dtors are automatic; RAII handles exceptions correctly via stack unwinding; Rule of Zero composes RAII-correct types. (D: RAII applies to ANY resource — files, locks, etc.)
11. **C** — Must match the allocator: `new[]` ↔ `delete[]`.
12. **B** — `V a` → `D`; `V b = std::move(a)` selects the move ctor → `M`; both destruct → `XX`. Output: `DMXX`.
13. **B** — A move ctor that doesn't null the source leaves two objects "owning" the same memory; both destructors `delete` it.
14. **A, B, C** — These are the three categories the tracker reports. (D: `bad_alloc` is not what the tracker observes; it observes `new`/`delete` calls. E: the tracker reports counts and totals, not call sites.)
15. **B** — Allowing `reverse()`/`operator+` to allocate once at the correct size avoids the extra round-trip through `strdup` (which would allocate, copy, then be replaced).
