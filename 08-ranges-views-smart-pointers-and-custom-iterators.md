# ICS 45C — Ranges, Views, Smart Pointers, and Custom Iterators
> Covers the Topic 9 slide deck + HW9 (custom `SetList<T>` with a forward iterator and `MapArray<K,V>` with a random-access iterator, plus a ranges/views reimplementation of mapset). Builds on Guide 07 (the STL).

This is the most "modern C++" of the homeworks. It builds two custom containers from scratch — making them work with `std::ranges` algorithms by exposing the right iterator traits — and uses `shared_ptr` to manage the linked list inside `SetList`.

---

## Table of Contents
1. [Ranges: The Big Idea](#1-ranges-the-big-idea)
2. [Views and the Pipe Operator](#2-views-and-the-pipe-operator)
3. [`istream_view` and Stream Reading](#3-istream_view-and-stream-reading)
4. [Smart Pointers Overview](#4-smart-pointers-overview)
5. [`std::shared_ptr` — Shared Ownership](#5-stdshared_ptr--shared-ownership)
6. [`std::unique_ptr` — Sole Ownership](#6-stdunique_ptr--sole-ownership)
7. [Iterator Categories and Tags](#7-iterator-categories-and-tags)
8. [Iterator Traits (the Five `using`s)](#8-iterator-traits-the-five-usings)
9. [Building a Forward Iterator (HW9 `SetList`)](#9-building-a-forward-iterator-hw9-setlist)
10. [Building a Random-Access Iterator (HW9 `MapArray`)](#10-building-a-random-access-iterator-hw9-maparray)
11. [Concepts and `static_assert`](#11-concepts-and-static_assert)
12. [The HW9 `mapset` Pipeline](#12-the-hw9-mapset-pipeline)
13. [HW9 Pattern Quick Reference](#13-hw9-pattern-quick-reference)
14. [Key Rules to Remember](#14-key-rules-to-remember)

---

## 1. Ranges: The Big Idea

In Guide 02 (and the STL sample guide) you've seen the classic STL: every algorithm takes a `begin()`/`end()` pair.

```cpp
std::sort(v.begin(), v.end());
std::copy(v.begin(), v.end(), std::back_inserter(dest));
```

A **range** is anything with `begin()` and `end()` (a container, a view, a string, a C-array, ...). The C++20 `<ranges>` library provides versions of the algorithms that take the whole range:

```cpp
#include <ranges>
#include <algorithm>

std::ranges::sort(v);                                 // entire container, much shorter
std::ranges::copy(v, std::back_inserter(dest));
```

Same semantics, less typing. `std::ranges::` versions live in `<algorithm>` and `<ranges>`.

---

## 2. Views and the Pipe Operator

A **view** is a lazy, lightweight wrapper over a range. It doesn't own data — it presents an existing range with some transformation or filter applied. Views compose with `|`:

```cpp
#include <ranges>
namespace views = std::views;                          // shorten the namespace

std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};

auto evens_squared = v
    | views::filter([](int x) { return x % 2 == 0; })   // keep evens
    | views::transform([](int x) { return x * x; });    // square them

for (int x : evens_squared)                            // {4, 16, 36, 64}
    std::cout << x << ' ';
```

| View | What it does |
|---|---|
| `views::filter(pred)` | keep elements satisfying `pred` |
| `views::transform(fn)` | apply `fn` to each element |
| `views::take(n)` | first `n` elements |
| `views::drop(n)` | skip first `n` |
| `views::reverse` | iterate in reverse |
| `views::iota(a, b)` | the integer sequence `a, a+1, ..., b-1` |

Views are **lazy**: nothing is computed until you iterate. You can chain as many as you want at zero memory cost; the work happens element by element when consumed.

---

## 3. `istream_view` and Stream Reading

`std::ranges::istream_view<T>(stream)` reads `T`-values from a stream as a range:

```cpp
std::istringstream ss{"10 20 30 40"};
for (int x : std::ranges::istream_view<int>(ss))
    std::cout << x << ' ';                             // 10 20 30 40
```

HW9 chains this with `transform` to lowercase words as they're read:

```cpp
auto words = std::ranges::istream_view<std::string>(stopwords_file)
           | std::views::transform(to_lowercase);
```

This single expression replaces the HW8 idiom:

```cpp
// HW8 — classic STL
std::transform(std::istream_iterator<std::string>(in),
               std::istream_iterator<std::string>(),
               std::inserter(S, S.begin()),
               to_lowercase);
```

Both do the same job; ranges/views read more declaratively.

---

## 4. Smart Pointers Overview

`<memory>` provides two pointer types that **manage** the lifetime of a heap object so you don't need to write `delete`:

| Smart pointer | Ownership model | Cost |
|---|---|---|
| `std::unique_ptr<T>` | exactly one owner; transferable via `std::move` | zero overhead vs raw |
| `std::shared_ptr<T>` | reference-counted shared ownership | atomic ref-count |
| `std::weak_ptr<T>` | non-owning observer of a shared_ptr | breaks cycles |

The pointer object owns the heap object: when the last owning smart pointer dies, the heap object is `delete`d automatically. This is RAII applied to pointers.

---

## 5. `std::shared_ptr` — Shared Ownership

Multiple `shared_ptr`s can point at the same object. A reference count is incremented on copy, decremented on destruction. When it hits zero, the object is freed.

```cpp
#include <memory>

auto sp = std::make_shared<int>(42);                   // preferred over new
{
    auto sp2 = sp;                                     // ref count → 2
    *sp2 = 100;
}                                                       // sp2 dies, count → 1
                                                        // sp still valid, *sp == 100
```                                                      // sp dies, count → 0, int freed

`std::make_shared<T>(args...)` allocates the object and the control block in one allocation — faster and exception-safe.

The HW9 `SetList<T>` builds its linked list with `shared_ptr`:

```cpp
struct ListNode {
    T                          data;
    std::shared_ptr<ListNode>  next;                   // shared, not raw
};
std::shared_ptr<ListNode> head = nullptr;

// Insert at front:
auto node = std::make_shared<ListNode>(ListNode{std::move(value), head});
head = node;
```

When `SetList`'s `head` goes out of scope, the first node's ref-count goes to zero, then the second's, and so on — the entire chain is freed without an explicit `~SetList`. **Rule of Zero achieved on a linked list.**

> **Watch out for cycles**: `shared_ptr` cannot detect cyclic ownership. `A → B → A` will leak unless you use `weak_ptr` for one direction.

---

## 6. `std::unique_ptr` — Sole Ownership

`unique_ptr` allows only **one** owner. Copying is forbidden; you transfer ownership with `std::move`.

```cpp
auto up = std::make_unique<int>(42);
auto up2 = std::move(up);                              // transfer; up is now nullptr
// auto up3 = up2;                                     // ERROR — copy ctor deleted
```

For a `unique_ptr`-based linked list each node owns its `next`:

```cpp
struct Node {
    int                       data;
    std::unique_ptr<Node>     next;
};
std::unique_ptr<Node> head;
```

This makes iterators awkward (an iterator can't observe a node without owning it), which is why HW9's `SetList` uses `shared_ptr` instead — iterators need a stable, non-owning view.

> **Default to `unique_ptr`**. Use `shared_ptr` only when you genuinely need shared ownership.

---

## 7. Iterator Categories and Tags

An iterator is any type that supports `*it` (dereference), `++it` (advance), and `it == other` (compare). The standard library distinguishes five categories by which extra operations they support:

| Category | Operations supported | Containers |
|---|---|---|
| **Input** | read, `++` once | `istream_iterator` |
| **Output** | write, `++` once | `ostream_iterator`, inserters |
| **Forward** | read+write, `++` repeatedly | `forward_list`, `unordered_*`, **`SetList`** |
| **Bidirectional** | forward + `--` | `list`, `set`, `map` |
| **Random Access** | bidirectional + `it + n`, `it - n`, `it[n]`, `<` | `vector`, `deque`, `string`, **`MapArray`** |
| Contiguous (C++17) | random + elements are physically adjacent | `array`, `vector`, `string` |

The standard library uses a **tag type** (an empty struct) to record an iterator's category:

```cpp
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag : public input_iterator_tag {};
struct bidirectional_iterator_tag : public forward_iterator_tag {};
struct random_access_iterator_tag : public bidirectional_iterator_tag {};
struct contiguous_iterator_tag    : public random_access_iterator_tag {};      // C++20
```

Higher categories inherit from lower ones, so a `random_access_iterator_tag` also counts as a `forward_iterator_tag`. The category controls which algorithms can call you efficiently — `std::sort` requires random access, `std::find` is happy with forward, etc.

---

## 8. Iterator Traits (the Five `using`s)

For an iterator to plug into the STL, it must expose five type aliases. The slides call these the *iterator traits*:

```cpp
class MyIterator {
public:
    using iterator_category = std::forward_iterator_tag;       // which category
    using value_type        = T;                                // T (the element)
    using difference_type   = std::ptrdiff_t;                   // signed type for it - it
    using pointer           = T*;                                // *it -> pointer
    using reference         = T&;                                // *it returns this
};
```

These aliases tell the standard library how to use your iterator. Without them, modern algorithms won't compile.

`std::ptrdiff_t` is the signed integer type used for "distance between two pointers." It's the natural type for iterator differences.

---

## 9. Building a Forward Iterator (HW9 `SetList`)

A **forward iterator** supports `*it`, `++it`, `it == other`, and can be passed over multiple times. The HW9 `SetList` (a singly-linked list of unique elements) builds one over `shared_ptr<ListNode>`:

```cpp
template <typename T>
class SetList {
    struct ListNode {
        T                          data;
        std::shared_ptr<ListNode>  next;
    };
public:
    class ListIterator {
    public:
        // The five required type aliases:
        using iterator_category = std::forward_iterator_tag;
        using value_type        = T;
        using difference_type   = std::ptrdiff_t;
        using pointer           = T*;
        using reference         = T&;

        explicit ListIterator(std::shared_ptr<ListNode> ptr = nullptr)
          : ptr{ptr} {}

        ListIterator& operator++()                     // ++it (pre-increment)
        {
            ptr = ptr->next;
            return *this;
        }
        ListIterator operator++(int)                    // it++ (post-increment)
        {
            ListIterator tmp(*this);
            ptr = ptr->next;
            return tmp;
        }

        T& operator*() const  { return ptr->data; }
        T* operator->() const { return &ptr->data; }

        bool operator==(const ListIterator& other) const = default;
        // != is auto-generated since C++20
    private:
        std::shared_ptr<ListNode> ptr;
    };

    using value_type = T;
    using iterator   = ListIterator;

    ListIterator begin() { return ListIterator(head); }
    ListIterator end()   { return ListIterator(nullptr); }   // nullptr = past-the-end

    bool contains(const T& value) {
        for (auto it = begin(); it != end(); ++it)
            if (*it == value) return true;
        return false;
    }

    ListIterator insert(T value) {
        for (auto it = begin(); it != end(); ++it)
            if (*it == value) return it;                       // already present
        auto node = std::make_shared<ListNode>(
            ListNode{std::move(value), head});
        head = node;
        return ListIterator(head);
    }

    // Construct from any input range
    template <std::ranges::input_range Rng>
    explicit SetList(Rng&& rng) {
        std::ranges::for_each(std::forward<Rng>(rng),
                              std::bind_front(&SetList::insert, this));
    }
    SetList() = default;

private:
    std::shared_ptr<ListNode> head = nullptr;
};
```

A few new ingredients deserve names:

- **Post-increment** (`it++`) saves the current state, advances, returns the saved state. That's why it returns *by value* and is slower — prefer `++it` when you can.
- **`= default` on `operator==`** asks the compiler to synthesize member-wise equality (compares `ptr` to `other.ptr`).
- **`template <std::ranges::input_range Rng>`** is a *constrained* template — `Rng` must satisfy the `input_range` concept, otherwise the template won't be selected.
- **`std::bind_front(&SetList::insert, this)`** produces a callable that calls `this->insert(arg)`. Used here to pipe a range through `for_each`.

---

## 10. Building a Random-Access Iterator (HW9 `MapArray`)

The HW9 `MapArray<K,V>` keeps a sorted `std::vector<std::pair<K,V>>` and exposes a **random-access** iterator — needed because the underlying storage is contiguous and we want full pointer-like behavior.

```cpp
template <typename Key, typename Value>
class MapArray {
public:
    class ArrayIterator {
    public:
        using iterator_category = std::random_access_iterator_tag;
        using value_type        = std::pair<Key, Value>;
        using difference_type   = std::ptrdiff_t;
        using pointer           = std::pair<Key, Value>*;
        using reference         = std::pair<Key, Value>&;

        explicit ArrayIterator(std::pair<Key, Value>* p = nullptr) : ptr{p} {}

        // Increment/decrement
        ArrayIterator& operator++()    { ++ptr; return *this; }
        ArrayIterator& operator--()    { --ptr; return *this; }
        ArrayIterator  operator++(int) { auto tmp = *this; ++ptr; return tmp; }
        ArrayIterator  operator--(int) { auto tmp = *this; --ptr; return tmp; }

        // Arithmetic
        ArrayIterator& operator+=(difference_type d) { ptr += d; return *this; }
        ArrayIterator& operator-=(difference_type d) { ptr -= d; return *this; }

        // Symmetric binary +/-
        friend ArrayIterator operator+(ArrayIterator it, difference_type d)
            { return ArrayIterator{it.ptr + d}; }
        friend ArrayIterator operator+(difference_type d, ArrayIterator it)
            { return ArrayIterator{it.ptr + d}; }
        friend ArrayIterator operator-(ArrayIterator it, difference_type d)
            { return ArrayIterator{it.ptr - d}; }
        friend difference_type operator-(ArrayIterator a, ArrayIterator b)
            { return a.ptr - b.ptr; }

        // Comparisons (all six via spaceship)
        auto operator<=>(const ArrayIterator&) const = default;

        // Access
        std::pair<Key, Value>& operator*()  const { return *ptr; }
        std::pair<Key, Value>* operator->() const { return ptr; }
        std::pair<Key, Value>& operator[](difference_type d) const { return ptr[d]; }

    private:
        std::pair<Key, Value>* ptr;
    };

    using value_type = std::pair<Key, Value>;
    using iterator   = ArrayIterator;

    ArrayIterator begin() { return ArrayIterator{data.data()}; }
    ArrayIterator end()   { return ArrayIterator{data.data() + data.size()}; }

    Value& operator[](const Key& key) {                 // map-style access
        auto it = std::lower_bound(
            data.begin(), data.end(), key,
            [](const std::pair<Key, Value>& elem, const Key& k) { return elem.first < k; });
        if (it != data.end() && it->first == key)
            return it->second;                          // already present
        it = data.insert(it, {key, Value{}});           // insert, keep sorted
        return it->second;
    }

private:
    std::vector<std::pair<Key, Value>> data;
};
```

Key operations a random-access iterator must support that a forward iterator doesn't:

| Op | Purpose |
|---|---|
| `it += n`, `it -= n` | jump |
| `it + n`, `n + it`, `it - n` | jump returning new iterator |
| `it1 - it2` | distance |
| `it[n]` | subscript |
| `<`, `<=`, `>`, `>=` (via `<=>`) | ordering |

The `<=>` spaceship from Guide 04 pays off handsomely here — `auto operator<=>(...) const = default;` synthesizes all four ordering operators because the underlying member (`ptr`) already supports them.

### Keeping `data` sorted: `lower_bound` + `vector::insert`

`std::lower_bound(begin, end, value, comp)` runs binary search on a sorted range, returning the first position where `value` could be inserted **without breaking the sort**. If `value` is already there, the iterator points at it; otherwise it points at the slot just after it would go.

`std::vector::insert(pos, elem)` inserts `elem` *before* `pos`, shifting everything after. O(N) due to the shift, but consistent with the map's semantics.

```cpp
auto it = std::lower_bound(data.begin(), data.end(), key, /*comp*/);
if (it != data.end() && it->first == key)
    return it->second;                                  // found
it = data.insert(it, {key, Value{}});                   // not found → insert here
return it->second;
```

The custom comparator deserves a look — `lower_bound`'s `comp` is heterogeneous: it compares an *element* (`pair<K,V>`) against the *key* alone.

---

## 11. Concepts and `static_assert`

C++20 **concepts** are named requirements on types. The standard library defines `std::forward_iterator`, `std::random_access_iterator`, etc. — types that satisfy the iterator traits and operations of those categories.

`static_assert(EXPR)` is a compile-time assertion. Combining the two gives you a one-line iterator audit:

```cpp
static_assert(std::random_access_iterator<MapArray<std::string, int>::iterator>);
static_assert(std::forward_iterator<SetList<int>::iterator>);
```

If your iterator is missing an operation, has a wrong return type, or forgets one of the five traits, **the build fails right there**. HW9 ships these `static_assert`s at the bottom of each header — they're a brutally effective spec.

---

## 12. The HW9 `mapset` Pipeline

HW8's mapset reads stopwords into a `set<string>`, scans the document, and counts words in a `map<string, int>` while skipping stopwords. HW9 does the same job with the custom containers and ranges/views:

```cpp
#include <algorithm>
#include <fstream>
#include <iterator>
#include <ranges>
#include "map_array.hpp"
#include "set_list.hpp"

using namespace std;

string to_lowercase(const string& str) {
    auto lower_view = views::transform(str, ::tolower);
    return {lower_view.begin(), lower_view.end()};       // string from view's iterators
}

SetList<string> load_stopwords(istream& stopwords) {
    return SetList<string>{
        ranges::istream_view<string>(stopwords)
          | views::transform(to_lowercase)
    };
}

MapArray<string, int> count_words(istream& document, SetList<string>& stopwords) {
    auto words_view = ranges::istream_view<string>(document)
                    | views::transform(to_lowercase)
                    | views::filter([&](const string& s) {
                          return !stopwords.contains(s);
                      });
    MapArray<string, int> result;
    for (const string& s : words_view) ++result[s];      // MapArray::operator[]
    return result;
}

void output_word_counts(MapArray<string, int>& counts, ostream& out) {
    for (const auto& [word, count] : counts)
        out << word << ' ' << count << '\n';
}

int main() {
    ifstream stopwords_file{"stopwords.txt"};
    ifstream document       {"sample_doc.txt"};
    ofstream output         {"frequency.txt"};

    auto stopwords    = load_stopwords(stopwords_file);
    auto word_counts  = count_words(document, stopwords);
    output_word_counts(word_counts, output);
}
```

Walk through what the pipeline does for each word in the document:
1. `ranges::istream_view<string>(document)` — read next whitespace-delimited word.
2. `views::transform(to_lowercase)` — lowercase it.
3. `views::filter([&](...){ ... })` — skip if it's a stopword.
4. Body of the for-loop: `++result[s]` — find/insert the key, increment count.

Everything is **lazy** — no intermediate vector or string list is built. The for-loop pulls one word through the pipeline at a time. That's the entire point of views.

---

## 13. HW9 Pattern Quick Reference

### Range-based stream reading + transform + filter

```cpp
auto words = std::ranges::istream_view<std::string>(in)
           | std::views::transform(to_lowercase)
           | std::views::filter(is_not_stopword);

for (const std::string& w : words) ++counts[w];
```

### Build container from any input range (constrained template)

```cpp
template <std::ranges::input_range Rng>
explicit SetList(Rng&& rng) {
    std::ranges::for_each(std::forward<Rng>(rng),
                          std::bind_front(&SetList::insert, this));
}
```

### The five iterator traits, every time

```cpp
class MyIt {
public:
    using iterator_category = std::forward_iterator_tag;   // or random_access_iterator_tag
    using value_type        = T;
    using difference_type   = std::ptrdiff_t;
    using pointer           = T*;
    using reference         = T&;
    /* ... operations ... */
};
```

### A random-access iterator over a raw pointer

```cpp
class It {
    Pair* ptr;
public:
    /* ... 5 traits ... */
    It& operator++()                                    { ++ptr; return *this; }
    It& operator+=(difference_type d)                   { ptr += d; return *this; }
    friend It operator+(It it, difference_type d)       { return It{it.ptr + d}; }
    friend difference_type operator-(It a, It b)        { return a.ptr - b.ptr; }
    auto operator<=>(const It&) const = default;
    Pair& operator*()  const                            { return *ptr; }
    Pair* operator->() const                            { return ptr; }
    Pair& operator[](difference_type d) const           { return ptr[d]; }
};
```

### Compile-time iterator audit

```cpp
static_assert(std::random_access_iterator<MapArray<std::string, int>::iterator>);
static_assert(std::forward_iterator<SetList<int>::iterator>);
```

### Smart-pointer linked list (Rule of Zero)

```cpp
struct Node {
    T                         data;
    std::shared_ptr<Node>     next;
};
std::shared_ptr<Node> head;                            // no destructor needed!
```

---

## 14. Key Rules to Remember

- A **range** is anything with `begin()`/`end()`. `std::ranges::sort(v)` replaces `std::sort(v.begin(), v.end())`.
- A **view** is a lazy adapter that doesn't own data. Compose with `|`: `vec | views::filter(...) | views::transform(...)`.
- `std::ranges::istream_view<T>(stream)` turns a stream into a range of `T`.
- **`unique_ptr`** = one owner, `move`-only. **`shared_ptr`** = reference-counted. Default to `unique_ptr`.
- Build with `std::make_unique<T>(args)` / `std::make_shared<T>(args)` — never `new` in client code.
- The **five iterator categories** are input, output, forward, bidirectional, random-access (+ contiguous). Each is a strict superset of the previous.
- Every iterator must publish **five type aliases**: `iterator_category`, `value_type`, `difference_type`, `pointer`, `reference`.
- A **forward iterator** needs `*it`, `++it`, `it++`, `==`. A **random-access iterator** adds `+`, `-`, `+=`, `-=`, `[]`, and full ordering.
- **Use `auto operator<=>(...) = default;`** on iterators wrapping a single pointer — you get all six comparisons for free.
- **`= default` on `operator==`** synthesizes member-wise equality.
- **`std::ranges::input_range`** etc. are *concepts*. Constrain templates with them: `template <std::ranges::input_range Rng>`.
- **`static_assert(std::forward_iterator<It>);`** is a one-line spec for your iterator. If it fails to compile, your iterator is broken.
- `lower_bound` + `vector::insert` keeps a sorted vector sorted in O(N) (binary search + shift).
- A `shared_ptr`-based linked list achieves the **Rule of Zero** — the chain frees itself.
- Watch for **cycles** with `shared_ptr` (use `weak_ptr` for back-pointers).

This is the last topical guide. Topic 10 (final review) is folded into the master index — see `00-INDEX.md` for the exam-prep summary.
