# Practice Quiz 01 — C++ Foundations

**Covers:** Guide 01 (C++ Tour, Arrays slides — types, I/O, control flow, references, fixed C arrays, char-as-index, file organization). Roughly maps to Reading Quizzes 1 & 2.

**Questions:** 15 (mix of multiple-choice and SELECT ALL)

> Answers and explanations follow the questions. Try the quiz cold, then check.

---

## Questions

### Question 1
What is the value of `result` after this code runs?
```cpp
int result = 0;
for (int i = 1; i <= 4; ++i)
    if (i % 2 == 0)
        result += i;
```
- A. 0
- B. 4
- C. 6
- D. 10
- E. none of the other choices

### Question 2
SELECT ALL statements that are true about the following line:
```cpp
const double& r = x;
```
- A. `r` is a reference
- B. `r` may be reassigned to refer to a different variable later
- C. `r` may NOT be used to modify `x`
- D. `r` must be initialized at the point of declaration
- E. `r` is a pointer

### Question 3
What is the output?
```cpp
#include <iostream>
using namespace std;
int main() {
    char c = 'A';
    cout << c + 1;
}
```
- A. `B`
- B. `66`
- C. `A1`
- D. compile error
- E. none of the other choices

### Question 4
Which array does NOT contain exactly 5 elements over its entire lifetime?
- A. `int a[5];`
- B. `int b[] = {1, 2, 3, 4, 5};`
- C. `int c[5] = {0};`
- D. `int d[3] = {1, 2, 3, 4, 5};`
- E. `int e[5] = {1, 2};`

### Question 5
What is printed?
```cpp
int x = 5;
int& r = x;
int  y = r;
y = 99;
std::cout << x << ' ' << r;
```
- A. `5 5`
- B. `5 99`
- C. `99 99`
- D. `99 5`
- E. none of the other choices

### Question 6
SELECT ALL statements that are true about C-style fixed arrays.
- A. The size must be a compile-time constant.
- B. The array decays to a pointer when passed to a function.
- C. The size of the array is recoverable inside the called function via `sizeof`.
- D. Range-for (`for (auto x : A)`) works on a local stack array.
- E. Out-of-bounds indexing throws `std::out_of_range`.

### Question 7
What is the value of `n` after the loop?
```cpp
int n = 0;
for (int i = 10; i > 0; i /= 2)
    ++n;
```
- A. 3
- B. 4
- C. 5
- D. 10
- E. infinite loop

### Question 8
Which expression converts the lowercase letter `c` to a 0-based alphabet index (a=0, ..., z=25)?
- A. `c - 0`
- B. `c % 26`
- C. `int(c)`
- D. `c - 'a'`
- E. `c & 0x1F`

### Question 9
SELECT ALL statements that are true about `.hpp` and `.cpp` files.
- A. A `.hpp` typically contains declarations; a `.cpp` typically contains definitions.
- B. Including a header twice in the same translation unit is allowed if the header uses an include guard.
- C. Function bodies must always live in `.cpp` files — defining them in `.hpp` causes a linker error.
- D. A `.cpp` file is normally compiled to an object file (`.o` / `.obj`) and later linked.
- E. The compiler implicitly compiles every `.hpp` you `#include`.

### Question 10
What does this code output?
```cpp
#include <iostream>
using namespace std;
int main() {
    int A[5] = {10, 20, 30};
    cout << A[2] << ' ' << A[3];
}
```
- A. `30 0`
- B. `30` followed by garbage
- C. compile error
- D. `30 4`
- E. none of the other choices

### Question 11
A function declared as `void f(const std::string& s)` is being called. SELECT ALL true.
- A. `s` cannot be modified inside `f`.
- B. `s` is a copy of the caller's string.
- C. Passing a string literal like `f("hello")` is allowed.
- D. `f` may pass `s` along to another function that takes `std::string&`.
- E. Pass-by-const-reference avoids the cost of copying.

### Question 12
What does this print?
```cpp
#include <iostream>
using namespace std;
int main() {
    int A[4] = {1, 2, 3, 4};
    int* p = A;
    cout << p[2];
}
```
- A. 3
- B. 2
- C. address of `A[2]`
- D. undefined behavior
- E. none of the other choices

### Question 13
Which keyword or attribute declares a compile-time constant whose value can be used as an array size?
- A. `const`
- B. `constexpr`
- C. `static`
- D. `volatile`
- E. `auto`

### Question 14
SELECT ALL true. The line `std::cin >> name;` where `name` is `std::string`:
- A. reads characters until whitespace.
- B. reads an entire line including spaces.
- C. returns the stream by reference so `cin >> a >> b` chains.
- D. silently truncates input longer than `name`'s current `.size()`.
- E. ignores leading whitespace before reading.

### Question 15
What is the output?
```cpp
#include <iostream>
using namespace std;
int main() {
    int total = 0;
    for (int x : {1, 2, 3, 4, 5}) {
        if (x == 3) continue;
        if (x == 5) break;
        total += x;
    }
    cout << total;
}
```
- A. 7
- B. 15
- C. 10
- D. 6
- E. none of the other choices

---

## Answer Key

1. **C** — `i=1` (odd, skip), `i=2` (+2 → 2), `i=3` (skip), `i=4` (+4 → 6).
2. **A, C, D** — A reference is bound at initialization and cannot be reseated. `const&` forbids modifying the referent. (B is false: references can't be rebound. E is false: a reference is not a pointer.)
3. **B** — `c + 1` promotes `c` to `int` (65), so it prints `66`.
4. **D** — `int d[3] = {1, 2, 3, 4, 5};` is a compile error — too many initializers for a size-3 array. All others have exactly 5 elements.
5. **A** — `r` aliases `x`; `y = r` copies the *value*, so modifying `y` doesn't touch `x`. Both `x` and `r` are still 5.
6. **A, B, D** — Fixed array size is `constexpr`-required; arrays decay to pointers on function call; range-for works locally. (C: once the array decays, the called function loses size; E: no bounds checking on raw arrays.)
7. **B** — i: 10, 5, 2, 1, then `1/2 = 0` exits → loop body ran 4 times.
8. **D** — `c - 'a'` gives 0 for `'a'`, 25 for `'z'`.
9. **A, B, D** — Standard split; include guards prevent re-inclusion; compilation produces object files for linking. (C: bodies *can* live in headers — `inline`, templates, or in-class definitions. E: only what's included via the preprocessor is compiled.)
10. **A** — Partial initializer fills the remaining slots with value-initialized zeros.
11. **A, C, E** — `const&` prevents modification, binds to temporaries (like literals), and avoids copying. (B: `&` means alias, not copy. D: dropping `const` is forbidden — can't pass a const ref to a non-const ref parameter.)
12. **A** — `p` points at `A`; `p[2]` means `*(p+2)` → `A[2]` → 3.
13. **B** — `constexpr` makes the value compile-time. `const` works in many cases but technically `constexpr` is the modern guarantee. The exam answer here is `constexpr`.
14. **A, C, E** — `>>` reads whitespace-delimited tokens, skips leading whitespace, returns the stream by reference. (B: `getline` reads the whole line. D: `string`s grow automatically.)
15. **A** — Walk-through: `x=1` → total=1; `x=2` → total=3; `x=3` → `continue` (skip add); `x=4` → total=7; `x=5` → `break`. Final `total = 7`.
