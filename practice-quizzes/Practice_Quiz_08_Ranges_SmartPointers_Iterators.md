# Practice Quiz 08 — Ranges, Views, Smart Pointers, Custom Iterators

**Covers:** Guide 08 (Topic 9 slides + HW9 — ranges/views with `|`, `istream_view`, `shared_ptr`/`unique_ptr`, iterator categories/tags/traits, building forward & random-access iterators, concepts).

**Questions:** 15

---

## Questions

### Question 1
What is the difference between `std::sort(v.begin(), v.end())` and `std::ranges::sort(v)`?
- A. They are different algorithms; ranges::sort is unstable.
- B. They are the same algorithm; ranges::sort just takes the whole range.
- C. ranges::sort needs an `<algorithm>` include; std::sort doesn't.
- D. ranges::sort is only for `std::ranges::vector`.
- E. std::sort works on any iterator; ranges::sort works only on `std::vector`.

### Question 2
What is the output?
```cpp
#include <iostream>
#include <vector>
#include <ranges>
int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6};
    auto out = v
             | std::views::filter([](int x) { return x % 2 == 0; })
             | std::views::transform([](int x) { return x * 10; });
    for (int x : out) std::cout << x << ' ';
}
```
- A. `10 20 30 40 50 60`
- B. `2 4 6`
- C. `20 40 60`
- D. nothing
- E. compile error

### Question 3
SELECT ALL true about views.
- A. They are lazy — work happens during iteration, not when the view is built.
- B. They own a copy of the underlying data.
- C. They can be composed with `|`.
- D. They invalidate the underlying container when iterated.
- E. They are typically very cheap to construct.

### Question 4
Which smart pointer should you reach for **first** when you need exclusive ownership?
- A. `std::shared_ptr`
- B. `std::unique_ptr`
- C. `std::weak_ptr`
- D. `std::auto_ptr`
- E. raw pointer

### Question 5
SELECT ALL true about `std::shared_ptr`.
- A. It maintains a reference count; the object dies when the count reaches zero.
- B. Copying a `shared_ptr` increments the count.
- C. Two `shared_ptr`s built from the same raw pointer (e.g., `shared_ptr<T> a(p); shared_ptr<T> b(p);`) share one control block.
- D. Cyclic ownership (A → B → A) causes leaks unless `weak_ptr` is used to break the cycle.
- E. `std::make_shared<T>(args)` is preferred over `std::shared_ptr<T>(new T(args))`.

### Question 6
What does this print?
```cpp
#include <iostream>
#include <memory>
int main() {
    auto p = std::make_unique<int>(42);
    auto q = std::move(p);
    std::cout << *q << ' ' << (p == nullptr);
}
```
- A. `42 0`
- B. `42 1`
- C. `0 1`
- D. compile error — `unique_ptr` can't be moved
- E. undefined behavior

### Question 7
SELECT ALL true about `std::unique_ptr`.
- A. Copy construction and copy assignment are deleted.
- B. Ownership transfer is done with `std::move`.
- C. When the `unique_ptr` dies, it `delete`s its managed object.
- D. `std::make_unique<T>(args)` constructs the object and the smart pointer in one step.
- E. Two `unique_ptr`s can share ownership of the same object.

### Question 8
The five iterator traits are the type aliases:
- A. `iterator_category`, `value_type`, `difference_type`, `pointer`, `reference`
- B. `iterator_type`, `value_type`, `size_type`, `pointer`, `reference`
- C. `category`, `type`, `ptr`, `ref`, `diff`
- D. `iterator_category`, `key_type`, `mapped_type`, `pointer`, `reference`
- E. just `iterator_category` and `value_type`

### Question 9
Which iterator category is required by `std::sort`?
- A. Input
- B. Output
- C. Forward
- D. Bidirectional
- E. Random-access

### Question 10
SELECT ALL true about a **forward iterator**.
- A. Supports `*it`, `++it`, `==`.
- B. Supports `--it`.
- C. Supports `it + n` and `it[n]`.
- D. Allows multiple passes over the same range.
- E. Is the iterator category of `std::list`.

