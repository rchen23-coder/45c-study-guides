# ICS 45C — Inheritance and Polymorphism
> Covers the Topic 6 slide deck + HW6 (the abstract `Shape` hierarchy and the `Picture` container)

Inheritance lets one class extend another, and polymorphism lets a single piece of code work with many derived types. HW6 puts both together in a `Picture` that holds any kind of `Shape*`.

---

## Table of Contents
1. [The "Is-a" vs "Has-a" Relationship](#1-the-is-a-vs-has-a-relationship)
2. [Declaring an Inheriting Class](#2-declaring-an-inheriting-class)
3. [Member Access: `public`/`protected`/`private`](#3-member-access)
4. [Subtype Conversion](#4-subtype-conversion)
5. [Static vs Dynamic Binding](#5-static-vs-dynamic-binding)
6. [`virtual` Methods](#6-virtual-methods)
7. [Pure Virtual and Abstract Base Classes](#7-pure-virtual-and-abstract-base-classes)
8. [Virtual Destructors (Mandatory)](#8-virtual-destructors-mandatory)
9. [The `clone()` Prototype Pattern](#9-the-clone-prototype-pattern)
10. [Construction Order and Initializer Lists](#10-construction-order-and-initializer-lists)
11. [Inheritance Modes Compared](#11-inheritance-modes-compared)
12. [Multiple Inheritance and `virtual` Base Classes](#12-multiple-inheritance-and-virtual-base-classes)
13. [The HW6 Hierarchy](#13-the-hw6-hierarchy)
14. [The `Picture` Container](#14-the-picture-container)
15. [HW6 Pattern Quick Reference](#15-hw6-pattern-quick-reference)
16. [Key Rules to Remember](#16-key-rules-to-remember)

---

## 1. The "Is-a" vs "Has-a" Relationship

Two ways one class can use another:

| Relationship | Phrase | C++ |
|---|---|---|
| **Containment / Has-a** | "an `Engine` is part of a `Car`" | data member: `class Car { Engine e; ... };` |
| **Inheritance / Is-a** | "a `Car` is a kind of `Vehicle`" | base class: `class Car : public Vehicle { ... };` |

Use inheritance only when "is-a" really holds. A `Stack` is **not** a kind of `Vector`, so deriving `Stack` from `Vector` would let users index a stack, which makes no sense. Composition (a `Stack` *has* a `Vector`) is the right relationship there.

---

## 2. Declaring an Inheriting Class

```cpp
class Shape { /* ... */ };

class Triangle : public Shape { /* ... */ };       // Triangle "is-a" Shape
```

- `Shape` is the **base class** (also: superclass, parent).
- `Triangle` is the **derived class** (also: subclass, child).
- `public` says "everywhere a `Shape` is expected, a `Triangle` can be used."

A derived class inherits all members of the base class — data, functions, and (with caveats) constructors/destructor — and can add more.

```cpp
class Window : public Screen { /* extra members */ };
class Menu   : public Window { /* extra members */ };       // chains of inheritance
```

---

## 3. Member Access

```cpp
class Shape {
public:                                              // accessible to everyone
    void print(std::ostream& out) const;
protected:                                            // accessible to derived classes
    Shape(const Shape& other) = default;
private:                                              // accessible only to Shape
    Point        center;
    std::string  name;
};
```

| Section | Class users | Derived classes | The class itself |
|---|---|---|---|
| `public:` | ✓ | ✓ | ✓ |
| `protected:` | ✗ | ✓ | ✓ |
| `private:` | ✗ | ✗ | ✓ |

**Rule of thumb**: data members → `private`. Helper methods that derived classes need → `protected`. Public interface for clients → `public`.

The HW6 `Shape`'s copy constructor is `protected` — derived classes (`Circle`, `Rectangle`, ...) can copy a `Shape` part, but outside code cannot construct a `Shape` directly (it's abstract anyway).

---

## 4. Subtype Conversion

Because a `Triangle` *is* a `Shape`, a pointer or reference to `Shape` can hold a `Triangle`:

```cpp
Triangle t({0,0}, "T", 3, 4);
Shape*   sp = &t;                                    // OK: Triangle → Shape*
Shape&   sr = t;                                     // OK: Triangle → Shape&
```

The reverse is **not** automatic: a `Shape*` is not guaranteed to point at a `Triangle`.

```cpp
Shape s = ... ; Triangle* tp = &s;                   // ERROR
```

If you need to downcast (rare; usually a sign of bad design):
- `static_cast<Triangle*>(sp)` — fast, trusts you. Undefined behavior if wrong.
- `dynamic_cast<Triangle*>(sp)` — runtime-checked; returns `nullptr` (or throws for refs) if `sp` doesn't actually point at a `Triangle`.

---

## 5. Static vs Dynamic Binding

Given:

```cpp
CheckedVector cv(20);
Vector& vp = cv;                                     // base reference to derived
vp[0];                                                // which operator[] is called?
```

| Binding | When chosen | Method called |
|---|---|---|
| **Static** | non-virtual methods | the one matching the **declared** type of `vp` → `Vector::operator[]` |
| **Dynamic** | `virtual` methods | the one matching the **actual** type at runtime → `CheckedVector::operator[]` |

In other words: if you want the derived class's version to be called through a base pointer/reference, the method **must be `virtual`**.

---

## 6. `virtual` Methods

```cpp
class Shape {
public:
    virtual double area() const;                     // dynamic binding
    void          print(std::ostream& out) const;    // static binding
};

class Circle : public Shape {
public:
    double area() const override;                     // override marks intent
};
```

- `virtual` in the **base** enables dynamic dispatch.
- `override` in the **derived** asks the compiler to verify you really are overriding. If the signature is wrong (e.g., typo, wrong const-ness), the compiler will tell you.
- Once a method is `virtual` in the base, it's `virtual` in every derived class — re-stating `virtual` in derived classes is allowed but optional.

```cpp
void print_area(const Shape& s) {                    // works for any Shape
    std::cout << s.area();                            // dispatches to derived
}

Circle c({0,0}, "C", 5);
Rectangle r({0,0}, "R", 4, 3);
print_area(c);                                        // calls Circle::area
print_area(r);                                        // calls Rectangle::area
```

That single function over a base reference is **polymorphic** code: it doesn't know or care which concrete shape it has.

**What about pass by value?** `void print_area(Shape s)` is wrong: it would **slice** the object, copying only the `Shape` portion (and you can't even compile it if `Shape` is abstract). Always use `Shape*` or `Shape&` for polymorphic code.

---

## 7. Pure Virtual and Abstract Base Classes

A **pure virtual** function has `= 0` after its signature — no implementation provided.

```cpp
class Shape {
public:
    virtual double area() const = 0;                 // pure virtual
    virtual void   draw(std::ostream& out) const = 0;
    virtual Shape* clone() const = 0;                 // see §9
};
```

A class with one or more pure virtuals is **abstract**. You cannot instantiate one:

```cpp
Shape s;                                              // ERROR — Shape is abstract
Shape* sp = new Triangle(...);                        // OK — Triangle overrides them all
```

Every derived class must override every pure virtual (or itself remain abstract). When `Circle` overrides `area`, `draw`, and `clone`, it becomes a **concrete** class.

Abstract bases let you express "every `Shape` must have an `area`" without committing to *how*.

---

## 8. Virtual Destructors (Mandatory)

When you `delete` an object through a base pointer, **the destructor that runs depends on whether `~Shape` is `virtual`**:

```cpp
Shape* sp = new Circle(...);
delete sp;                                            // which destructor runs?
```

| `~Shape` | What runs |
|---|---|
| non-virtual | only `~Shape` — `Circle`'s additions are leaked |
| `virtual`   | `~Circle` then `~Shape` — correct cleanup |

> **The rule**: if a class has any `virtual` method, its destructor **must** be `virtual`.

The HW6 `Shape` declares:

```cpp
virtual ~Shape() = default;                          // virtual & defaulted
```

`= default` says "use the compiler-generated body (an empty one here)" — but the `virtual` is the part that matters. Forgetting this is one of the most common (and silent) bugs in polymorphic code.

---

## 9. The `clone()` Prototype Pattern

Problem: `Picture::add(const Shape& s)` accepts any shape. To store it, the `Picture` needs to **copy** it — but it only sees the base type, so it can't say `new Circle(s)`.

Solution: every `Shape` knows how to clone itself.

```cpp
class Shape {
public:
    virtual Shape* clone() const = 0;                 // each subclass returns its own type
};

class Circle : public Shape {
public:
    Circle* clone() const override {                  // covariant return type — allowed
        return new Circle(*this);                      // call Circle's copy ctor
    }
};
class Rectangle : public Shape {
public:
    Rectangle* clone() const override { return new Rectangle(*this); }
};
class Triangle  : public Shape { public: Triangle*  clone() const override { return new Triangle(*this);  } };
class Square    : public Rectangle { public: Square* clone() const override { return new Square(*this);    } };
```

Now any code can deep-copy a shape it only sees as a `Shape&`:

```cpp
void Picture::add(const Shape& s) {
    ListNode* node = new ListNode;
    node->shape = s.clone();                          // dispatches to the right new
    node->next  = nullptr;
    /* ... link node in ... */
}
```

This is the **Prototype** design pattern. `Square::clone()` returns `Square*` even though the base says `Shape*` — that's called a **covariant return type**, and the language explicitly permits it.

---

## 10. Construction Order and Initializer Lists

When a derived object is constructed:

1. The base class's constructor runs first.
2. Then each new data member of the derived class is initialized.
3. Then the derived constructor body runs.

If you don't say which base constructor to call, the compiler picks the **default** (no-arg) base constructor. If the base has no default constructor, you **must** name one in the initializer list:

```cpp
Circle::Circle(Point center, std::string name, int radius)
  : Shape(center, std::move(name)),                   // base ctor — required, no default
    radius(radius) {}                                  // own data members

Square::Square(Point center, std::string name, int side)
  : Rectangle(center, std::move(name), side, side) {}  // forward to Rectangle
```

The Square case is interesting: a Square *is a* Rectangle with width == height, so its constructor just delegates to `Rectangle`'s 4-arg constructor with `side` for both width and height.

**Destruction order is reversed**: derived destructor body → derived members → base destructor.

---

## 11. Inheritance Modes Compared

Three keywords change visibility of base members in the derived class:

```cpp
class A : public    B { ... };     // most common
class A : protected B { ... };
class A : private   B { ... };
```

| Mode | `public` in B becomes ... | `protected` in B becomes ... | "is-a" relationship? |
|---|---|---|---|
| `public`    | `public`    in A | `protected` in A | yes — external code can convert A→B |
| `protected` | `protected` in A | `protected` in A | only friends/derived can convert |
| `private`   | `private`   in A | `private`   in A | no — A "uses" B's implementation |

In `private`-inheritance there's no "is-a" relationship to outsiders — derived members can use base methods, but external code can't pass `A*` where `B*` is expected.

`public` inheritance is the default and what HW6 uses. The other two are rare; pick composition instead.

---

## 12. Multiple Inheritance and `virtual` Base Classes

A class may inherit from more than one base:

```cpp
class Derived : public Base1, public Base2 { ... };
```

Two problems can arise:
- **Ambiguity**: if both bases have a method `foo()`, calling `d.foo()` is ambiguous. Fix by qualifying: `d.Base1::foo();`.
- **Diamond**: if `Base1` and `Base2` both derive from a common `Top`, `Derived` ends up with *two* copies of `Top`. Mark the inheritance `virtual` to share a single instance:

```cpp
class Top { ... };
class Base1 : virtual public Top { ... };
class Base2 : virtual public Top { ... };
class Derived : public Base1, public Base2 { ... };   // one Top, not two
```

`virtual` base classes have restrictions (they need a no-arg constructor) and are tricky. HW6 sticks to single inheritance; multiple inheritance shows up in the slides as awareness, not as required practice.

---

## 13. The HW6 Hierarchy

```
            Shape (abstract)
              ├── Circle
              ├── Rectangle ─── Square
              └── Triangle
```

The `Shape` interface (`shape.hpp`):

```cpp
struct Point { int x; int y; };

class Shape {
public:
    Shape(Point center, std::string name);            // base constructor
    void print(std::ostream& out) const;              // non-virtual: name + center + area
    Shape& operator=(const Shape&) = delete;          // no slicing-assignment!

    virtual double area() const = 0;                  // pure virtual
    virtual void   draw(std::ostream& out) const = 0; // pure virtual
    virtual Shape* clone() const = 0;                 // pure virtual

    virtual ~Shape() = default;                       // virtual destructor — mandatory
protected:
    Shape(const Shape& other) = default;              // copy ctor protected — only derived can copy
private:
    Point        center;
    std::string  name;
};
```

`Shape::print` is **non-virtual** — every shape prints the same way (name, center, area). It calls the **virtual** `area()`, so the right one runs:

```cpp
void Shape::print(std::ostream& out) const {
    out << name << " at (" << center.x << ',' << center.y << ") area = " << area() << '\n';
    //                                                                       ^^^^^^
    //                                                              virtual call!
}
```

`operator=` is **deleted** to prevent slicing assignment: `*sp = *sq;` would copy only the `Shape` portion of a `Square`. Deleting it makes that a compile error.

A concrete derived class (`Circle`):

```cpp
class Circle : public Shape {
public:
    Circle(Point center, std::string name, int radius);
    double  area() const override;
    void    draw(std::ostream& out) const override;
    Circle* clone() const override;                   // covariant return
protected:
    Circle(const Circle&) = default;                  // protected, like Shape's
private:
    int radius;
};

Circle::Circle(Point center, std::string name, int radius)
  : Shape(center, std::move(name)),                   // forward to base
    radius(radius) {}

double Circle::area() const {
    return std::numbers::pi * radius * radius;
}
void Circle::draw(std::ostream& out) const {
    for (int y = -radius; y <= radius; y += 2) {
        for (int x = -radius; x <= radius; ++x)
            out << (x*x + y*y <= radius*radius ? '*' : ' ');
        out << '\n';
    }
}
Circle* Circle::clone() const { return new Circle(*this); }
```

`Square : public Rectangle` is even simpler — it inherits `area` and `draw` from `Rectangle` (a square *is a* rectangle with equal sides) and only overrides `clone`:

```cpp
class Square : public Rectangle {
public:
    Square(Point center, std::string name, int side);
    Square* clone() const override;
};

Square::Square(Point center, std::string name, int side)
  : Rectangle(center, std::move(name), side, side) {}   // forward with side, side

Square* Square::clone() const { return new Square(*this); }
```

---

## 14. The `Picture` Container

`Picture` owns a singly-linked list of `Shape*`. Because shapes are polymorphic, this is the natural fit for a heap-allocated linked list (vs. an array of `Shape` objects, which would slice).

```cpp
class Picture {
public:
    Picture();
    Picture(const Picture& other);                    // deep copy via clone
    Picture(Picture&& other);                          // steal head/tail
    void    swap(Picture& other);
    Picture& operator=(const Picture& other);          // copy-and-swap
    Picture& operator=(Picture&& other);               // swap

    void   add(const Shape& shape);                    // clone and append
    void   print_all(std::ostream& out) const;
    void   draw_all(std::ostream& out) const;
    double total_area() const;

    ~Picture();
private:
    struct ListNode {
        Shape*    shape;                              // owned heap shape
        ListNode* next;
    };
    ListNode* head;
    ListNode* tail;
};
```

The `add` method is the key insight — `Picture` doesn't know what kind of shape it's getting, so it calls `clone()`:

```cpp
void Picture::add(const Shape& shape) {
    ListNode* node = new ListNode;
    node->shape = shape.clone();                      // virtual dispatch picks the right new
    node->next  = nullptr;
    if (!tail) head = tail = node;
    else { tail->next = node; tail = node; }
}
```

`Picture` is the owner — when it dies, it `delete`s every shape **through the base pointer**, which is why `~Shape` must be virtual:

```cpp
Picture::~Picture() {
    ListNode* curr = head;
    while (curr) {
        ListNode* next = curr->next;
        delete curr->shape;                           // virtual destructor → right one runs
        delete curr;
        curr = next;
    }
}
```

The copy constructor walks the source list and re-`add`s every shape (which clones it):

```cpp
Picture::Picture(const Picture& other) : head(nullptr), tail(nullptr) {
    for (ListNode* c = other.head; c; c = c->next)
        add(*c->shape);                                // dereference + add (re-clones)
}
```

Move constructor steals; copy-and-swap handles assignment:

```cpp
Picture::Picture(Picture&& other) : head(other.head), tail(other.tail) {
    other.head = other.tail = nullptr;
}
Picture& Picture::operator=(const Picture& other) {
    Picture tmp(other); swap(tmp); return *this;
}
Picture& Picture::operator=(Picture&& other) { swap(other); return *this; }
```

`total_area` is a tiny polymorphic loop:

```cpp
double Picture::total_area() const {
    double total = 0.0;
    for (ListNode* c = head; c; c = c->next) total += c->shape->area();
    return total;                                      // each ->area() dispatches per type
}
```

---

## 15. HW6 Pattern Quick Reference

### Building a hierarchy: abstract base + concrete leaves + virtual dtor

```cpp
class Shape {
public:
    virtual double  area()  const = 0;
    virtual void    draw(std::ostream&) const = 0;
    virtual Shape*  clone() const = 0;
    virtual ~Shape() = default;                       // VIRTUAL!
};

class Circle    : public Shape   { /* override 3 */ };
class Rectangle : public Shape   { /* override 3 */ };
class Triangle  : public Shape   { /* override 3 */ };
class Square    : public Rectangle { Square* clone() const override; };
```

### Polymorphic ownership: container of base-pointers, deep-copy via clone

```cpp
struct ListNode { Shape* shape; ListNode* next; };

void Picture::add(const Shape& s) {                  // any Shape — even a Square
    ListNode* n = new ListNode{s.clone(), nullptr};
    if (!tail) head = tail = n;
    else { tail->next = n; tail = n; }
}

Picture::~Picture() {
    while (head) {
        auto* n = head->next;
        delete head->shape;                           // virtual dtor → correct cleanup
        delete head;
        head = n;
    }
}
```

### Calling polymorphic methods through a base reference

```cpp
void print_all(const Picture& p, std::ostream& out) {
    for (auto* n = p.head; n; n = n->next) {
        n->shape->print(out);                         // non-virtual; calls virtual area() inside
        n->shape->draw(out);                          // virtual: shape-specific
    }
}
```

### Constructor chain for a Square

```cpp
Square::Square(Point c, std::string name, int side)
  : Rectangle(c, std::move(name), side, side) {}     // Rectangle ctor calls Shape ctor first
```

The construction sequence is: `Shape` body → `Rectangle` body → `Square` body. Destruction is reversed.

---

## 16. Key Rules to Remember

- Use inheritance for **is-a** relationships only. Otherwise, prefer composition.
- The base class must be **declared `public`** in the derivation list for external code to see the subtype relationship.
- A method must be **`virtual`** in the base to dispatch dynamically. Use `override` in derived classes — it catches signature typos.
- An **abstract class** has one or more pure virtuals (`= 0`). You cannot instantiate one. Use it as a base for polymorphic code.
- **Always declare `~Base()` `virtual`** when a class has any virtual function. Otherwise `delete sp;` through a base pointer leaks.
- Use **`clone()`** to deep-copy a polymorphic object you only know by reference.
- A derived constructor's initializer list must **name the base constructor** if the base has no default.
- The derived constructor body runs **after** the base constructor.
- `delete` an inheriting `operator=` from the base to avoid **object slicing** during assignment.
- Pass polymorphic objects by **pointer or reference**, never by value (it slices to the base subobject).
- `dynamic_cast` is the runtime-safe way to downcast — usually a sign you should rethink the design.
- **`Picture` owns its shapes.** Cloning on `add` and `delete`-ing on destruction is the discipline; nobody else may touch those pointers.

Next guide: **06 — Templates, Lambdas, Exceptions, and Namespaces** generalizes a class to work with any type, then layers in the supporting features used everywhere from HW7 on.
