# Practice Quiz 05 — Inheritance and Polymorphism

**Covers:** Guide 05 (Topic 6 slides — is-a vs has-a, public/protected/private inheritance, static vs dynamic binding, `virtual`/`override`, pure virtual & abstract bases, virtual destructors, `clone()`, multiple inheritance).

**Questions:** 15

---

## Questions

### Question 1
Which derivation expresses "a Dog is-a Animal"?
- A. `class Dog { Animal a; };`
- B. `class Dog : private Animal { };`
- C. `class Dog : public Animal { };`
- D. `class Dog : protected Animal { };`
- E. `class Dog { Animal* a; };`

### Question 2
What does this print?
```cpp
struct A { virtual void f() { std::cout << "A"; } };
struct B : public A { void f() override { std::cout << "B"; } };
int main() {
    A* p = new B;
    p->f();
    delete p;
}
```
- A. `A`
- B. `B`
- C. `AB`
- D. compile error — missing `virtual` on `B::f`
- E. none of the other choices

### Question 3
What does this print?
```cpp
struct A { void f() { std::cout << "A"; } };  // NOT virtual
struct B : public A { void f() { std::cout << "B"; } };
int main() {
    A* p = new B;
    p->f();
    delete p;
}
```
- A. `A`
- B. `B`
- C. `AB`
- D. compile error
- E. none of the other choices

### Question 4
What is the **most likely** problem with this program?
```cpp
struct Base { };                            // no virtual destructor
struct Derived : public Base {
    char* buf;
    Derived() : buf(new char[100]) {}
    ~Derived() { delete[] buf; }
};
int main() {
    Base* p = new Derived;
    delete p;
}
```
- A. The destructor of `Derived` does not run — `buf` is leaked.
- B. Double delete of `buf`.
- C. The constructor of `Derived` does not run.
- D. The program prints `Base` and exits cleanly.
- E. No problem.

### Question 5
SELECT ALL true about `protected:` members of `class Base`.
- A. They are accessible inside member functions of `Base`.
- B. They are accessible inside member functions of a class derived `public`ly from `Base`.
- C. They are accessible from outside via `b.member;` if `b` is a `Base`.
- D. They are still inaccessible from outside via a `Derived` object.
- E. `friend` functions can access them.

### Question 6
SELECT ALL true about a **pure virtual** member function (e.g., `virtual double area() const = 0;`).
- A. Its class is abstract and cannot be instantiated directly.
- B. Every concrete derived class must override it.
- C. You may not store a `Shape*` if `Shape` is abstract.
- D. The base class may still provide a body (definition) for it.
- E. Calling it through a base pointer/reference dispatches dynamically.

### Question 7
What is the output?
```cpp
struct A { A() { std::cout << "A"; } ~A() { std::cout << "a"; } };
struct B : public A { B() { std::cout << "B"; } ~B() { std::cout << "b"; } };
int main() {
    B x;
}
```
- A. `ABba`
- B. `ABab`
- C. `BAab`
- D. `BAba`
- E. none of the other choices

### Question 8
Why does the `Picture` class in HW6 call `s.clone()` rather than `new Shape(s)` when adding a shape?
- A. Because `Shape` is abstract and cannot be instantiated.
- B. Because `clone()` returns the **derived** type via covariant return.
- C. Because `Picture` only sees a `Shape&` and doesn't know what concrete type to construct.
- D. Because virtual dispatch picks the correct `new` per shape kind.
- E. Because copy-and-swap requires it.

### Question 9
What does this print?
```cpp
struct A {
    virtual int f() const { return 1; }
    int g() const { return f() * 10; }   // calls virtual f()
};
struct B : public A {
    int f() const override { return 7; }
};
int main() {
    B b;
    A& r = b;
    std::cout << r.g();
}
```
- A. 10
- B. 70
- C. 17
- D. 71
- E. compile error

### Question 10
SELECT ALL situations where `dynamic_cast<Derived*>(basePtr)` is appropriate.
- A. The base must have at least one virtual function.
- B. You want a runtime-checked downcast.
- C. You don't care if the cast is correct; speed matters.
- D. Returns `nullptr` if the cast fails (for pointer form).
- E. The hierarchy must use `virtual` inheritance everywhere.

