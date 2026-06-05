# ICS 45C — Linked Lists and Recursion
> Covers the Linked List slide deck + HW5 (`String` re-implemented on a singly-linked list of chars, plus the `<=>` spaceship operator and namespaces)

A linked list is the second standard way to lay out a sequence in memory (the first was a contiguous array, Guide 03). It teaches the same Rule-of-Five obligations on a very different shape of memory.

---

## Table of Contents
1. [Why a Linked List?](#1-why-a-linked-list)
2. [The `Node` Struct](#2-the-node-struct)
3. [Traversing a Linked List](#3-traversing-a-linked-list)
4. [Inserting at the Front](#4-inserting-at-the-front)
5. [Removing a Node](#5-removing-a-node)
6. [Namespaces](#6-namespaces)
7. [The `list::` Helper Library (HW5)](#7-the-list-helper-library-hw5)
8. [Recursion Basics](#8-recursion-basics)
9. [Memory Ownership in Linked Structures](#9-memory-ownership-in-linked-structures)
10. [The `switch` Statement](#10-the-switch-statement)
11. [The HW5 `String` Class on a List](#11-the-hw5-string-class-on-a-list)
12. [The `<=>` Spaceship Operator](#12-the--spaceship-operator)
13. [HW5 Pattern Quick Reference](#13-hw5-pattern-quick-reference)
14. [Key Rules to Remember](#14-key-rules-to-remember)

---

## 1. Why a Linked List?

An array is one big object — fast random access (`a[i]`), but you can't grow it once allocated. A linked list is many small objects chained by pointers — slow random access, but you can splice nodes in or out anywhere in O(1) once you have a pointer to the spot.

| Property | Array | Linked List |
|---|---|---|
| Memory layout | one contiguous block | many small heap blocks |
| Index `a[i]` | O(1) | O(N) (walk from head) |
| Grow | reallocate + copy (expensive) | allocate one node, splice in |
| Insert in middle | shift everything right | rewire one pointer |
| Cache friendliness | excellent | poor (nodes scattered) |

A linked list **should only be used as the implementation of a class**, the same advice as raw arrays.

---

## 2. The `Node` Struct

A singly-linked list (SLL) is a chain of nodes. Each node holds one element plus a pointer to the next node. `nullptr` marks the end.

```cpp
namespace list {                               // see §6
    struct Node {
        char  data;                            // payload — one char here
        Node* next;                            // pointer to next node, or nullptr
    };
}
```

A whole list is represented by a single pointer to the first node, conventionally called `head`. An empty list has `head == nullptr`.

```
head → ['H' | *] → ['i' | *] → ['!' | nullptr]
```

---

## 3. Traversing a Linked List

The canonical loop:

```cpp
for (Node* p = head; p != nullptr; p = p->next) {
    std::cout << p->data;
}
```

`p->next` is shorthand for `(*p).next`. The arrow dereferences and selects a member in one step. Some quick rules:

- Start at `head`.
- Stop when `p == nullptr` (you've walked off the end).
- Advance with `p = p->next` (move the pointer, don't modify the node).

**Do not use `head` as the loop variable** — you would lose track of the list's first node:

```cpp
// DON'T:
while (head) { ... ; head = head->next; }      // head is gone after the loop
```

---

## 4. Inserting at the Front

The cheapest insertion is at the head:

```cpp
head = new Node{'C', head};
```

Read right-to-left: make a fresh node with `data='C'` whose `next` field is the old `head`. Then re-aim `head` at this new node. O(1) and trivial.

A push method:

```cpp
void push(char c) {
    head = new Node{c, head};                  // prepend
}
```

For HW5, `list::from_string("Hello")` builds five nodes — but in document order, not prepended in reverse. The implementation walks `s` left to right, appending each character:

```cpp
Node* from_string(const char* s) {
    if (!s || *s == '\0') return nullptr;
    Node* head = new Node{*s, nullptr};
    Node* curr = head;
    for (const char* p = s + 1; *p != '\0'; ++p) {
        curr->next = new Node{*p, nullptr};
        curr = curr->next;
    }
    return head;
}
```

---

## 5. Removing a Node

**Removing the head** is the easy case:

```cpp
char pop() {
    Node* tmp = head;                          // save the old head
    char  ret = tmp->data;
    head = head->next;                         // unlink
    delete tmp;                                // free
    return ret;
}
```

You must `delete` the unlinked node, otherwise its heap memory leaks forever.

**Removing a node from the middle** requires a pointer to the **predecessor** so you can re-aim `prev->next` to skip the doomed node:

```cpp
Node* prev = find_prev(head, 'X');
Node* cur  = prev->next;
prev->next = cur->next;                        // skip cur
delete cur;
```

---

## 6. Namespaces

A **namespace** groups names so they don't collide with names elsewhere. The whole standard library lives in `std::` — `std::cout`, `std::vector`, `std::string`.

```cpp
namespace list {                               // open a namespace
    struct Node { char data; Node* next; };
    Node* copy(Node* head);
    void  free(Node* head);
}                                              // no semicolon

// outside, refer to it via list:: prefix
list::Node* h = list::copy(other);
list::free(h);
```

Three ways to use names from a namespace:

```cpp
list::Node* h;                                 // qualified — explicit
using list::Node;  Node* h;                    // single-name using
using namespace list;                          // open the whole namespace (risky in headers)
```

The HW5 design tucks all the low-level list operations into `namespace list` so the `String` class can call them as `list::copy`, `list::free`, etc. without polluting the global namespace. Multiple `.cpp`/`.hpp` files can reopen the same namespace and add to it — namespaces are cumulative.

---

## 7. The `list::` Helper Library (HW5)

HW5 ships a complete SLL helper namespace declared in `list.hpp`:

```cpp
namespace list {
    struct Node { char data; Node* next; };

    Node* from_string(const char* s);                   // build list from C-string
    void  free(Node* head);                              // delete every node
    Node* copy(Node* head);                              // deep copy
    int   length(Node* head);                            // count nodes

    int   compare(Node* lhs, Node* rhs);                 // strcmp-style
    int   compare(Node* lhs, Node* rhs, int n);          // strncmp-style

    Node* reverse(Node* head);                           // returns reversed copy
    Node* append(Node* lhs, Node* rhs);                  // returns lhs ++ rhs

    int   index(Node* head, Node* target);               // position of target
    Node* find_char(Node* head, char c);                 // first match
    Node* find_list(Node* hay, Node* needle);            // substring match

    Node* nth(Node* head, int n);                        // pointer to nth node
    Node* last(Node* head);                              // last node
    void  print(std::ostream& out, Node* head);
}
```

Like the static helpers in HW3, these are general-purpose and do no error checking. `String` methods compose them.

A few representative implementations:

```cpp
int length(Node* head) {                                 // iterative — clean & fast
    int n = 0;
    for (Node* p = head; p; p = p->next) ++n;
    return n;
}

Node* copy(Node* head) {                                 // deep clone, iterative
    if (!head) return nullptr;
    Node* new_head = new Node{head->data, nullptr};
    Node* tail = new_head;
    for (Node* src = head->next; src; src = src->next) {
        tail->next = new Node{src->data, nullptr};
        tail = tail->next;
    }
    return new_head;
}

void free(Node* head) {                                  // delete each node
    while (head) {
        Node* next = head->next;                          // capture next BEFORE delete
        delete head;
        head = next;
    }
}

Node* reverse(Node* head) {                              // prepend-each = reversed result
    Node* result = nullptr;
    for (Node* p = head; p; p = p->next)
        result = new Node{p->data, result};
    return result;
}

int compare(Node* lhs, Node* rhs) {                      // strcmp for lists
    while (lhs && rhs) {
        if (lhs->data != rhs->data)
            return (unsigned char)lhs->data - (unsigned char)rhs->data;
        lhs = lhs->next; rhs = rhs->next;
    }
    if (lhs) return  1;                                  // rhs ended first → lhs > rhs
    if (rhs) return -1;
    return 0;
}
```

**Two gotchas appear constantly:**
- **Capture `next` before `delete`**: otherwise you dereference a freed node to find the rest of the list.
- **One pass**, not two. Walking the list twice is wasteful and tempts off-by-one bugs.

---

## 8. Recursion Basics

A recursive function calls itself on a smaller problem. It needs:
- A **base case** that terminates without recursing.
- A **recursive case** that reduces the problem and calls itself.

```cpp
int factorial(int n) {
    if (n == 0) return 1;                                // base
    return n * factorial(n - 1);                          // recursive
}
```

Linked lists are recursive structures (a list is either empty or "a node + a list"), so recursion fits them naturally:

```cpp
int length(Node* head) {
    if (!head) return 0;                                 // base: empty list
    return 1 + length(head->next);                       // recursive
}

Node* copy(Node* head) {
    if (!head) return nullptr;
    return new Node{head->data, copy(head->next)};
}
```

Both versions are correct; iteration is usually preferred in HW5 to avoid deep recursion blowing the stack for long lists. But recursion is cleaner for teaching and for some structures (binary trees, parsers).

---

## 9. Memory Ownership in Linked Structures

Each node lives on the heap (born via `new`). Who is responsible for `delete`ing them?

The slides' design choice:
- **Node**: does **not** own its `next`. (`Node`'s destructor doesn't recurse.)
- **`String`**: owns all the nodes reachable from `head`. When `String` dies, `list::free(head)` is called.

This means **two `String`s must not share nodes**. Sharing would make destruction a nightmare — who frees what? The discipline is: every `String` has its own deep copy.

This is exactly RAII (Guide 03), specialized to a linked structure:
- **Constructor** acquires the list (`from_string`, `copy`).
- **Destructor** releases the list (`free`).
- **Copy ctor / copy assignment** deep-copy (`copy`).
- **Move ctor / move assignment** steal `head` and `nullptr` the source.

The Rule of Five becomes:

| Special member | List analogue |
|---|---|
| Default ctor | `head = nullptr` |
| Copy ctor | `head = list::copy(s.head)` |
| Copy assignment | free old + deep copy new |
| Move ctor | `head = s.head; s.head = nullptr` |
| Move assignment | `swap(s)` |
| Destructor | `list::free(head)` |

---

## 10. The `switch` Statement

A `switch` is a multi-way branch on a discrete value (`int`, `char`, or `enum`). It's O(1) (the compiler builds a jump table) and is very fast.

```cpp
switch (c) {
    case 'A':
    case 'B':
        std::cout << "Got an A or B";
        break;                                            // exit the switch
    case 'C':
        std::cout << "Got a C";
        break;
    default:
        std::cout << "oops!";
}
```

Key rules:
- The `switch` expression must be discrete (no `double` or `std::string`).
- Without `break`, control **falls through** to the next case — sometimes wanted (`'A'` and `'B'` share a body above), more often a bug.
- `default:` handles everything that didn't match.

`switch` doesn't appear directly in HW5, but it's covered in the Linked List slides as a fast alternative to long `if/else if` chains over `int`/`char`/`enum` values.

---

## 11. The HW5 `String` Class on a List

```cpp
#include "list.hpp"
#include <compare>                                       // for std::strong_ordering

class String {
    list::Node* head;                                     // owns the chain

    explicit String(list::Node* h) : head(h) {}           // PRIVATE — takes ownership
public:
    explicit String(const char* s = "");                  // build from C-string
    String(const String& s);                              // deep copy
    String(String&& s);                                   // steal head
    String& operator=(const String& s);                   // free + deep copy
    String& operator=(String&& s);                        // swap
    void    swap(String& s);

    ~String();                                            // list::free(head)

    int  size() const;                                    // list::length
    char operator[](int i) const;                         // list::nth(head, i)->data
    bool in_bounds(int i) const;

    String reverse() const;                                // return String(list::reverse(head))
    int    indexOf(char c) const;
    int    indexOf(const String& s) const;

    bool                  operator==(const String& s) const;
    std::strong_ordering  operator<=>(const String& s) const;  // covers <, >, <=, >=

    String  operator+(const String& s) const;
    String& operator+=(const String& s);

    void print(std::ostream& out) const;
    void read(std::istream& in);
};
```

Selected implementations show how cleanly the helpers compose into a class:

```cpp
String::String(const char* s)        { head = list::from_string(s); }
String::String(const String& s)      { head = list::copy(s.head); }
String::String(String&& s)           { head = s.head; s.head = nullptr; }
String::~String()                    { list::free(head); }

void   String::swap(String& s)        { auto* t = head; head = s.head; s.head = t; }

String& String::operator=(const String& s) {              // classic — not copy-and-swap
    if (this != &s) {                                      // self-assignment guard
        list::free(head);
        head = list::copy(s.head);
    }
    return *this;
}
String& String::operator=(String&& s) { swap(s); return *this; }

int    String::size() const           { return list::length(head); }
char   String::operator[](int i) const{ return list::nth(head, i)->data; }
String String::reverse() const        { return String(list::reverse(head)); }

String String::operator+(const String& s) const {
    return String(list::append(head, s.head));            // private ctor takes the list
}
```

Notice the role of `explicit String(list::Node* h)` — a **private constructor** that lets `String`'s own methods build a `String` directly around a freshly-built list, with no extra copy. Same idea as the `String(int length)` ctor in HW4.

The `read` helper is also instructive:

```cpp
void String::read(std::istream& in) {
    char buf[1024];
    in >> buf;                                            // C-string buffer
    list::free(head);                                     // release the old list
    head = list::from_string(buf);                         // build a new one
}
```

---

## 12. The `<=>` Spaceship Operator

C++20 provides one operator that defines **all six** comparisons at once: `operator<=>`. It returns an ordering value with three possible states.

```cpp
#include <compare>

class String {
public:
    bool                 operator==(const String& s) const;     // still write this
    std::strong_ordering operator<=>(const String& s) const;     // gives <, >, <=, >=
};
```

The three ordering types:

| Type | Semantics |
|---|---|
| `std::strong_ordering`  | totally ordered, equal-iff-substitutable (`int`-like) |
| `std::weak_ordering`    | totally ordered, "equal" may mean "equivalent" |
| `std::partial_ordering` | comparable values may also be **incomparable** (e.g., NaN) |

Each has constants `less`, `greater`, `equal` (`equivalent` for weak, `unordered` for partial). They convert to `bool` via the obvious comparisons (`< 0`, `> 0`, `== 0`).

The HW5 `String` defines `<=>` by spaceshipping the list-compare result against zero:

```cpp
std::strong_ordering String::operator<=>(const String& s) const {
    return list::compare(head, s.head) <=> 0;
}
```

With `<=>` defined, the compiler synthesizes `<`, `>`, `<=`, `>=` for free. You still write `==` separately (a deliberate split — equality can often be implemented cheaper than ordering, e.g. via `size()` comparison first).

Earlier (HW2) we used `bool operator==(const Coins&) const = default;` — that's the **defaulted** form. For `<=>` you can also default it (`auto operator<=>(...) const = default;`) and the compiler will lexicographically compare members. HW5 writes it by hand because the ordering is defined by `list::compare`, not by member layout.

---

## 13. HW5 Pattern Quick Reference

### Building, copying, freeing — every `String` method composes `list::` helpers

```cpp
String::String(const char* s)      { head = list::from_string(s); }
String::String(const String& s)    { head = list::copy(s.head); }
String::~String()                  { list::free(head); }
```

### Reversing returns a fresh String from a freshly-reversed list

```cpp
String String::reverse() const {
    return String(list::reverse(head));         // private ctor takes ownership of new list
}
```

### `operator+` returns the appended list directly

```cpp
String String::operator+(const String& s) const {
    return String(list::append(head, s.head));   // append builds a fresh list
}
```

### `operator+=` is the only in-place edit

```cpp
String& String::operator+=(const String& s) {
    if (!s.head) return *this;                   // appending nothing → no-op
    list::Node* tail_copy = list::copy(s.head);
    if (!head) head = tail_copy;
    else       list::last(head)->next = tail_copy;
    return *this;
}
```

### All six comparisons via `<=>`

```cpp
bool                 operator==(const String& s) const { return list::compare(head, s.head) == 0; }
std::strong_ordering operator<=>(const String& s) const { return list::compare(head, s.head) <=> 0; }
```

### Move ops steal the head

```cpp
String::String(String&& s)           { head = s.head; s.head = nullptr; }
String& String::operator=(String&& s) { swap(s); return *this; }
```

---

## 14. Key Rules to Remember

- A linked list is a chain of heap-allocated nodes. **Head pointer = the whole list.** `head == nullptr` means empty.
- The traversal idiom is `for (Node* p = head; p != nullptr; p = p->next)`. **Don't touch `head` in the loop.**
- **Insert at front in O(1)**: `head = new Node{data, head};`.
- **When deleting, save `next` first**: `Node* n = p->next; delete p; p = n;`.
- **Namespaces** scope helper names. `list::copy`, `list::free`, etc. They're cumulative across files.
- **Memory ownership belongs to the class**, not the nodes. `String` owns the whole chain; `Node` doesn't `free` its `next`.
- **No sharing of subchains between objects.** Every `String` has its own deep copy.
- The Rule of Five maps to: copy = `list::copy`, free = `list::free`, move = steal `head` and null source.
- **`<=>`** (`#include <compare>`) gives you `<`, `>`, `<=`, `>=` at once. Choose `strong_ordering` for `int`-like total orders.
- **`==` is still written separately** even when `<=>` is defined.
- Recursion on lists is natural and short; iteration is usually safer to avoid deep call stacks.
- **`switch` is for discrete values** (`int`, `char`, `enum`). Fall-through is intentional only when you want it.
- A **private constructor that takes a `list::Node*`** lets `String` methods build new strings from freshly-constructed lists without extra copies — same idea as `String(int length)` in HW4.

Next guide: **05 — Inheritance and Polymorphism** moves from a single class to a family of classes (`Shape`, `Circle`, `Rectangle`, ...).
