# Practice Quiz 07 — The STL

**Covers:** Guide 07 (Topic 8 slides + HW8 — containers, iterators, algorithms, adapters, ranges intro, string/file streams, iomanip, STL exceptions).

**Questions:** 15

---

## Questions

### Question 1
SELECT ALL true about `std::vector<T>`.
- A. Random access via `[]` is O(1).
- B. `push_back` is amortized O(1).
- C. Iterators may be invalidated when the vector grows.
- D. `vector<int>` and `vector<double>` are unrelated types (template instantiations).
- E. `vector::sort()` is a member function.

### Question 2
Which container is best for "frequency count: word → count"?
- A. `std::vector<int>`
- B. `std::set<std::string>`
- C. `std::map<std::string, int>`
- D. `std::list<std::string>`
- E. `std::stack<int>`

### Question 3
What does this print?
```cpp
#include <iostream>
#include <map>
int main() {
    std::map<std::string, int> m;
    ++m["a"]; ++m["b"]; ++m["a"]; ++m["c"]; ++m["a"];
    for (const auto& [k, v] : m)
        std::cout << k << v << ' ';
}
```
- A. `a3 b1 c1`
- B. `a1 a1 a1 b1 c1`
- C. `a1 b1 c1`
- D. `c1 b1 a3`
- E. compile error

### Question 4
SELECT ALL true about iterators.
- A. `begin()` returns an iterator to the first element.
- B. `end()` returns an iterator to the last element.
- C. Algorithms like `std::sort` use iterators and therefore work across multiple container types.
- D. `*it` dereferences the iterator to get the current element.
- E. `cbegin()` returns a read-only iterator.

### Question 5
Which is the correct way to sort a vector ascending?
- A. `v.sort();`
- B. `std::sort(v.cbegin(), v.cend());`
- C. `std::sort(v.begin(), v.end());`
- D. `sort v;`
- E. `std::ranges::sort_asc(v);`

### Question 6
What does `std::set<T>::find(key)` return when `key` is **not** present?
- A. `nullptr`
- B. `false`
- C. `-1`
- D. `s.end()`
- E. throws `std::out_of_range`

### Question 7
What does this print?
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
int main() {
    std::vector<int> v = {5, 1, 4, 2, 3};
    std::sort(v.begin(), v.end());
    auto it = std::find(v.begin(), v.end(), 4);
    if (it != v.end()) std::cout << *it << ' ' << (it - v.begin());
    else                std::cout << "missing";
}
```
- A. `4 3`
- B. `4 4`
- C. `3 4`
- D. `missing`
- E. compile error

### Question 8
SELECT ALL true about stream iterators.
- A. `std::istream_iterator<int>(cin)` lets algorithms read from `cin` as a sequence.
- B. `std::istream_iterator<int>()` (default-constructed) is the EOF sentinel.
- C. `std::ostream_iterator<int>(cout, " ")` writes ints to `cout` with " " as a separator after each.
- D. `back_inserter(v)` is an output iterator that calls `v.push_back(...)`.
- E. `inserter(s, s.begin())` works for `std::set` and `std::map`.

### Question 9
What does this print?
```cpp
#include <iostream>
#include <sstream>
int main() {
    std::istringstream ss("10 20 30");
    int a, b, c;
    ss >> a >> b >> c;
    std::cout << (a + b + c);
}
```
- A. `102030`
- B. `60`
- C. compile error
- D. nothing (extraction fails)
- E. none of the other choices

### Question 10
Which header provides `std::setw`, `std::left`, `std::right`?
- A. `<iostream>`
- B. `<iomanip>`
- C. `<fstream>`
- D. `<format>`
- E. `<sstream>`

### Question 11
SELECT ALL true about `std::accumulate(v.begin(), v.end(), init)`.
- A. It is declared in `<numeric>`.
- B. With `init = 0` it sums the elements.
- C. The result type follows the type of `init` — pass `0.0` for a double sum.
- D. It works on any input-iterator range, not just vectors.
- E. It is a member function of `std::vector`.

### Question 12
What does this print?
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    int n = std::count_if(v.begin(), v.end(),
                          [](int x) { return x % 2 == 0; });
    std::cout << n;
}
```
- A. 0
- B. 2
- C. 3
- D. 5
- E. compile error

### Question 13
What does this print?
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>
int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::vector<int> out;
    std::copy_if(v.begin(), v.end(),
                 std::back_inserter(out),
                 [](int x) { return x > 2; });
    for (int x : out) std::cout << x;
}
```
- A. `12`
- B. `345`
- C. `12345`
- D. nothing
- E. compile error

### Question 14
SELECT ALL true about iterator categories.
- A. `std::vector::iterator` is random-access.
- B. `std::list::iterator` is random-access.
- C. `std::map::iterator` is bidirectional.
- D. `std::forward_list::iterator` is forward.
- E. `std::istream_iterator` is input.

### Question 15
Which catch clause matches an `std::out_of_range` thrown by `vector::at()`?
- A. `catch (std::exception&)`
- B. `catch (std::logic_error&)`
- C. `catch (std::out_of_range&)`
- D. `catch (...)`
- E. All of the above (the first matching clause runs).

---

## Answer Key

1. **A, B, C, D** — Standard vector facts. (E: `vector` has no `sort()` member; use `std::sort`.)
2. **C** — `map<string, int>` is the canonical word-count container; `++m[word]` is the idiom.
3. **A** — Map keys are sorted; counts: a=3, b=1, c=1.
4. **A, C, D, E** — Standard iterator facts. (B: `end()` returns one-PAST-the-last element, not the last.)
5. **C** — Classic STL: pair of mutable iterators to `std::sort`. (B uses `cbegin/cend` which are const — sort needs to modify. A: no member. E doesn't exist.)
6. **D** — `find` returns `end()` when the key is missing.
7. **A** — After sort: `{1,2,3,4,5}`. `find` returns iterator to `4` at index 3. Output: `4 3`.
8. **A, B, C, D, E** — All five are standard stream-iterator facts.
9. **B** — `istringstream` parses each int; sum = 60.
10. **B** — `<iomanip>`.
11. **A, B, C, D** — Standard `accumulate` facts. (E: it is a free function in `<numeric>`, not a member.)
12. **B** — Even numbers in `{1..5}`: 2 and 4 → count = 2.
13. **B** — `copy_if` keeps elements > 2: `345`.
14. **A, C, D, E** — Standard categories. (B: `list` is bidirectional, not random-access.)
15. **E** — `out_of_range` derives from `logic_error` which derives from `exception`. The **first matching** clause runs; later ones are skipped — so in a single `try/catch`, you'd write the most specific FIRST.
