# Practice Quiz 04 — Linked Lists and Recursion

**Covers:** Guide 04 (Linked List slides — `Node`, traversal, push/pop, namespace `list`, recursion, memory ownership, `<=>` spaceship operator, `switch`).

**Questions:** 15

---

## Questions

### Question 1
A singly-linked list of `char` has the shape:
```cpp
namespace list {
    struct Node { char data; Node* next; };
}
```
An empty list is conventionally represented by:
- A. `Node{}` (a default-constructed node)
- B. A `Node` with `data == '\0'`
- C. A `Node*` set to `nullptr`
- D. `Node* head = new Node{};`
- E. none of the other choices

### Question 2
What does this print? (assume `Node` is the SLL node above)
```cpp
list::Node* head = new list::Node{'A', new list::Node{'B', new list::Node{'C', nullptr}}};
for (list::Node* p = head; p; p = p->next)
    std::cout << p->data;
```
- A. `A`
- B. `ABC`
- C. `CBA`
- D. infinite loop
- E. none of the other choices

### Question 3
What is wrong with this attempt at `length`?
```cpp
int length(list::Node* head) {
    int n = 0;
    while (head) {
        ++n;
        head = head->next;
    }
    return n;
}
```
- A. It does not handle empty lists.
- B. It modifies the caller's `head` pointer.
- C. There is no bug — this is a correct iterative `length`.
- D. It leaks every node it visits.
- E. It will segfault on the last node.

### Question 4
Which is the cheapest place to insert a new node into a singly-linked list that has only a `head` pointer?
- A. After the head node
- B. At the tail
- C. At the head
- D. In the middle, alphabetically
- E. They are all the same cost.

### Question 5
SELECT ALL true about `namespace list`.
- A. Multiple `.hpp`/`.cpp` files may all reopen and add to `namespace list`.
- B. `list::copy` and `std::list` would collide if both were `using namespace`'d.
- C. The functions inside `namespace list` automatically have access to `String`'s private members.
- D. Namespaces nest: `namespace list { namespace detail { ... } }` is allowed.
- E. A `using namespace list;` inside a header file is a recommended best practice.

### Question 6
What does this print?
```cpp
int length(list::Node* head) {
    if (!head) return 0;
    return 1 + length(head->next);
}
list::Node* h = new list::Node{'X', new list::Node{'Y', new list::Node{'Z', nullptr}}};
std::cout << length(h);
```
- A. 0
- B. 1
- C. 2
- D. 3
- E. stack overflow

### Question 7
SELECT ALL requirements for a correct recursive function.
- A. At least one base case.
- B. At least one recursive case.
- C. Progress toward the base case.
- D. A `while` loop inside the body.
- E. A conditional (`if` / `?:` / `switch`).

### Question 8
What is wrong with this `free`?
```cpp
void free(list::Node* head) {
    for (list::Node* p = head; p; p = p->next)
        delete p;
}
```
- A. It leaks the head node.
- B. After `delete p`, accessing `p->next` is undefined behavior.
- C. It only frees the first node.
- D. It is correct.
- E. It double-frees the last node.

### Question 9
What does this print?
```cpp
#include <iostream>
struct Node { int v; Node* next; };
int main() {
    Node* h = new Node{1, new Node{2, new Node{3, nullptr}}};
    Node* p = h;
    while (p) {
        p->v += 10;
        p = p->next;
    }
    for (Node* q = h; q; q = q->next)
        std::cout << q->v << ' ';
}
```
- A. `1 2 3`
- B. `11 12 13`
- C. `11 2 3`
- D. compile error
- E. infinite loop

### Question 10
With which two return types can `operator<=>` be declared to provide all six comparisons for a class? SELECT ALL.
- A. `std::strong_ordering`
- B. `bool`
- C. `int`
- D. `std::weak_ordering`
- E. `std::partial_ordering`

### Question 11
When `operator<=>` is declared `= default`, which operators does the compiler synthesize for the class?
- A. Just `<`
- B. `<`, `>`, `<=`, `>=`
- C. `<`, `>`, `<=`, `>=`, and `==`/`!=`
- D. Only `==`
- E. All six explicit comparison operators including `==` (requires `<=>` plus a separate `==`)

### Question 12
What does this `reverse` return when called on `"ABC"`? (You can assume `list::from_string` builds the list `A → B → C → nullptr`.)
```cpp
list::Node* reverse(list::Node* h) {
    list::Node* r = nullptr;
    for (list::Node* p = h; p; p = p->next)
        r = new list::Node{p->data, r};
    return r;
}
```
- A. `A → B → C`
- B. `C → B → A`
- C. `nullptr`
- D. Two interleaved lists
- E. The original list, but with reversed pointers in-place.

### Question 13
SELECT ALL true about ownership in the HW5 `String`-on-SLL design.
- A. `Node` owns its `next` node and frees it in its destructor.
- B. `String` owns the entire chain and frees it via `list::free` in its destructor.
- C. Two `String`s may safely share a sublist of nodes.
- D. The copy ctor performs a deep copy via `list::copy`.
- E. The move ctor steals `head` and nulls out the source.

### Question 14
What is wrong with this code?
```cpp
char c = 'B';
switch (c) {
    case 'A': std::cout << "A"; 
    case 'B': std::cout << "B"; 
    case 'C': std::cout << "C"; 
}
```
- A. `switch` cannot use a `char`.
- B. There is no `default` — compile error.
- C. The cases fall through; output is `BC`, not `B`.
- D. Output is `B`.
- E. None of the cases match.

### Question 15
SELECT ALL idioms that show up in a typical SLL traversal.
- A. `for (Node* p = head; p; p = p->next) { ... }`
- B. `while (head) { /* modify head */; head = head->next; }`
- C. `for (Node* p = head; p->next; p = p->next) { ... }` (one node short)
- D. Capturing `next` before `delete` when freeing nodes.
- E. Using `p == nullptr` as the loop's stop condition.

---

## Answer Key

1. **C** — Empty SLL is just a null head pointer.
2. **B** — Walks the list from A to C, printing each `data`.
3. **C** — The function is correct. `head` is a *local* copy of the pointer (pass-by-value); modifying it doesn't touch the caller.
4. **C** — Inserting at the head is `head = new Node{data, head};` — O(1) with no traversal.
5. **A, B, D** — Namespaces are cumulative across files; name collisions occur on `using`; nesting is allowed. (C: namespaces don't grant access to private class members — `friend` does. E: `using namespace` in a header pollutes every including file — avoid.)
6. **D** — Recursive `length` of a 3-node list returns 3.
7. **A, B, C, E** — A conditional is required to distinguish base from recursive case; loops are NOT required.
8. **B** — After `delete p`, reading `p->next` is undefined behavior. Capture `next` first.
9. **B** — The loop adds 10 to every node's value; second print shows `11 12 13`.
10. **A, D, E** — The three standard ordering types. `bool` and `int` are not orderings.
11. **B** — `<=>` default-synthesizes the four relational operators. **You still write `==` separately** — that's the explicit design choice in C++20.
12. **B** — Each new node is *prepended* to the result, so the result is the input reversed: `C → B → A`.
13. **B, D, E** — String owns the chain; deep copy on construction; move steals. (A: nodes don't free `next`. C: sharing nodes makes destruction ambiguous and is forbidden by design.)
14. **C** — `case 'B':` matches but lacks `break`, so it falls through to `'C':`. Output is `BC`.
15. **A, D, E** — Canonical idioms. (B is dangerous: modifying `head` loses access to the list. C is bug-prone — you intentionally skip the last node and risk null-deref if the list is empty.)
