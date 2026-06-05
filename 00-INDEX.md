# ICS 45C — Final Exam Study Guide Index
> Master index for all study guides, mapped to topics, homeworks, and Topic 10's review checklist

---

## The Guides

| # | Guide | What it covers | Homeworks folded in |
|---|---|---|---|
| 01 | [C++ Foundations](01-cpp-foundations.md) | Built-in types, I/O streams, control flow, functions, references, fixed C arrays, char-as-index, `.hpp`/`.cpp` split, compilation pipeline | HW1 (intro), HW2 *Coins* basics |
| 02 | [Classes and Operator Overloading](02-classes-and-operator-overloading.md) | Classes vs structs, ctor/dtor, `this`, init lists, `const`, all the operator overloads, `<<`/`>>`, static C-string helpers, TDD | HW2 *Coins* (fully), HW3 *String* (fixed buffer, fully) |
| 03 | [Memory Management and the Big Five](03-memory-management-and-the-big-five.md) | Pointers vs references, `new`/`delete`, RAII, Rule of Three/Five/Zero, copy-and-swap, move semantics, the `AllocationTracker` | HW4 *String* (raw pointer), HW4 `AllocationTracker` |
| 04 | [Linked Lists and Recursion](04-linked-lists-and-recursion.md) | `Node`, traversal, push/pop, recursion, memory ownership in SLLs, namespaces, the `<=>` spaceship operator, `switch` | HW5 *String* on `list::Node` (fully) |
| 05 | [Inheritance and Polymorphism](05-inheritance-and-polymorphism.md) | Is-a vs has-a, `public`/`protected`/`private` inheritance, static vs dynamic binding, `virtual`/`override`, pure virtual & abstract bases, virtual destructors, `clone()`, multiple inheritance | HW6 *Shape*/*Picture* (fully) |
| 06 | [Templates, Lambdas, Exceptions, Namespaces](06-templates-lambdas-exceptions-namespaces.md) | Lambdas + capture modes, exceptions + the std hierarchy, function & class templates, specialization, member templates, type aliases | HW7 *Array<T>*/*Matrix<T>* (fully) |
| 07 | [The STL](07-the-stl.md) | Containers, iterators, adapters, algorithms, ranges (intro), string/file streams, iomanip, STL exceptions | HW8 *process_numbers*, *mapset*, *compute_grades* |
| 08 | [Ranges, Views, Smart Pointers, and Custom Iterators](08-ranges-views-smart-pointers-and-custom-iterators.md) | `\|`-composable views, `istream_view`, `shared_ptr`/`unique_ptr`, iterator categories/tags/traits, building forward and random-access iterators, C++20 concepts | HW9 *SetList*, *MapArray*, ranges-based mapset (fully) |

---

## Final Exam Prep Overview — What to Study

This list maps the "Topic 10 — Final Review" lecture bullets to the guide that covers each item. **Bold** items are the highest-priority material (the topics most likely to show up).

### Foundation (skim — should be reflex by now)
- **C++ I/O**: `cin`, `cout`, `cerr`, `<<` `>>`, `endl`, `setw(n)`, `left`/`right` — **Guide 01 §5; Guide 07 §10**
- **Iteration**: `for` (classic + range-for), `while`, `do-while` — Guide 01 §6
- **switch / case / default / break** — Guide 04 §10
- **Operator precedence and mixed arithmetic** — Guide 01 §4
- **Declarations vs definitions; preprocessor → compiler → linker → loader** — Guide 01 §13

### Pointers, references, and lifetimes (heavily tested)
- **References vs pointers vs objects** — **Guide 03 §1**
- **`new`/`delete`/`new[]`/`delete[]`** and the matching rules — **Guide 03 §2-3**
- **`const` with pointers and references; `const this`** — Guide 02 §5, Guide 03 §1
- **`constexpr` vs `const`** — Guide 01 §3
- Storage areas: static, stack, heap — Guide 01 §3

### Classes (the heart of the course)
- **The special members** (default ctor, copy ctor & copy assign, move ctor & move assign, dtor) — when do they get called? what are the defaults? when do you write or `=delete` them? — **Guide 03 §5-9**
- **Operator overloading**: member vs non-member; `=`, `==`/`!=`/`<`/`>`/`<=`/`>=`/`<=>`, `+`/`-`/`*`/`/`/`%`, `()`, `*`/`->`, `[]` — **Guide 02 §7-13, Guide 04 §12**
- **`<<`/`>>` non-member operators** — Guide 02 §9
- **`friend` functions** — Guide 06 §7, Guide 07 §10 (the in-class `friend swap` and friend `operator+`)
- **Inheritance & polymorphism**: virtual, pure virtual, virtual destructor, `clone()`, slicing — **Guide 05 §5-9**