### Question 11
What does this `static_assert` do?
```cpp
static_assert(std::random_access_iterator<MapArray<std::string, int>::iterator>);
```
- A. Runtime check, fails with an exception if the iterator is wrong.
- B. Compile-time check that the iterator satisfies the `random_access_iterator` concept; fails the build if it doesn't.
- C. A documentation comment that the compiler ignores.
- D. Disables iterator checks in release builds.
- E. Macro that prints debug output.

### Question 12
Which two operations does a **random-access iterator** support that a forward iterator does not? SELECT TWO.
- A. `*it`
- B. `it + n`
- C. `it1 - it2` (distance)
- D. `++it`
- E. `it == other`

### Question 13
What does this print?
```cpp
#include <iostream>
#include <sstream>
#include <ranges>
int main() {
    std::istringstream ss("apple banana cherry");
    auto words = std::ranges::istream_view<std::string>(ss);
    for (const auto& w : words) std::cout << w.size() << ' ';
}
```
- A. `apple banana cherry`
- B. `5 6 6`
- C. `apple_size banana_size cherry_size`
- D. nothing
- E. compile error

### Question 14
SELECT ALL true about the HW9 `SetList<T>` iterator.
- A. Its `iterator_category` is `std::forward_iterator_tag`.
- B. It maintains a `std::shared_ptr<ListNode>` to the current node.
- C. `end()` is represented by a default-constructed (null) iterator.
- D. It supports `it--`.
- E. It supports `it + n`.

### Question 15
Why might `SetList` use `shared_ptr` for its linked list rather than `unique_ptr`?
- A. `unique_ptr` is more expensive than `shared_ptr`.
- B. Iterators need to *observe* nodes without owning them; `shared_ptr` allows multiple non-owning observers via copies.
- C. `unique_ptr` would force the destructor to recurse the chain and may overflow the stack.
- D. `shared_ptr` allows a `forward_iterator` to copy itself cheaply (the iterator can hold a `shared_ptr` too).
- E. `shared_ptr` automatically detects cycles, making the design safer.

---

## Answer Key

1. **B** — `ranges::sort(v)` is the same algorithm; the ranges API just takes the whole range.
2. **C** — Pipeline: keep evens (`2, 4, 6`), multiply by 10 (`20, 40, 60`). Lazy until the for-loop iterates.
3. **A, C, E** — Standard view facts. (B: views don't own. D: iterating doesn't invalidate.)
4. **B** — Default to `unique_ptr` unless you genuinely need shared ownership.
5. **A, B, D, E** — Standard `shared_ptr` facts. (C: two `shared_ptr`s built from the same raw pointer DO NOT share a control block — each makes its own, leading to double-delete. Always copy an existing `shared_ptr` to share.)
6. **B** — Ownership moves from `p` to `q`; `p` becomes null; `*q == 42`; `p == nullptr` is `true` → printed as `1`.
7. **A, B, C, D** — Core `unique_ptr` facts. (E: only one owner, by definition.)
8. **A** — The five canonical traits.
9. **E** — `std::sort` needs random-access iterators (so it can binary-jump). For `std::list`, use `list::sort()` member.
10. **A, D** — Forward iterators support read+write, `++`, and multi-pass. (B, C are higher categories. E: `list` is *bidirectional*, not forward.)
11. **B** — Compile-time concept check. If the iterator is missing required operations, the build fails right at this line.
12. **B, C** — Pointer arithmetic and distance are random-access-only.
13. **B** — `istream_view` yields each word; print each word's size: `5 6 6`.
14. **A, B, C** — Correct trait, ownership, end-sentinel. (D / E: forward iterators don't support `--` or `+n`.)
15. **B, D** — Iterators need a non-owning *but stable* handle to the current node. `shared_ptr` copies are cheap and don't transfer ownership away. (A / E are false. C is unrelated — `shared_ptr` doesn't help with stack-overflow on long lists either.)
