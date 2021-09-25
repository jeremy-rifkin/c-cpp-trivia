This repository is a collection of neat C & C++ trivia and oddities.

### Table of contents: <!-- omit in toc -->
- [Both languages](#both-languages)
	- ["Special operators"](#special-operators)
	- [Bugs and Implementation Quirks](#bugs-and-implementation-quirks)
- [C](#c)
	- [Bugs and Implementation Quirks](#bugs-and-implementation-quirks-1)
- [C++](#c-1)
	- [Bugs and Implementation Quirks](#bugs-and-implementation-quirks-2)

## Both languages

- `0` is technically tokenized as an octal literal.
- Array access is commutative: `arr[i]` and `i[arr]` are equivalent. This is because array access is
  defined as a direct translation to `*(arr + i)`.
- `sizeof(0)["abcd"]` is `1`.
- `https://www.google.com` is a valid line of C/C++ code, but you're limited to one occurrence of
  each protocol per function.
- Unknown attributes are ignored without causing an error (since C++17 and C23). This allows all
  sorts of attribute nonsense (And all of these can of course be applied to variables too):
```cpp
[[std::vector]] void foo() {} // Yes, even in C
[[code::blocks]] void foo() {}
[[]] void foo() {}
[[,]] void foo() {}
[[]][[]][[]][[]][[]] void foo() {}
[[typedef ::long]] void foo() {}
[[
#include "/proc/cpuinfo"
]] void foo() {}
// C++ only:
[[foo...]] void foo() {}
[[using std:]] void foo() {}
```
- The operand of the `sizeof` operator cannot be a C-style cast. `sizeof (int)*p` is parsed as
  `(sizeof(int)) * p` rather than `sizeof((int)*p)`.
- Precedence is ignored in the conditional operator between `?` and `:`:
  `c ? a = 1, y = 2 : foo();` is parsed as `c ? (a = 1, y = 2) : foo();`.
- `llU` is a valid (non-user-defined) integer suffix
- Preprocessor directives can be empty:
```c
#include <stdio.h>
#
#
int main() {
	#
	// ...
}
```
- Switch statement bodies are allowed to be single statements as opposed to statement sequences (or
  compound statements), like other control flow structures:
```cpp
switch(x) case 1: case 2: puts("foo");
```
- Case labels do not need to be in the top-level statement sequence
```cpp
int x = 2;
int i = 0;
switch(x) {
	default:
	if(foo()) {
		while(i++ < 5) {
			case 2:
				puts("lol");
		}
	}
}
```
- `"a" + 1 == ""` can technically evaluate to `true`. As can `"a" == "a\0\0"`.
### "Special operators"
- ["`-->` operator"](https://stackoverflow.com/q/1642028/15675011), really just a
  combination of two operators
```cpp
int x = 10;
while (x --> 0) { // x goes to 0
	printf("%d ", x);
}
```
- ["Tadpole operator"](https://devblogs.microsoft.com/oldnewthing/20150525-00/?p=45044):

Syntax | Meaning | Mnemonic
-------|---------|---------
-~y    | y + 1   | Tadpole swimming toward a value makes it bigger
~-y    | y - 1   | Tadpole swimming away from a value makes it smaller
- "Unset operator": `x &~ mask` unsets `mask` bits in `x`
### Bugs and Implementation Quirks
- `0XE+2` should evaluate to `16`, however, both gcc and clang give an error: `invalid suffix "+2"
  on integer constant`. Both bugs are known:
  [gcc](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=63337),
  [clang](https://bugs.llvm.org/show_bug.cgi?id=26910). MSVC handles it correctly. This may be due
  to the definition of `pp-number`s and is mentioned in the
  [standard](https://eel.is/c++draft/lex.pptoken#example-2).

## C

- Source code of [the very first C compiler](https://github.com/mortdeus/legacy-cc).
- An empty struct is UB in C. Standard quote: 6.7.2.1.8 (C11-C23).
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
```
### Bugs and Implementation Quirks
- gcc allows completely empty case labels (in C mode only):
```c
switch(x) { case 1: }
```
- gcc allows labels to be applied to declarations
```c
switch(x) { default: int y; }
switch(x) { default:; int y; } // must be this in clang
```

## C++

- The size of an empty struct is `1`. Standard
  [link](https://eel.is/c++draft/basic.memobj#intro.object-9.sentence-2).
- All types must be deduced the same in an `auto` declarator list. I.e. `auto x = 1, y = 1.5;` is
  not allowed.
- C++ structs can have stray semicolons:
```cpp
struct S { ;;;;; };
```
- The language grammar allows `for`-style `init-statement`s in `switch` and `if` statements, Since
  C++17:
```cpp
switch(int x = foo(); t[x]) { ... }
if(auto [a, b, c] = foo(); c) { ... }
```
- `while` loops do not support `init-statement`s because that would make them
  [just another for loop](https://stackoverflow.com/a/59986173/15675011).
- A `condition` may be a declaration. This allows up to two declarations per `switch`, `if`, or
  `for` statement:
```cpp
if(int x = foo()) { ... } // intended use
if(auto [a, b, c] = foo(); auto x = bar(a, b, c)) { ... }
for(auto [a, b, c] = foo(); int x = baz(); c++) { ... }
```
- While an `init-statement` may make array or structured binding declarations, `condition`
  declarations may not. I.e. these are not valid:
```cpp
if(auto [a, b, c] = foo()) {  }
if(int arr[] = {1, 2, 3, 4}) {  }
```
- The following are valid C++ statements:
```cpp
if(; true) { ... } // empty init-statement
if(false; true) { ... }
if(auto main() -> int; true) { ... }
if(class foobar; true) { ... }
if(typedef int i32; true) { ... }
if(using A = B; true) { ... } // Since C++23
for(struct { int a = 0, b = 100; } s; s.a < s.b; s.a++, s.b--) { ... }
```
- We cannot, however, do any of the following:
```cpp
if(static_assert(true); true) { ... }
if(using namespace std; true) { ... }
if(extern "C" int puts(const char*); true) { puts("hello world"); }
if(friend void operator<<(); true) { ... } // syntactically valid, not semantically valid
```
- A lambda's parameter list can be omitted: `[]{ return 42; }`.
- `(*****+***+**+*+[]{})();` is valid C++. Global operators `T& operator*(T*)` and
  `T* operator+(T*)` can be used on lambdas with no captures (which decay to function pointers).
- The following is valid (since C++23)
```cpp
[] [[deprecated]] [[deprecated]] {}; // self-deprecating lambda
```
- `[[likely]]` can be applied outside of control flow structures:
```cpp
[[likely]] ;
[[likely]] {};
[[likely]] 1 + 2;
```
- There are special rules for lexing `<:` digraphs so that `std::vector<::std::string>` is lexed
  correctly and not as `std::vector[:std::string>`:
> Otherwise, if the next three characters are <​::​ and the subsequent character is neither : nor >, the < is treated as a preprocessing token by itself and not as the first character of the alternative token <:

[https://eel.is/c++draft/lex.pptoken#3.2](https://eel.is/c++draft/lex.pptoken#3.2)
### Bugs and Implementation Quirks
- `decltype(std)` is an `int` in gcc. Bug reports:
  [#1](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=100482),
  [#2](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=101213).
- Prior to gcc 10, `decltype(decltype(decltype))` could be used to generate [exponential error
  messages](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=92105).
- `typedef int i = 0;` segfaults msvc
- This compiles and links in gcc
```cpp
namespace foobar {
    extern "C" int main() {
        puts("Hello world!");
    }
}
```
