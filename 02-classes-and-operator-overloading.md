# ICS 45C — Classes and Operator Overloading
> Covers the Classes and Strings slide decks + HW2 (Coins, briefly) and HW3 (the fixed-buffer `String` class, in depth)

This guide turns the data + behavior of a problem into a class, and then teaches every operator the course expects you to overload. HW3 is the canonical exercise — a `String` class with the **Rule of Three** (still without raw pointers; that's the next guide).

---

## Table of Contents
1. [What a Class Is](#1-what-a-class-is)
2. [Members, Access, and `this`](#2-members-access-and-this)
3. [Constructors](#3-constructors)
4. [Destructors](#4-destructors)
5. [`const`-Correctness](#5-const-correctness)
6. [`class` vs `struct`](#6-class-vs-struct)
7. [Operator Overloading: The Big Picture](#7-operator-overloading-the-big-picture)
8. [Comparison Operators: `==`, `!=`, `<`, `>`, `<=`, `>=`](#8-comparison-operators)
9. [Stream Operators: `<<` and `>>`](#9-stream-operators)
10. [Indexing: `operator[]`](#10-indexing-operator)
11. [Concatenation: `operator+` and `operator+=`](#11-concatenation-operator-and-operator)
12. [Assignment: `operator=`](#12-assignment-operator)
13. [Static Helper Functions (C-Strings)](#13-static-helper-functions-c-strings)
14. [TDD: How to Build a Class Like `String`](#14-tdd-how-to-build-a-class-like-string)
15. [HW2 / HW3 Pattern Quick Reference](#15-hw2--hw3-pattern-quick-reference)
16. [Key Rules to Remember](#16-key-rules-to-remember)

---

## 1. What a Class Is

A **class** bundles data (members) with the operations that work on it (methods). An **object** is an instance of a class. Each object has its own copy of the data members.

```cpp
class Complex {
    float re, im;                            // private data (the "representation")
public:
    Complex(float r = 0.0, float i = 0.0)    // constructor with defaults
      : re(r), im(i) {}                      // initializer list (see §3)

    Complex add(Complex c) const {           // method: const = does not modify *this
        return Complex(re + c.re, im + c.im);
    }
    void print(std::ostream& out) const {
        out << '(' << re << '+' << im << "i)";
    }
};

int main() {
    Complex a(1.0, 1.0);                     // object on the stack
    Complex b(2.0, 1.0);
    Complex c = a.add(b);                    // call method on object a
    c.print(std::cout);                      // prints (3+2i)
}
```

A class definition describes a **type**; it does not allocate anything. Storage is allocated when you construct an object.

---

## 2. Members, Access, and `this`

Inside a member function the hidden parameter `this` is a pointer to the current object. Member access is implicit — `re` inside `print` means `this->re`.

```cpp
void Complex::print(std::ostream& out) const {
    out << re;                               // same as this->re
    out << "+" << im << "i";                 // same as this->im
}
```

Access levels:

| Section | Visibility |
|---|---|
| `public:` | accessible to any code |
| `private:` (default for `class`) | accessible only to members of the same class |
| `protected:` | members + derived classes (Guide 05) |

**Simple rule for now:**
- Data members → `private`.
- Most member functions → `public`.
- Helpers (e.g., the C-string `strncpy` used inside `String`) → may be `private` or `static`.

---

## 3. Constructors

A **constructor** runs automatically when an object is born. Its name matches the class. It often uses an **initializer list** ( the `:` before `{`) to construct each member.

```cpp
class Coins {
    int quarters, dimes, nickels, pennies;
public:
    Coins(int q, int d, int n, int p)        // four-argument constructor
      : quarters(q), dimes(d),
        nickels(n),  pennies(p) {}           // body empty — init list does the work
};

Coins purse(3, 1, 0, 5);                     // 3 quarters, 1 dime, 0 nickels, 5 pennies
```

**Default constructor**: a constructor with no arguments (or all defaulted). If you write none, the compiler may provide one — but only if it can default every data member.

```cpp
class String {
public:
    explicit String(const char* s = "");     // single-arg w/ default → also default-constructible
};

String s;           // calls String("")
String s2("hi");
```

**`explicit`** prevents implicit one-argument conversions: with `explicit`, you cannot write `String s = "hi";` (it must be `String s("hi");` or `String s{"hi"};`). Without `explicit`, an implicit conversion `const char* → String` would happen anywhere a `String` is expected — usually undesirable.

---

## 4. Destructors

A **destructor** (`~ClassName()`) runs automatically when the object dies. It cleans up resources. For HW3 the destructor is empty — no heap memory yet:

```cpp
class String {
public:
    ~String() {}                              // nothing to free; the buf array
                                              //  is a member, dies with the object
};
```

Starting in HW4 (raw pointer) the destructor must `delete[] buf`. See Guide 03.

When an object dies depends on where it lives:
- Stack object dies at end of its enclosing scope (`}`).
- Heap object (`new`) dies when you call `delete`.
- Temporary objects die at end of the full expression.

---

## 5. `const`-Correctness

A member function marked `const` promises not to modify `*this`. You can call `const` methods on a `const` object; you cannot call non-`const` methods on a `const` object.

```cpp
class Coins {
public:
    int total_value_in_cents() const;        // read-only, safe on const Coins
    void deposit_coins(Coins& other);        // modifies *this — not const
};

void show(const Coins& c) {                  // c is read-only
    std::cout << c.total_value_in_cents();   // OK — const method
    // c.deposit_coins(...);                 // ERROR — not const
}
```

| Use of `const` | Meaning |
|---|---|
| `int f(const T& x)` | parameter `x` cannot be modified |
| `int f() const` (member) | method does not modify `*this` |
| `const T t = 5;` | the object `t` itself is fixed |
| `const T* p` | pointer to a fixed `T` (the target is read-only) |

Make every method that doesn't change state `const`. This catches bugs and lets your class work in `const` contexts.

---

## 6. `class` vs `struct`

They are the same feature, with one defaults difference:

| | `class` | `struct` |
|---|---|---|
| Default access | `private` | `public` |
| Convention | richer types with invariants | passive data bags |

Use `struct` for small POD-like things (`struct Point { int x, y; };`). Use `class` for anything with non-trivial methods or invariants.

---

## 7. Operator Overloading: The Big Picture

C++ lets you give meaning to built-in operators (`+`, `==`, `<<`, `[]`, etc.) for your own types. Two forms:

| Form | Looks like | Use for |
|---|---|---|
| Member function | `T T::operator@(args)` | Operators whose left side is **always** an object of this class (e.g., `[]`, `+=`, comparison). The implicit `this` is the left operand. |
| Non-member (free) function | `T operator@(L, R)` | Operators where the left side is **not** the class (e.g., `out << s` — the left operand is `ostream`). Often declared `friend` to access private members. |

Rules of thumb:
- `<<` and `>>` **must** be non-member functions.
- `=`, `[]`, `()`, `->` **must** be member functions.
- `==`, `<`, etc. — either works. Member is cleanest.

---

## 8. Comparison Operators

For HW2 `Coins` the language can synthesize equality:

```cpp
class Coins {
public:
    bool operator==(const Coins& other) const = default;   // member-wise compare
};
```

For HW3 `String`, comparison delegates to the static helper `strcmp`:

```cpp
bool String::operator==(const String& s) const { return strcmp(buf, s.buf) == 0; }
bool String::operator!=(const String& s) const { return strcmp(buf, s.buf) != 0; }
bool String::operator<(const String& s)  const { return strcmp(buf, s.buf) <  0; }
bool String::operator>(const String& s)  const { return strcmp(buf, s.buf) >  0; }
bool String::operator<=(const String& s) const { return strcmp(buf, s.buf) <= 0; }
bool String::operator>=(const String& s) const { return strcmp(buf, s.buf) >= 0; }
```

`strcmp` returns negative / zero / positive — same contract as `memcmp` and the future `<=>`.

In C++20 you can collapse all six into one `<=>` (spaceship) operator. We defer that to Guide 04 (HW5 uses it).

---

## 9. Stream Operators

A class becomes printable by overloading `operator<<` as a **non-member function** taking an `ostream&` and your class. It returns the stream so prints can chain.

```cpp
// Pattern: forward to a const print() member, return the stream.
void String::print(std::ostream& out) const { out << buf; }

std::ostream& operator<<(std::ostream& out, const String& s) {
    s.print(out);                            // delegate to the class
    return out;                              // allow chaining: cout << s << '\n'
}
```

For input the symmetric pattern:

```cpp
void String::read(std::istream& in) { in >> buf; }   // reads one word

std::istream& operator>>(std::istream& in, String& s) {  // s is non-const (modified)
    s.read(in);
    return in;
}
```

Use `const T&` for `<<` (we are not modifying the object) and `T&` for `>>` (we are).

---

## 10. Indexing: `operator[]`

Returns a **reference** so the result can sit on the left-hand side (`s[0] = 'H';`).

```cpp
char& String::operator[](int index) {
    if (in_bounds(index)) return buf[index];
    std::cout << "ERROR";
    return buf[0];                           // sentinel; HW3 does not throw
}
```

A `const` overload returns a `const char&` so indexing into a `const String` is allowed:

```cpp
const char& String::operator[](int index) const { ... }
```

`in_bounds(i)` is a private helper that returns `i >= 0 && i < strlen(buf)`.

---

## 11. Concatenation: `operator+` and `operator+=`

`operator+` returns a **new** `String`; `operator+=` modifies `*this` and returns it by reference.

```cpp
String String::operator+(const String& s) const {        // returns a fresh String
    char temp[MAXLEN];
    strcpy(temp, buf);                                   // copy left half
    strncat(temp, s.buf, MAXLEN - 1 - strlen(temp));     // append right half, bounded
    return String(temp);
}

String& String::operator+=(const String& s) {            // mutates *this
    int available = MAXLEN - 1 - strlen(buf);
    if (available <= 0) { std::cout << "ERROR"; return *this; }
    if (&s == this) {                                    // self-append: a += a
        String copy(s);                                   // copy first to avoid clobber
        strncat(buf, copy.buf, available);
    } else {
        strncat(buf, s.buf, available);
    }
    return *this;
}
```

Two patterns to memorize:
- **Return a fresh object by value** when the operator builds a new result (`+`, `reverse`, `operator+`).
- **Return `*this` by reference** when the operator modifies and returns the left operand (`+=`, `=`, `++`).

---

## 12. Assignment: `operator=`

For HW3 (no heap memory) assignment is a one-liner:

```cpp
String& String::operator=(const String& s) {
    strncpy(buf, s.buf, MAXLEN - 1);
    return *this;                            // by reference, for chaining: a = b = c
}
```

Once raw pointers enter (HW4), assignment becomes the **most error-prone** method to write — see Guide 03 for the copy-and-swap idiom.

---

## 13. Static Helper Functions (C-Strings)

HW3 implements its own versions of the C standard `<cstring>` helpers, declared `static` so they belong to the class (no `this`):

| Helper | Behavior |
|---|---|
| `strlen(s)` | count chars up to `'\0'` |
| `strcpy(dest, src)` | copy `src` (incl. `'\0'`) into `dest`; no bounds check |
| `strncpy(dest, src, n)` | copy at most `n` chars; always null-terminate |
| `strcat(dest, src)` | append `src` to the end of `dest` |
| `strncat(dest, src, n)` | append at most `n` chars |
| `strcmp(l, r)` | return < 0 / 0 / > 0 |
| `strncmp(l, r, n)` | first `n` chars only |
| `reverse_cpy(dest, src)` | write reversed `src` into `dest` |
| `strchr(str, c)` | pointer to first `c` in `str`, else `nullptr` |
| `strstr(haystack, needle)` | pointer to first occurrence of `needle`, else `nullptr` |

Each is a flat loop; none knows about `MAXLEN`. Methods of `String` call them. Example:

```cpp
char* String::strcpy(char* dest, const char* src) {
    int i = 0;
    while (src[i] != '\0') { dest[i] = src[i]; ++i; }
    dest[i] = '\0';                          // null terminator!
    return dest;
}

int String::strlen(const char* s) {
    int len = 0;
    while (s[len] != '\0') ++len;
    return len;
}
```

**Two rules** the slides hammer on:
1. Static helpers are **general purpose** — no error checking, no class knowledge. They will be reused in HW4.
2. Public methods do error checking, then **call helpers** to do the work. Don't duplicate logic in the method body.

---

## 14. TDD: How to Build a Class Like `String`

The Strings slide deck lays out a Test-Driven Development cycle. For each public method:

1. Pick a public method, e.g. `reverse()`.
2. Write a nominal "happy-path" test in `student_gtests.cpp`.
3. Identify a helper you can write (e.g. `reverse_cpy(dest, src)`).
4. Write the public method using only the helper.
5. Write the helper.
6. Write a test for the helper, run it. Fix until green.
7. Run the public-method test. Fix until green. **Commit after each cycle.**

Why this works: the helpers are tiny and easy to verify; the public method is a one-line composition. When a test fails, the bug is localized.

Example: `reverse()` written in this style:

```cpp
// Helper — no error checking, no class knowledge
void String::reverse_cpy(char* dest, const char* src) {
    int len = strlen(src);
    for (int i = 0; i < len; ++i)
        dest[i] = src[len - 1 - i];
    dest[len] = '\0';
}

// Public method — one line of real work
String String::reverse() const {
    char temp[MAXLEN];
    reverse_cpy(temp, buf);
    return String(temp);
}
```

---

## 15. HW2 / HW3 Pattern Quick Reference

### HW2: `Coins` — class + `operator==` + `operator<<`

```cpp
class Coins {
public:
    Coins(int q, int d, int n, int p);
    void deposit_coins(Coins& other);                   // mutates both
    bool has_exact_change_for_coins(const Coins& c) const;
    Coins extract_exact_change(const Coins& c);
    int  total_value_in_cents() const;
    void print(std::ostream& out) const;
    bool operator==(const Coins& other) const = default;
private:
    int quarters, dimes, nickels, pennies;
};

std::ostream& operator<<(std::ostream& out, const Coins& c) {
    c.print(out);  return out;
}
```

`Coins::Coins(int q,int d,int n,int p) : quarters(q), dimes(d), nickels(n), pennies(p) {}` — every member initialized via the init list.

### HW3: full `String` skeleton

```cpp
class String {
public:
    explicit String(const char* s = "");
    String(const String& s);                  // copy constructor
    String& operator=(const String& s);       // copy assignment

    char& operator[](int i);
    int   size() const;

    String reverse() const;
    int    indexOf(char c) const;
    int    indexOf(const String& s) const;

    bool operator==(const String& s) const;
    bool operator!=(const String& s) const;
    bool operator< (const String& s) const;
    bool operator> (const String& s) const;
    bool operator<=(const String& s) const;
    bool operator>=(const String& s) const;

    String  operator+(const String& s) const;
    String& operator+=(const String& s);

    void print(std::ostream& out) const;
    void read(std::istream& in);

    ~String();

    // public for the autograder; conceptually private
    static int   strlen(const char* s);
    static char* strcpy(char* dest, const char* src);
    /* ... other helpers ... */
private:
    char buf[MAXLEN];                          // fixed buffer — no heap yet
};
```

The two operators `<<` and `>>` are non-member functions declared **outside** the class:

```cpp
std::ostream& operator<<(std::ostream& out, const String& s);
std::istream& operator>>(std::istream& in,  String& s);
```

### Copy-construct-then-assign idiom (preview)

In HW3, copy assignment is trivial because the buffer is fixed-size. In HW4 (next guide), this same `operator=` becomes the most subtle method in the class — that's where `String& operator=(String s) { swap(s); return *this; }` enters.

---

## 16. Key Rules to Remember

- **`class` defaults to `private:`**; expose only what callers need.
- **Constructors use an initializer list** (`: re(r), im(i)`). It's faster, and required for `const` / reference / no-default-ctor members.
- **`explicit` on single-argument constructors** prevents silent conversions.
- **Every method that doesn't modify `*this` is `const`.** Pass non-trivial parameters as `const T&`.
- **Choose member vs non-member operator** by what sits on the **left**:
  - Left is the class → member (`bool operator==(const String& s) const`).
  - Left is something else (`ostream`) → non-member.
- **`operator<<` returns the stream by reference** so prints chain.
- **`operator[]` returns a reference** so indexing can be assigned to.
- **`operator+` returns a fresh object**; **`operator+=` returns `*this` by reference**.
- **Use static helpers** to do the low-level work; public methods compose helpers and do error checking.
- **TDD beats winging it.** Write nominal test → invent helper → implement → test helper → test public method → commit.
- **HW3 has no heap memory** (everything fits in `char buf[MAXLEN]`), so the destructor is empty and the Rule of Three is "trivially" satisfied. HW4 changes that — see Guide 03.

Next guide: **03 — Memory Management and the Big Five** introduces raw pointers, `new`/`delete`, the Rule of Three/Five/Zero, and the copy-and-swap idiom on the HW4 `String`.