### Templates and the STL
- **What can a template parameter be?** `typename`, non-type `int N` — Guide 06 §5
- **What operations does the template body assume of `T`?** — Guide 06 §5
- **What special members are required of `T`?** ctor(s), dtor, assign — Guide 06 §7
- **Containers, iterators, algorithms** — **Guide 07 §3-6**
- **`auto`** in declarations and range-for, capture modes for lambdas — Guide 06 §1-2
- **Smart pointers**: `unique_ptr`, `shared_ptr` — purpose, advantages over raw pointers — **Guide 07 §4-6**

### Always-on reminders (from Topic 10's "Remember" slide)
- "Three areas of memory" — Guide 01 §3
- "Every object has a lifetime" — Guide 02 §3-4
- "Pass by reference has two purposes" (modify caller, avoid copy) — Guide 01 §8
- "If you don't use it, …" — i.e., the **Rule of Zero** (don't write what you don't need) — Guide 03 §12

---

## Quick Lookup Table — HW → Guide

| HW | Primary guide | Also touches |
|---|---|---|
| HW1 (knots, letter_count, Stack) | 01 | — |
| HW2 (Coins, word_count) | 02 | 01, 07 |
| HW3 (String, fixed buffer) | 02 | — |
| HW4 (String w/ raw pointer, AllocationTracker) | 03 | — |
| HW5 (String on SLL, `<=>`) | 04 | 03 (RAII), 02 (operators) |
| HW6 (Shape hierarchy, Picture) | 05 | 03 (RAII), 04 (SLL inside Picture) |
| HW7 (`Array<T>`, `Matrix<T>`) | 06 | 03 (copy-and-swap), 02 (operators) |
| HW8 (process_numbers, mapset, compute_grades) | **07** | 02 (operators), 06 (lambdas, exceptions) |
| HW9 (SetList, MapArray, ranges-based mapset) | 08 | 07 (STL), 06 (templates), 04 (`<=>`) |

---

## Quick Lookup Table — Lecture Deck → Guide

| Slide deck | Primary guide |
|---|---|
| C++ Tour | 01 |
| Arrays | 01 |
| Strings | 02 |
| Classes | 02 |
| Dyn Array | 03 |
| Linked List | 04 |
| Topic 6 (Inheritance & Polymorphism) | 05 |
| Topic 7 (Lambdas/Exceptions/Templates/Namespaces) | 06 |
| Topic 8 (STL) | 07 |
| Topic 9 (Ranges/Smart Pointers/Custom Iterators) | 08 |
| Topic 10 (Final Review) | this index |

---

## Highest-Priority Material (read these first if time is short)

1. **Guide 03 — Memory Management and the Big Five.** Pointer rules, RAII, copy/move ctor/assignment, copy-and-swap. Underlies HW4-7.
2. **Guide 05 — Inheritance and Polymorphism.** Virtual/override/pure virtual/clone/virtual dtor. Lots of conceptual questions here.
3. **Guide 07 — The STL.** Containers, iterators, algorithms, adapters, ranges intro.
4. **Guide 02 — Classes and Operator Overloading.** The basis for everything in the class half of the course.
5. **Guide 06 — Templates, Lambdas, Exceptions, Namespaces.** Templates are short to read, easy to test on.

After those: Guide 04 (linked lists + `<=>`), Guide 08 (ranges + custom iterators), Guide 01 (foundations).

---

## Notes / Assumptions

A few judgment calls during authoring:
- **HW1 and HW2 (the introductory homeworks) get brief coverage in Guide 01 and Guide 02** rather than dedicated guides — per the assignment brief.
- **The provided STL guide is included verbatim as Guide 07** (it was authored separately as the template for the rest of this set).
- **Topic 7 (Lambdas, Exceptions, Templates, Namespaces) is bundled into Guide 06** because the topics are interrelated and HW7 exercises them all together. Splitting them would have produced four small guides that mostly point at each other.
- **Topic 9 and HW9 are combined into Guide 08** because HW9 ties ranges/views to the custom iterator work — the smart-pointer / range / custom-iterator concepts are easier to teach together than apart.
- **Topic 10 (Final Review)** is folded into this index as the "What to Study" overview above, mapping each Topic 10 bullet back to the guide that covers it.
- **Slide deck contents were extracted via `python-pptx`** and stored under `.slide_text/` for reference. Reading those files alongside this guide set will not surface anything not already covered.
- **`<<` and `>>` operators** are introduced in Guide 02 (since they're operator overloads), even though HW2's `Coins` is the first homework to require them — the same logic applies to every later String/Array/Matrix/Student class.
- **The `AllocationTracker` PMR machinery** in HW4 is described at the "what does it observe" level in Guide 03 §14 — the internal `std::pmr::memory_resource` plumbing is beyond exam-relevant detail.
