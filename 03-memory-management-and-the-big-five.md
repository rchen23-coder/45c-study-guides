# ICS 45C — Memory Management and the Big Five
> Covers the Dyn Array slide deck + HW4 (raw-pointer `String`) + the `AllocationTracker`

This is the guide where the rules of the language stop being suggestions. Once you `new` something on the heap, you own it. This guide builds the discipline that keeps you safe.

---

## Table of Contents
1. [Pointers vs References vs Objects](#1-pointers-vs-references-vs-objects)
2. [The Heap: `new` and `delete`](#2-the-heap-new-and-delete)
3. [Pointer Pitfalls (Why It's Hard)](#3-pointer-pitfalls-why-its-hard)
4. [RAII: Resources In Classes](#4-raii-resources-in-classes)
5. [The Rule of Three](#5-the-rule-of-three)
6. [The Copy Constructor](#6-the-copy-constructor)
7. [Copy Assignment `operator=`](#7-copy-assignment-operator)
8. [The Destructor](#8-the-destructor)
9. [Move Semantics — The Rule of Five](#9-move-semantics--the-rule-of-five)
10. [The Copy-and-Swap Idiom](#10-the-copy-and-swap-idiom)
11. [Self-Assignment and Other Gotchas](#11-self-assignment-and-other-gotchas)
12. [The Rule of Zero](#12-the-rule-of-zero)
13. [Private Helper Constructors](#13-private-helper-constructors)
14. [The `AllocationTracker` (HW4)](#14-the-allocationtracker-hw4)
15. [HW4 Pattern Quick Reference](#15-hw4-pattern-quick-reference)
16. [Key Rules to Remember](#16-key-rules-to-remember)

---

## 1. Pointers vs References vs Objects

| Concept | Syntax | Holds | Can rebind? | Can be null? |
|---|---|---|---|---|
| Object | `int i;` | a value | n/a | n/a |
| Reference | `int& r = i;` | an alias for `i` | **no** | **no** |
| Pointer | `int* p = &i;` | address of an object | yes (`p = &j;`) | yes (`p = nullptr;`) |

```cpp
int i = 10;
int& r = i;          // r is another name for i
int* p = &i;         // p stores the address of i

i  = 50;             // changes i to 50
r  = 60;             // changes i to 60 (r IS i)
*p = 70;             // *p means "the object p points to" → changes i to 70

p = nullptr;         // p now points to nothing
// *p = 80;          // CRASH: dereferencing nullptr is undefined behavior
```

**`*p`** is the **dereference** operator: "the object at the address in `p`". **`&x`** is the **address-of** operator: "give me a pointer to `x`".

A reference is initialized once at birth and forever refers to the same object. A pointer can be re-aimed at any time. References never need `nullptr`-checks; pointers usually do.

---

## 2. The Heap: `new` and `delete`

The **stack** holds local variables; they die at the end of their scope. The **heap** (a.k.a. *free store*) holds objects whose lifetime you manage explicitly. `new` allocates; `delete` frees.

```cpp
int*  p = new int{42};           // one int on heap, initialized to 42
delete p;                         // free it. p is now dangling — don't dereference.

int*  a = new int[100];           // 100 ints on heap (default-initialized = garbage)
int*  z = new int[100]{};         // 100 ints, all zero
delete[] a;                       // MUST be delete[] for array form
delete[] z;
```

Two pairs that must match exactly:

| Allocator | Deallocator |
|---|---|
| `new T`      | `delete p;` |
| `new T[N]`   | `delete[] p;` |

**Mismatching them** (e.g. `delete` on a `new[]` pointer) is undefined behavior. The HW4 `AllocationTracker` will literally count these mismatches.

Heap-size **may** be a runtime value (`new int[n]` where `n` is read from input). Stack arrays cannot — they need a `constexpr` size.

---

## 3. Pointer Pitfalls (Why It's Hard)

| Error | What it looks like | Consequence |
|---|---|---|
| **Uninitialized pointer** | `int* p;  *p = 0;` | Segfault (or worse — silent corruption) |
| **Memory leak** | `int* p = new int; p = new int;` (lost the first one) | Process grows; storage gone forever |
| **Dangling pointer** | `delete p; *p = 5;` | Undefined behavior |
| **Double delete** | `delete p; delete p;` | Heap corruption; usually crashes |
| **Mismatched delete** | `int* a = new int[10]; delete a;` | Undefined behavior |
| **Heap/stack confusion** | `delete localVar;` | Crash |

> "Wisdom: restrict dynamic allocation to class implementations."

That is the entire philosophy of the rest of this guide. Put `new`/`delete` inside class constructors and destructors; never let user code see raw `new`.

---

## 4. RAII: Resources In Classes

**RAII** = *Resource Acquisition Is Initialization*. The idea:

- A resource (heap memory, a file, a lock) is **acquired in the constructor**.
- It is **released in the destructor**.
- The compiler **guarantees the destructor runs** when the object dies, even if an exception is thrown.

So scope brackets `{ ... }` become resource brackets: as long as the object lives, the resource is alive. When it dies, the resource is freed automatically.

```cpp
class String {
    char* buf;                                // resource: heap memory
public:
    String(const char* s)  { buf = strdup(s); }   // acquire
    ~String()              { delete[] buf;     }  // release
    // ... and don't forget copy ctor / copy assignment / move ctor / move assignment ...
};

int main() {
    String s("hello");                        // buf allocated on heap
}                                              // ~String runs → delete[] buf
```

A class that obeys RAII never leaks memory — the compiler keeps you honest.

---

## 5. The Rule of Three

> **Rule of Three**: if your class manually manages a resource (e.g., holds a raw pointer that it `new`s and `delete`s), you must define **all three** of: copy constructor, copy assignment operator, destructor.

Why? Because the compiler's defaults do member-wise copies. With a raw pointer, that copies the **pointer**, not the **target**:

```cpp
String a("hello");        // a.buf → ['h','e','l','l','o','\0']
String b = a;             // default copy ctor: b.buf = a.buf  ← SAME ADDRESS!
// Now both destructors will run, both will delete[] the same memory → double delete.
```

The rule is named "Three" because before C++11 there were only three special members. With C++11 move ctor and move assignment join the party — see §9 ("Rule of Five").

Quick reference for the special members of a class that owns a heap array:

| Special member | What it must do |
|---|---|
| Default constructor | acquire (or leave null) |
| Copy constructor | **deep** copy of the target |
| Copy assignment | release old, deep-copy new |
| Destructor | release |
| Move constructor (C++11) | take ownership; null out source |
| Move assignment (C++11) | release old; take new; null out source |

---

## 6. The Copy Constructor

A copy constructor builds a fresh object as a **clone** of an existing one. Its signature:

```cpp
ClassName(const ClassName& other);
```

The `const &` is critical: we don't modify the source, and we don't want to recurse forever by copying it again.

```cpp
String::String(const String& s) {
    buf = strdup(s.buf);                      // strdup allocates fresh memory
                                              //   and copies the bytes over
}

// where:
char* String::strdup(const char* src) {
    int len = strlen(src);
    char* dup = new char[len + 1];            // +1 for '\0'
    strcpy(dup, src);
    return dup;
}
```

After the copy, `a.buf` and `b.buf` point to **different** chunks of memory holding the **same** characters. A "deep copy".

When is the copy constructor called? **Any time an object is built from another**:

```cpp
String b = a;             // direct initialization (copy ctor)
String c(a);              // direct initialization (copy ctor)
void f(String x);  f(a);  // pass-by-value (copy ctor builds x)
String g() { return a; }  // return-by-value (often elided — see §9)
```

---

## 7. Copy Assignment `operator=`

```cpp
ClassName& operator=(const ClassName& other);    // classic form
ClassName& operator=(ClassName s);                // copy-and-swap form (see §10)
```

The naive (and broken) version:

```cpp
String& String::operator=(const String& s) {     // ← DO NOT USE
    delete[] buf;                                  // release old
    buf = new char[strlen(s.buf) + 1];             // allocate new
    strcpy(buf, s.buf);                            // copy content
    return *this;
}
```

Two bugs lurk here:
1. **Self-assignment** (`s = s;`) deletes `s.buf` and then reads from it.
2. If `new char[...]` throws (out of memory), `buf` is already deleted — the object is left in a broken state ("not exception-safe").

The robust fix is the **copy-and-swap idiom** (§10).

---

## 8. The Destructor

```cpp
String::~String() { delete[] buf; }
```

That's it. The buffer was acquired by the constructor (or copy ctor / strdup); the destructor releases it. `delete[]` on `nullptr` is a no-op, which makes move semantics safe (§9).

The destructor is **called automatically** when:
- A stack object goes out of scope.
- A heap object has `delete` called on it.
- A class member dies because its containing object died.

You never call destructors explicitly.

---

## 9. Move Semantics — The Rule of Five

Move semantics let you **transfer** ownership of resources instead of copying them. A move steals the heap pointer; copying allocates a fresh buffer.

```cpp
// Move constructor: take resources from a soon-to-die object
String::String(String&& s)                     // && = rvalue reference
  : buf(s.buf)                                  // steal the buffer pointer
{
    s.buf = nullptr;                            // critical! source must not delete[] it
}

// Move assignment: usually written via copy-and-swap or:
String& String::operator=(String&& s) {
    delete[] buf;                                // release current
    buf   = s.buf;                               // steal new
    s.buf = nullptr;                             // null source
    return *this;
}
```

When does the compiler call the **move** ctor instead of the copy ctor?

```cpp
String s1("hello");
String s2 = std::move(s1);             // explicit: I'm done with s1
String s3 = make_string("world");      // implicit: the return value is a temporary
```

`std::move(x)` is a cast: it doesn't move anything, it tells the compiler "treat x as an rvalue, so the move ctor/assignment is picked." After `std::move(x)`, `x` is in a valid-but-unspecified state (HW4: `x.buf == nullptr`).

**Rule of Five**: a class that manually manages a resource should define **all five** — default? no, but: copy ctor, copy assignment, destructor, move ctor, move assignment. HW4 wants all of them.

> **Why move at all?** Performance. Copying a 1-MB string into a function argument costs 1 MB of allocation + copy. Moving it costs setting two pointers.

---

## 10. The Copy-and-Swap Idiom

The most robust way to write copy assignment. HW4 uses it:

```cpp
void String::swap(String& s) {                 // member swap — just exchanges buf
    char* tmp = buf;
    buf       = s.buf;
    s.buf     = tmp;
}

String& String::operator=(String s) {          // ← s is taken **by value**!
    swap(s);                                    // swap our state with the copy
    return *this;
}                                                // s dies, frees old buf
```

What happens step by step:
1. The argument `String s` is constructed via the copy ctor (or move ctor — the compiler picks based on whether the caller passed an lvalue or rvalue).
2. We `swap(s)` — `*this` now holds the new data; `s` holds our old data.
3. `s` goes out of scope; its destructor frees the old data.

Why it's beautiful:
- **Self-assignment safe**: `a = a` makes a copy first, then swaps — no `delete` of live memory.
- **Exception safe**: if the copy fails, `*this` is untouched.
- **One operator= for both copy and move**: passing an rvalue makes the parameter use the move ctor; passing an lvalue uses the copy ctor.
- **No code duplication**: assignment delegates to the constructors.

The HW4 `String` literally has:

```cpp
String& String::operator=(String s) { swap(s); return *this; }
```

That one line handles every assignment case correctly.

---

## 11. Self-Assignment and Other Gotchas

Even with copy-and-swap, watch for:

```cpp
String s("hello");
s += s;                                         // self-append
```

The HW4 `operator+=` builds a *new* `String result(buf + s.buf)` and swaps — safe.
The HW3 `operator+=` had to explicitly copy `s` first when `&s == this`.

Other classic mistakes:

| Mistake | Fix |
|---|---|
| `delete p; p->foo();` (dangling) | Set `p = nullptr;` after delete; check before deref. |
| `new int[n];` no `delete[]` | RAII — put it in a class. |
| Copy ctor that **shares** the pointer | Always deep-copy. |
| Move ctor that doesn't null the source | Always `s.buf = nullptr;` after stealing. |

---

## 12. The Rule of Zero

If your class only holds standard-library types (`std::string`, `std::vector`, `std::map`, smart pointers, ...) you may **not need to write any of the five**. Those types already manage their own resources, and the compiler's defaults call into them correctly.

```cpp
struct Named_map {                              // Rule of Zero
    std::string         name;
    std::map<int, int>  rep;
};
Named_map nm;                                    // default ctor: works
Named_map nm2 = nm;                              // copy ctor: works
nm = nm2;                                        // copy assign: works
// destructor: works
```

HW7 `Matrix<T>` (next guide) and HW8 `Gradebook` (STL guide) use this rule — they hold a `vector` and let the compiler manage everything.

---

## 13. Private Helper Constructors

When a method needs a partially-constructed instance (e.g., `reverse()` wants a fresh buffer of a known size), use a **private** constructor that does the minimum:

```cpp
class String {
public:
    String(const char* s);                      // public — fully constructs
private:
    explicit String(int length);                // private — buf allocated, content empty
};

String::String(int length) {
    buf = new char[length + 1];                 // raw heap allocation
    buf[0] = '\0';                              // valid but empty
}

String String::reverse() const {                // public method that needs it
    int len = strlen(buf);
    String result(len);                          // private ctor — no double allocation
    reverse_cpy(result.buf, buf);                // fill the buffer in place
    return result;
}
```

The HW4 write-up calls this out explicitly. Without the private ctor, `reverse()` would have to allocate a temp buffer, build a public `String` from it (which allocates a second buffer), and copy across — wasteful.

---

## 14. The `AllocationTracker` (HW4)

HW4 ships an `AllocationTracker` class that **overrides global `new`/`delete`** to count every allocation. It uses the **C++17 polymorphic memory resource** (`std::pmr`) so the tracker's own bookkeeping doesn't recursively call `new`.

What you really need to know about it:

| Method | Returns |
|---|---|
| `get_num_allocations()` | total `new`s performed |
| `get_bytes_allocated()` | sum of sizes |
| `get_double_deletes()` | number of `delete` calls on already-freed pointers |
| `get_mismatching_deletes()` | `delete` vs `delete[]` mix-ups |
| `print_allocation_report(ostream&)` | human-readable summary |

The pattern in HW4 tests:

```cpp
AllocationTracker tracker;          // start tracking
{
    String s("hello");               // 1 allocation
    String t = s;                    // 2nd allocation (deep copy)
}                                     // both destructors run → 2 deletes
tracker.print_allocation_report(std::cout);
// "2 allocations have been performed in total. All allocations have been deleted!"
```

If your destructor forgets to `delete[] buf`, the tracker reports leaks. If your move ctor forgets `s.buf = nullptr;`, you get a double delete. The tracker is your safety net — run it on every test.

Implementation gist (you do not write this, but it shows what's happening):

```cpp
void* operator new(std::size_t size) {           // global override!
    void* p = std::malloc(size);
    for (auto* t : trackers) t->add_allocation(p, size, Single);
    return p;
}
void operator delete(void* p) noexcept {
    for (auto* t : trackers) t->delete_allocation(p, Single);
    std::free(p);
}
// + matching new[] / delete[] versions tagged Array
```

So every `new` in your code (and the library's code) is observable.

---

## 15. HW4 Pattern Quick Reference

### The HW4 `String` class — the Big Five

```cpp
class String {
public:
    explicit String(const char* s = "");        // 1. constructor — allocates
    String(const String& s);                    // 2. copy ctor — deep copy
    String(String&& s);                         // 3. move ctor — steal pointer
    void   swap(String& s);                     // helper — used by op=
    String& operator=(String s);                // 4. copy-AND-move assignment via copy-and-swap
    ~String();                                  // 5. destructor — delete[]
private:
    char* buf;                                  // raw pointer; private ctor for sizing
    explicit String(int length);
};
```

Implementations distilled:

```cpp
String::String(const char* s)        { buf = strdup(s); }
String::String(const String& s)      { buf = strdup(s.buf); }
String::String(String&& s)           { buf = s.buf; s.buf = nullptr; }
String::String(int length)           { buf = new char[length + 1]; buf[0] = '\0'; }

void   String::swap(String& s)        { char* t = buf; buf = s.buf; s.buf = t; }
String& String::operator=(String s)   { swap(s); return *this; }
String::~String()                     { delete[] buf; }
```

### `reverse` / `operator+` use the private constructor

```cpp
String String::reverse() const {
    int len = strlen(buf);
    String r(len);                   // private ctor — single allocation
    reverse_cpy(r.buf, buf);
    return r;                         // move ctor (compiler usually elides entirely)
}

String String::operator+(const String& s) const {
    String r(strlen(buf) + strlen(s.buf));
    strcpy(r.buf, buf);
    strcat(r.buf, s.buf);
    return r;
}
```

### `read()` reuses `strdup`

```cpp
void String::read(std::istream& in) {
    char tmp[4096];
    in >> tmp;                        // reads one whitespace-delimited word
    delete[] buf;                     // release old
    buf = strdup(tmp);                // acquire new
}
```

### Auditing with `AllocationTracker`

```cpp
AllocationTracker tracker;
{
    String s("hello world");
    String t = s.reverse();
}                                      // both destructors run here
tracker.print_allocation_report(std::cout);
// 2 allocations (?? bytes) have been performed in total.
// All allocations have been deleted!
// There were no delete errors!
```

---

## 16. Key Rules to Remember

- **`new` ↔ `delete`** and **`new[]` ↔ `delete[]`**. Never cross the streams.
- **`delete nullptr` is safe**; `delete` of any other invalid pointer is not.
- **Always pair allocation with a destructor** — apply RAII.
- **Rule of Three (or Five) or Zero.** Half-managing a resource leads to crashes.
- **The copy constructor takes `const T&`.** Anything else is wrong.
- **The destructor never throws.**
- **The move ctor must null out the source's resource pointers.**
- **`std::move(x)` does not move; it only marks `x` as an rvalue.** After it, the original is in a valid-but-unspecified state.
- **Self-assignment** (`a = a;`) must be safe — copy-and-swap gets this for free.
- **Prefer copy-and-swap** for `operator=`. One line handles copy + move + exception safety.
- **Static C-string helpers carry over from HW3.** Don't duplicate them — `strdup` and friends were promoted to do the heap allocation in HW4.
- **Private size constructor** (e.g., `String(int length)`) avoids redundant allocation in `reverse()` / `operator+()`.
- **Track allocations** when debugging — every leak the `AllocationTracker` reports is a bug in your class.

Next guide: **04 — Linked Lists and Recursion** rewrites `String` on top of a linked list of `Node`s — same Rule-of-Five obligations, different ownership graph.
