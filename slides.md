---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://images.unsplash.com/photo-1619796753108-cba77bacf03d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1374&q=80
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
---

# Niebloid in C++


<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/neal/2018" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>


---
layout: image-right
image: https://images.unsplash.com/photo-1619796753108-cba77bacf03d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1374&q=80
---

# Background: Function Overloading

```cpp 
int a = 0;
int a = 0; // error here, can not have same name
```

<br>

```cpp 
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}

int add(int a, int b, int c) {
    return a + b + c;
}
```
<br>

```cpp 
int result = add(1, 1);
// call the first function
```

---
layout: image-right
image: https://images.unsplash.com/photo-1619796753108-cba77bacf03d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1374&q=80
---

# Background: Namespace

```cpp 
std::cout << "Hello, world!";
```

<br>

```cpp 
using namespace std;
// do not need `std::`
cout << "Hello, world!"; 
```

<!-- This statement like print in Python;.it is like a folder -->

---
layout: two-cols-header
---

# `std::ranges` Namespace (C++20)

<br>

::left::

```cpp
std::vector a = {1, 3, 2, 4};
```

with previous function...

```cpp
// sort it
std::sort(a.begin(), a.end());
// it is sorted
assert(std::is_sorted(a.begin(), a.end()));
```

with `std::ranges`...

```cpp
// sort it
std::ranges::sort(a);
// it is sorted
assert(std::ranges::is_sorted(a));
```

::right::
<div p-5>

```cpp
// std namespace
void std::sort(T first, T last);

// std::ranges namespace
void std::ranges::sort(T ranges);
void std::ranges::sort(I first, S last);
// we ignore the return value for simplicity
```
</div>

---
layout: two-cols-header
---

# Which Function will It Call?

::left::

```cpp
// using namespace
using namespace std;
using namespace std::ranges;

vector a = {1, 3, 2, 4};
sort(a); // which function will it call?
```

<br>

```cpp
// three candidates...
void std::sort(T first, T last);
void std::ranges::sort(T ranges);
void std::ranges::sort(I first, S last);
```
::right::

<div p-5>
<v-click>

Compile Error!!

https://godbolt.org/z/s7bjT8xh1

</v-click>
</div>

---
layout: image-right
image: https://images.unsplash.com/photo-1619796753108-cba77bacf03d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1374&q=80
---

# Argument-Dependent Lookup

<br>

Recall the statement printing "hello world"...

```cpp
std::cout << "Hello, world!";
```

is actually

```cpp 
operator<<(std::cout, "Hello, world!");
```
How does it find `std::operator<<`?

---
layout: two-cols
---

# Argument-Dependent Lookup

<br>

Recall the statement printing "hello world"...

```cpp
std::cout << "Hello, world!";
```

is actually

```cpp 
operator<<(std::cout, "Hello, world!");
```
How does it find `std::operator<<`?

::right::
<div p-5>

`std::cout` is inside `std`,
the compiler will look for all possible _**functions**_ in `std` as well

This allows us to write

```cpp
std::cout << "Hello, world!"";
```

instead of
```cpp 
std::operator<<(std::cout, "Hello, world!");
```
</div>

---
layout: two-cols
---

# Here Comes the Problem...

<br>


```cpp
using namespace std;

vector<int> a = {1, 3, 2, 4};
// which function will it call?
sort(a.begin(), a.end());
```

<br>

```cpp
using namespace std::ranges;
// not using std

std::vector<int> a = {1, 3, 2, 4};
// which function will it call?
sort(a.begin(), a.end());
```


::right::
<div p-5 py-30>

```cpp
// three candidates...
void std::sort(T first, T last);
void std::ranges::sort(T ranges);
void std::ranges::sort(I first, S last);
```

<v-click>

The second one still calls `std::sort` since we have ADL and `std::sort` is more suitable.

Counter-intuitive!
</v-click>

</div>

---

# Disable ADL: Niebloid!

<div></div>

If `sort` is a object instead of a function, ADL will not be effective.

Because ADL only work for actual functions.

```cpp
namespace std::ranges {
struct __sort_fn {
    void operator()(R ranges) {
        // implementation  
    }
};

// declare the functor object
__sort_fn sort;
} // namespace std::ranges

// call it
std::vector a = {1, 3, 2, 4};
std::ranges::sort(a);
// std::ranges::sort.operator()(a);
```

The functors that disable ADL are called Niebloids.

---
layout: two-cols-header
---

# Consequence...

::left::

<br>

Without ADL, we can call `std::ranges` correctly.

```cpp
using namespace std::ranges;
// not using std

std::vector<int> a = {1, 3, 2, 4};
sort(a.begin(), a.end());
```

::right::

<div p-5>

Function overloading no longer works...

Here we have two identities...

```cpp
// using namespace
using namespace std;
using namespace std::ranges;

vector a = {1, 3, 2, 4};
sort(a);
```

`std::sort` is a function, while `std::ranges::sort` is an object.

</div>

---
layout: cover
image: https://images.unsplash.com/photo-1619796753108-cba77bacf03d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1374&q=80
---

# Thanks

# Q&A?
