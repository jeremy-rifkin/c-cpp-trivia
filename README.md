This repository is a collection of neat C & C++ trivia and oddities.

# Both languages

- `0` is technically tokenized as an octal literal.
- `0XE+2` should evaluate to `16`, however, both gcc and clang give an error: `invalid suffix "+2"
  on integer constant`. Both bugs are known:
  [gcc](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=63337),
  [clang](https://bugs.llvm.org/show_bug.cgi?id=26910). MSVC handles it correctly. This may be due
  to the definition of `pp-number`s and is mentioned in the
  [standard](https://eel.is/c++draft/lex.pptoken#example-2).
- `sizeof(0)["abcd"]` is `1`.
- `https://www.google.com` is a valid line of C/C++ code, but you're limited to one occurrence of
  each protocol per function.
- Unknown attributes are ignored without causing an error (since C++17 and C23). This allows all
  sorts of attribute nonsense:
```cpp
[[std::vector]] void foo() {}
[[code::blocks]] void foo() {}
[[foo...]] void foo() {}
[[]] void foo() {}
[[,]] void foo() {}
[[]][[]][[]][[]][[]] void foo() {}
[[using std:]] void foo() {}
[[typedef ::long]] void foo() {}
[[
#include "/proc/cpuinfo"
]] void foo() {}
```
- The operand of the `sizeof` operator cannot be a C-style cast. `sizeof (int)*p` is parsed as
  `(sizeof(int)) * p` rather than `sizeof((int)*p)`.
- Precedence is ignored in the conditional operator between `?` and `:`:
  `c ? a = 1, y = 2 : foo();` is parsed as `c ? (a = 1, y = 2) : foo();`.
- `llU` is a valid (non-user-defined) integer suffix

# C

- Source code of [the very first C compiler](https://github.com/mortdeus/legacy-cc).
- A significant subset of possible identifiers are reserved in C++. These include identifiers which
  begin with `is` or `to`, `str`, or `mem` followed by a lowercase letter in the global scope. It's
  undefined to declare/define a one of these reserved identifiers in the global scope. So, the
  following program may 1) print 1, 2) wipe your hard drive, 3) summon cthulhu, 4) other. All are
  behaviors are equally correct.
```cpp
#include <stdio.h>
int iseven(int n) {
    return n % 2 == 0;
}
int main() {
    printf("%d", iseven(2));
}
````

# C++

- `decltype(std)` is an `int` in gcc. Bug reports:
  [#1](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=100482),
  [#2](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=101213).
- Prior to gcc 10, `decltype(decltype(decltype))` could be used to generate [exponential error
  messages](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=92105).
- `typedef int i = 0;` segfaults msvc
- The following are valid C++ statements:
```cpp
if(; true) { ... }
if(false; true) { ... }
if(auto main() -> int; true) { ... }
if(class foobar; true) { ... }
```
- We cannot, however, do any of the following:
```cpp
if(static_assert(true); true) { ... }
if(using namespace std; true) { ... }
if(extern "C" int puts(const char*); true) { puts("hello world"); }
if(friend void operator<<(); true) { ... } // syntactically valid, not semantically valid
```