### Question 11
Which feature lets `Square::clone()` return `Square*` even though the base declares `Shape* clone() const = 0;`?
- A. Operator overloading
- B. Template specialization
- C. Covariant return type
- D. Virtual inheritance
- E. RTTI (`typeid`)

### Question 12
What does this print?
```cpp
struct Base {
    Base(int x)  { std::cout << "B" << x; }
    virtual ~Base() { std::cout << "~B"; }
};
struct Derived : public Base {
    Derived(int x) : Base(x + 1) { std::cout << "D" << x; }
    ~Derived() override { std::cout << "~D"; }
};
int main() {
    Base* p = new Derived(5);
    delete p;
}
```
- A. `B6D5~D~B`
- B. `D5B6~B~D`
- C. `B6D5~B`
- D. `B5D5~D~B`
- E. none of the other choices

### Question 13
SELECT ALL ways to prevent **object slicing** of polymorphic types.
- A. Pass arguments by reference (`const Shape&`) rather than by value (`Shape`).
- B. `= delete` the copy/move constructors and assignment in the base.
- C. Pass arguments via base **pointer** (`const Shape*`).
- D. Make the base class abstract so it can never be instantiated by value.
- E. Add a `virtual` keyword to the destructor.

### Question 14
Why **must** the destructor of a polymorphic base class be `virtual`?
- A. Because the C++ standard requires it.
- B. Because otherwise `delete basePtr` on a derived object skips the derived destructor — likely leaking resources.
- C. To allow `Derived` to override it.
- D. To enable runtime type identification (`typeid`).
- E. Because `dynamic_cast` requires it.

### Question 15
What does this print?
```cpp
struct A { virtual void f() = 0; };
struct B : public A { void f() override { std::cout << "B"; } };
struct C : public B { void f() override { std::cout << "C"; } };

void call(A& a) { a.f(); }

int main() {
    B b; C c;
    call(b);
    call(c);
}
```
- A. `BC`
- B. `CC`
- C. `BB`
- D. compile error — A is abstract
- E. none of the other choices

---

## Answer Key

1. **C** — Public inheritance models "is-a".
2. **B** — `f` is virtual, so dynamic dispatch picks `B::f`. Output: `B`. (Even if `B` left out `override`, the override is implicit; the slides recommend using `override` to catch typos.)
3. **A** — `f` is NOT virtual, so static binding picks the declared type's `f` (`A::f`). Output: `A`.
4. **A** — Without a virtual destructor, `delete p` (where `p`'s declared type is `Base*`) runs only `~Base`, leaving `buf` leaked.
5. **A, B, D, E** — `protected` is accessible from the class itself, derived classes, and friends — but NOT through an object from outside (`b.member` or `d.member` in `main`).
6. **A, B, D, E** — Abstract class can't be instantiated; concretes must override; pure virtuals MAY have bodies (called via `Base::area()`); calls still dispatch dynamically. (C is false — you can hold `Shape*` even though you can't construct a `Shape` directly.)
7. **A** — Construction is base→derived: `AB`. Destruction is reverse: `ba`. Output: `ABba`.
8. **A, B, C** — All three are true reasons. (D / E are not the reason. D mixes up new and dispatch; clone() is virtual so the derived `new` is dispatched, but the question is about why you use clone, not how it works internally.)
9. **B** — `r.g()` is non-virtual `A::g`, which calls virtual `f()`. Through `r`, `f()` dispatches to `B::f` (7), so `g` returns `7 * 10 = 70`.
10. **A, B, D** — These are the standard preconditions and behavior of `dynamic_cast`. (C: don't use `dynamic_cast` if you don't care about safety. E: virtual inheritance is unrelated.)
11. **C** — Covariant return type: an override may return a pointer/reference to a **more-derived** type than the base declared.
12. **A** — `new Derived(5)`: `Base(6)` runs → `B6`, then `Derived` body → `D5`. `delete p` dispatches virtually → `~Derived` (`~D`) then `~Base` (`~B`). Output: `B6D5~D~B`.
13. **A, B, C, D** — All four prevent slicing. (E: virtual dtor is for cleanup correctness, not slicing.)
14. **B** — That's the consequence in practical terms. (A is too vague; C is incidental; D / E aren't the reason.)
15. **A** — `b` dispatches to `B::f` (`B`); `c` dispatches to `C::f` (`C`). Output: `BC`.
