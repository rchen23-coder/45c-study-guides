# ICS 45C — C++ STL Study Guide
> Covers Topic 8 slides + all STL used in HW8 (process_numbers, mapset, compute_grades)

---

## Table of Contents
1. [STL Overview](#1-stl-overview)
2. [Headers Cheatsheet](#2-headers-cheatsheet)
3. [Containers](#3-containers)
4. [Iterators](#4-iterators)
5. [Iterator Adapters](#5-iterator-adapters)
6. [Algorithms](#6-algorithms)
7. [Ranges Library](#7-ranges-library)
8. [String Streams](#8-string-streams)
9. [File Streams](#9-file-streams)
10. [Formatting (iomanip)](#10-formatting-iomanip)
11. [Numeric Utilities](#11-numeric-utilities)
12. [STL Exceptions](#12-stl-exceptions)
13. [HW8 Patterns Quick Reference](#13-hw8-patterns-quick-reference)

---

## 1. STL Overview

The STL has three main composable pieces:

| Piece | Purpose | Examples |
|---|---|---|
| **Containers** | Hold collections of objects | `vector`, `set`, `map` |
| **Iterators** | Connect algorithms to containers | `begin()`, `end()`, `istream_iterator` |
| **Algorithms** | Process elements | `sort`, `copy`, `transform`, `accumulate` |

The power of the STL is **composability**: any algorithm works with any container via iterators.

---

## 2. Headers Cheatsheet

```cpp
#include <vector>      // std::vector
#include <set>         // std::set, std::multiset
#include <map>         // std::map, std::multimap
#include <algorithm>   // sort, copy, copy_if, transform, for_each, find, accumulate, etc.
#include <iterator>    // istream_iterator, ostream_iterator, back_inserter, inserter
#include <numeric>     // std::accumulate
#include <sstream>     // std::istringstream, std::ostringstream
#include <fstream>     // std::ifstream, std::ofstream
#include <iostream>    // std::cin, std::cout, std::istream, std::ostream
#include <string>      // std::string, std::to_string, std::getline
#include <iomanip>     // std::setw, std::left, std::right
#include <cmath>       // std::round
#include <stdexcept>   // std::domain_error, std::out_of_range, etc.
#include <compare>     // std::strong_ordering (for operator<=>)
```

---

## 3. Containers

### 3.1 `std::vector<T>` — Dynamic Array
```cpp
#include <vector>

std::vector<int> v;            // empty
std::vector<int> v2(100, 0);   // 100 elements, all zero
std::vector<int> v3 = {1,2,3}; // initializer list

v.push_back(42);   // append to end — O(1)
v.pop_back();      // remove from end — O(1)
v[i];              // random access — O(1), no bounds check
v.at(i);           // random access — O(1), throws out_of_range
v.size();          // number of elements
v.empty();         // true if size() == 0
v.begin();         // iterator to first element
v.end();           // iterator one past last element
```
**Use when:** you need a resizable ordered list with fast random access.

---

### 3.2 `std::set<T>` — Sorted Unique Elements
```cpp
#include <set>

std::set<std::string> s;
s.insert("hello");          // O(log N)
s.contains("hello");        // C++20 — true/false
s.count("hello");           // 0 or 1 (use for pre-C++20 membership check)
s.find("hello");            // returns iterator; == s.end() if not found
s.erase("hello");           // O(log N)
s.size();
```
- Always sorted in ascending order
- No duplicates — inserting a duplicate is silently ignored
- `multiset` allows duplicates

**Used in HW8 (mapset):** storing stopwords as a sorted, unique set.

---

### 3.3 `std::map<K, V>` — Sorted Key-Value Store
```cpp
#include <map>

std::map<std::string, int> m;
m["key"] = 42;           // inserts default if key doesn't exist!
m.at("key");             // throws out_of_range if key not found
m.insert({"key", 42});   // no overwrite if key exists
m.contains("key");       // C++20
m.find("key");           // returns iterator to pair, or m.end()
m.count("key");          // 0 or 1

// Range-for — each element is a pair
for (const auto& [key, val] : m) {
    // key = first, val = second
}
for (const auto& p : m) {
    p.first;   // key
    p.second;  // value
}
```
- Keys always sorted ascending
- No duplicate keys (`multimap` allows them)
- `m[key]++` is the idiom for counting word frequency

**Used in HW8 (mapset):** counting word frequencies.

---

### 3.4 Other Containers (from slides)

| Container | Description | Key operations |
|---|---|---|
| `list<T>` | Doubly linked list | `push_front`, `push_back`, bidirectional iter |
| `forward_list<T>` | Singly linked list | `push_front`, forward iter only |
| `deque<T>` | Double-ended queue | `push_front`, `push_back`, random access |
| `array<T,N>` | Fixed-size wrapper around C-array | `[]`, contiguous memory |
| `stack<T>` | Adapts container, LIFO | `push`, `pop`, `top` |
| `queue<T>` | Adapts container, FIFO | `push`, `pop`, `front` |
| `priority_queue<T>` | Max-heap by default | `push`, `pop`, `top` |
| `multiset<T>` | Like set, allows duplicates | same as set |
| `multimap<K,V>` | Like map, allows duplicate keys | same as map |

---

## 4. Iterators

### 4.1 Basic Usage
Every container provides `begin()` and `end()`:
```cpp
std::vector<int> v = {1, 2, 3};
for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it)
    std::cout << *it;    // dereference with *

// Cleaner with auto
for (auto it = v.begin(); it != v.end(); ++it)
    std::cout << *it;

// Even cleaner: range-for (preferred)
for (const auto& e : v)
    std::cout << e;
```

### 4.2 Iterator Operations

| Operation | Meaning |
|---|---|
| `*it` | Dereference — get the element |
| `it->member` | Member access from iterator |
| `++it` | Advance to next element |
| `--it` | Move to previous (bidirectional only) |
| `it == other` / `it != other` | Compare iterators |
| `it = other` | Assign iterator |
| `it[n]` | Random access (random-access iterators only) |

### 4.3 Iterator Categories

| Category | Operations | Containers |
|---|---|---|
| Input | read, `++` | `istream_iterator` |
| Output | write, `++` | `ostream_iterator` |
| Forward | read+write, `++` | `forward_list` |
| Bidirectional | forward + `--` | `list`, `set`, `map` |
| Random Access | bidirectional + `[]` | `vector`, `deque`, `string` |
| Contiguous | random access, physically adjacent | `array`, `vector`, `string` |

### 4.4 Reverse Iterators
```cpp
std::vector<int> v = {1, 2, 3};
// Iterate in reverse
for (auto it = v.rbegin(); it != v.rend(); ++it)
    std::cout << *it;   // prints 3 2 1

// With copy
std::copy(v.rbegin(), v.rend(), std::ostream_iterator<int>(std::cout, " "));
```

---

## 5. Iterator Adapters

### 5.1 `std::back_inserter` — Append to Container
Calls `push_back()` on the destination. Use with `vector`, `list`, `deque`.
```cpp
std::vector<int> dest;
std::copy(src.begin(), src.end(), std::back_inserter(dest));
// each element is push_back'd into dest
```

### 5.2 `std::inserter` — Insert into Associative Container
Calls `insert()` on the destination. Use with `set`, `map`.
```cpp
std::set<int> dest;
std::copy(src.begin(), src.end(), std::inserter(dest, dest.begin()));
```

### 5.3 `std::front_inserter` — Prepend to Container
Calls `push_front()`. Use with `list`, `deque`.
```cpp
std::deque<int> dest;
std::copy(src.begin(), src.end(), std::front_inserter(dest));
// elements arrive in reversed order
```

### 5.4 `std::istream_iterator<T>` — Read from Stream
Reads whitespace-delimited values of type T from any istream.
```cpp
// Read all ints from cin into a vector
std::vector<int> v;
std::copy(std::istream_iterator<int>(std::cin),
          std::istream_iterator<int>(),   // default = EOF sentinel
          std::back_inserter(v));

// Read from file
std::ifstream f("input.txt");
std::copy(std::istream_iterator<int>(f),
          std::istream_iterator<int>(),
          std::back_inserter(v));

// Read from istringstream (used in HW8)
std::istringstream ss("10 20 30");
std::copy(std::istream_iterator<int>(ss),
          std::istream_iterator<int>(),
          std::back_inserter(v));
```

### 5.5 `std::ostream_iterator<T>` — Write to Stream
Writes values to any ostream, with an optional delimiter.
```cpp
std::vector<int> v = {1, 2, 3};

// Print space-separated to cout
std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " "));
// output: 1 2 3

// Write one per line to a file
std::ofstream f("output.txt");
std::copy(v.begin(), v.end(), std::ostream_iterator<int>(f, "\n"));

// With ranges::
std::ranges::copy(v, std::ostream_iterator<int>(std::cout, " "));
```

---

## 6. Algorithms

All from `#include <algorithm>` unless noted. Classic versions take `begin()`/`end()`; `ranges::` versions take the whole container.

### 6.1 `std::copy` / `std::ranges::copy`
Copies all elements from source to destination.
```cpp
std::copy(src.begin(), src.end(), std::back_inserter(dest));
std::ranges::copy(src, std::back_inserter(dest));
std::ranges::copy(src, std::ostream_iterator<int>(std::cout, "\n"));
```

### 6.2 `std::copy_if` / `std::ranges::copy_if`
Copies elements that satisfy a condition.
```cpp
std::vector<int> evens;
std::ranges::copy_if(v, std::back_inserter(evens),
    [](int x){ return x % 2 == 0; });

// To a file
std::ranges::copy_if(v, std::ostream_iterator<int>(file, "\n"),
    [](int x){ return x % 2 == 0; });
```

### 6.3 `std::sort` / `std::ranges::sort`
Sorts in ascending order by default. O(N log N).
```cpp
std::sort(v.begin(), v.end());              // ascending
std::ranges::sort(v);                       // ascending (cleaner)
std::ranges::sort(v, std::greater<int>());  // descending

// Custom comparator (used in HW8 for sorting students)
std::ranges::sort(students, [](const Student& a, const Student& b){
    if (a.last_name != b.last_name) return a.last_name < b.last_name;
    return a.first_name < b.first_name;
});
```

### 6.4 `std::transform`
Applies a function to each element, writing the result to a destination.
```cpp
// In-place: modify each element
std::transform(s.begin(), s.end(), s.begin(), ::tolower);

// To a new container
std::transform(src.begin(), src.end(), std::back_inserter(dest), my_func);

// From istream to set (used in HW8 mapset)
std::transform(std::istream_iterator<std::string>(in),
               std::istream_iterator<std::string>(),
               std::inserter(S, S.begin()),
               to_lowercase);
```

### 6.5 `std::for_each` / `std::ranges::for_each`
Applies a function to every element (no result returned).
```cpp
std::ranges::for_each(v, [](int x){ std::cout << x << "\n"; });

// With capture (used in HW8)
std::ranges::for_each(students, [](Student& s){ s.compute_grade(); });
std::ranges::for_each(quiz, [](const int& score){
    if (score < 0 || score > 100)
        throw std::domain_error("Error: invalid percentage " + std::to_string(score));
});
```

### 6.6 `std::accumulate` — Sum / Fold
From `#include <numeric>`.
```cpp
int sum = std::accumulate(v.begin(), v.end(), 0);     // sum of ints
double sum = std::accumulate(v.begin(), v.end(), 0.0); // sum as double

// Custom operation
int product = std::accumulate(v.begin(), v.end(), 1,
    [](int acc, int x){ return acc * x; });
```

### 6.7 `std::ranges::min` / `std::ranges::max`
```cpp
int lowest = std::ranges::min(quiz);   // min element in container
int highest = std::ranges::max(quiz);  // max element in container

// Classic:
auto it = std::min_element(v.begin(), v.end());
int val = *it;
```

### 6.8 `std::ranges::partition_point`
Returns iterator to first element where condition becomes false. Requires sorted input.
```cpp
// Used in HW8 compute_grades to find letter grade
auto it = std::ranges::partition_point(scale,
    [&](const auto& p){ return p.first > course_score; });
// it->second is the letter grade
```

### 6.9 Other Algorithms (from slides)

| Algorithm | What it does |
|---|---|
| `std::find(b, e, val)` | Returns iterator to first match, or `end()` |
| `std::reverse(b, e)` | Reverses elements in range |
| `std::unique_copy(b, e, out)` | Copies without consecutive duplicates |
| `std::min_element(b, e)` | Returns iterator to minimum element |
| `std::max_element(b, e)` | Returns iterator to maximum element |

---

## 7. Ranges Library
`#include <algorithm>` (ranges are part of C++20)

The `std::ranges::` versions of algorithms accept a whole container instead of `begin()`/`end()` pairs:

```cpp
// Classic STL
std::sort(v.begin(), v.end());
std::copy_if(v.begin(), v.end(), std::back_inserter(dest), pred);

// Ranges — pass container directly
std::ranges::sort(v);
std::ranges::copy_if(v, std::back_inserter(dest), pred);
```

All `ranges::` algorithms: `sort`, `copy`, `copy_if`, `for_each`, `find`,
`min`, `max`, `partition_point`, `transform`, `fill`, `count_if`, and more.

Reference: https://en.cppreference.com/w/cpp/ranges

---

## 8. String Streams
`#include <sstream>`

### 8.1 `std::istringstream` — Parse a String Like a Stream
```cpp
std::string line = "Quiz 80 90 70";
std::istringstream ss(line);

std::string keyword;
ss >> keyword;   // reads "Quiz"

// Read remaining ints into vector
std::vector<int> scores;
std::copy(std::istream_iterator<int>(ss),
          std::istream_iterator<int>(),
          std::back_inserter(scores));
// scores = {80, 90, 70}
```
**Used in HW8:** parsing each line of the gradebook by keyword.

### 8.2 `std::ostringstream` — Build a String via Stream
```cpp
std::ostringstream oss;
oss << "Name: " << first << " " << last << "\n";
std::string result = oss.str();
```

### 8.3 `std::getline`
Reads an entire line from an istream into a string.
```cpp
std::string line;
std::getline(in, line);          // reads until '\n'
std::getline(in, line, ',');     // reads until ',' (custom delimiter)

// Used in HW8 operator>> to read student blocks
while (std::getline(in, line) && !line.empty()) {
    // process each non-blank line
}
```

---

## 9. File Streams
`#include <fstream>`

```cpp
// Writing
std::ofstream out("output.txt");         // creates/overwrites
std::ofstream out("output.txt", std::ios::app);  // append mode
out << "hello " << 42 << "\n";
out.close();   // optional — closes on scope exit

// Reading
std::ifstream in("input.txt");
if (!in) { /* failed to open */ }
int x;
in >> x;       // read one value
std::string line;
std::getline(in, line);   // read one line

// Used in HW8 main.cpp:
std::ifstream numbers("rand_numbers.txt");
std::ofstream odds("odds.txt");
split_odd_even(numbers, odds, evens);   // pass as istream/ostream
```

**Key rule:** In functions, use `std::istream&` / `std::ostream&` parameters — they accept any stream (file, console, string stream).

---

## 10. Formatting (iomanip)
`#include <iomanip>`

```cpp
// Used in HW8 compute_grades operator<<
out << std::left << std::setw(8) << "Name:" << name << "\n";
//                  ^^^^^^^^^^^    ^^^^^^
//                  field width 8  left-aligned

out << std::right << std::setw(5) << 42;  // right-aligned in width 5
```

| Manipulator | Effect |
|---|---|
| `std::setw(n)` | Set field width to n (applies to next output only) |
| `std::left` | Left-align output |
| `std::right` | Right-align output (default) |
| `std::setprecision(n)` | Set decimal precision |
| `std::fixed` | Fixed-point notation |

---

## 11. Numeric Utilities

### `std::to_string`
```cpp
#include <string>
std::string s = std::to_string(42);     // "42"
std::string s = std::to_string(3.14);   // "3.140000"
```

### `std::round`
```cpp
#include <cmath>
int score = std::round(0.4 * quiz_avg + 0.3 * hw_avg + 0.3 * final_score);
```

### `std::accumulate`
```cpp
#include <numeric>
int total = std::accumulate(v.begin(), v.end(), 0);  // 0 is the start value
```

---

## 12. STL Exceptions

### Standard Exception Hierarchy
```
exception
├── logic_error
│   ├── domain_error       ← used in HW8 validate()
│   ├── invalid_argument
│   ├── length_error
│   └── out_of_range       ← thrown by .at() on out-of-bounds access
└── runtime_error
    ├── range_error
    ├── overflow_error
    └── underflow_error
```

### Usage
```cpp
#include <stdexcept>

// Throw
throw std::domain_error("Error: invalid percentage " + std::to_string(score));

// Catch
try {
    s.validate();
} catch (const std::domain_error& e) {
    std::cerr << e.what() << "\n";
}
```

### `std::bad_alloc`
Thrown when `new` fails (out of memory). No need to include `<stdexcept>`.

---

## 13. HW8 Patterns Quick Reference

### process_numbers — Read → Sort → Filter → Write
```cpp
// Read ints from istream into vector
std::vector<int> V;
std::copy(std::istream_iterator<int>(numbers),
          std::istream_iterator<int>(),
          std::back_inserter(V));

// Sort
std::ranges::sort(V);

// Write odds (space-separated) to ostream
std::ranges::copy_if(V, std::ostream_iterator<int>(odds, " "),
    [](int x){ return x % 2 != 0; });
odds << "\n";

// Write evens (one per line)
std::ranges::copy_if(V, std::ostream_iterator<int>(evens, "\n"),
    [](int x){ return x % 2 == 0; });
```

---

### mapset — Read → Transform → Filter → Count
```cpp
// Load stopwords from istream into set, lowercased
std::set<std::string> S;
std::transform(std::istream_iterator<std::string>(stopwords),
               std::istream_iterator<std::string>(),
               std::inserter(S, S.begin()),
               to_lowercase);

// Count words from document, skipping stopwords
std::map<std::string, int> M;
std::for_each(std::istream_iterator<std::string>(document),
              std::istream_iterator<std::string>(),
              [&](const std::string& word){
                  if (!stopwords.contains(to_lowercase(word)))
                      M[to_lowercase(word)]++;
              });

// Output map entries
std::ranges::for_each(M, [&output](const auto& p){
    output << p.first << " " << p.second << "\n";
});
```

---

### compute_grades — Parse → Compute → Sort → Output
```cpp
// Parse istream line by line
std::string line;
while (std::getline(in, line) && !line.empty()) {
    std::istringstream ss(line);
    std::string keyword;
    ss >> keyword;
    if (keyword == "Quiz")
        std::copy(std::istream_iterator<int>(ss),
                  std::istream_iterator<int>(),
                  std::back_inserter(s.quiz));
}

// Compute quiz average (drop lowest of 2+)
int total = std::accumulate(quiz.begin(), quiz.end(), 0);
int lowest = std::ranges::min(quiz);
quiz_avg = double(total - lowest) / (quiz.size() - 1);

// Validate
std::ranges::for_each(quiz, [](const int& score){
    if (score < 0 || score > 100)
        throw std::domain_error("Error: invalid percentage " + std::to_string(score));
});

// Find letter grade
auto it = std::ranges::partition_point(scale,
    [&](const auto& p){ return p.first > course_score; });
course_grade = it->second;

// Sort students by last name, then first name
std::ranges::sort(students, [](const Student& a, const Student& b){
    if (a.last_name != b.last_name) return a.last_name < b.last_name;
    return a.first_name < b.first_name;
});

// Output with formatting
out << std::left << std::setw(8) << "Name:" << first << " " << last << "\n";

// Copy all students to output stream
std::ranges::copy(students, std::ostream_iterator<Student>(out));
```

---

## Key Rules to Remember

**No raw loops** — use STL algorithms instead:
- Iterating + doing something → `for_each`
- Filtering into container → `copy_if`
- Transforming elements → `transform`
- Summing → `accumulate`
- Sorting → `ranges::sort`

**Destination iterator by container type:**
- `vector`, `list` → `back_inserter`
- `set`, `map` → `inserter(s, s.begin())`
- `deque` → `front_inserter` or `back_inserter`
- file/console → `ostream_iterator<T>(stream, delimiter)`

**`ranges::` vs classic:**
- `ranges::` takes the whole container, classic takes `begin()`/`end()`
- Both work; `ranges::` is cleaner for C++20

**`istream_iterator<T>` sentinel:**
- `istream_iterator<T>(stream)` — start (reads first value immediately)
- `istream_iterator<T>()` — end (EOF sentinel, default constructor)