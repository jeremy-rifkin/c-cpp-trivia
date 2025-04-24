This repository is a collection of neat C & C++ trivia and oddities.

### Table of contents: <!-- omit in toc -->
- [Both languages](#both-languages)
  - ["Special operators"](#special-operators)
  - [Bugs and Implementation Quirks](#bugs-and-implementation-quirks)
- [C++](#c)
  - [Bugs and Implementation Quirks](#bugs-and-implementation-quirks-1)
- [C](#c-1)
  - [Bugs and Implementation Quirks](#bugs-and-implementation-quirks-2)
- [Talks](#talks)

## Both languages

- `0` is technically tokenized as an octal literal.
- Array access is commutative: `arr[i]` and `i[arr]` are equivalent. This is because array access is
  defined as a direct translation to `*(arr + i)`.
- `sizeof(0)["abcd"]` is `1`.
- C and C++ grammar allows prototypes in declaration lists: `int a, foo(), * bar(), main();`.
- `https://www.google.com` is a valid line of C/C++ code, but you're limited to one occurrence of
  each protocol per function.
- Operator precedence and associativity is *not* the same as order of evaluation. The following are
  all undefined or unspecified behavior:
```cpp
void foo(int i, int* arr) {
    i = i++; // UB in C or before C++17
    i = i++ + ++i; // UB
    arr[i] = i++; // UB in C or before C++17
    bar(puts("a"), puts("b")); // clang spits out a b, gcc spits out b a
}
```
[https://en.cppreference.com/w/cpp/language/eval_order](https://en.cppreference.com/w/cpp/language/eval_order)
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
- Attributes may appear almost anywhere in a declaration:
```cpp
[[foo]] int [[bar]] baz [[biz]] () [[buz]];
[[foo]] constexpr [[bar]] int [[baz]] biz [[buz]] () [[boz]];
// ^ second one is gcc and msvc only, decl-specifier-spec technically prevents an attribute here
```
- The operand of the `sizeof` operator cannot be a C-style cast. `sizeof (int)*p` is parsed as
  `(sizeof(int)) * p` rather than `sizeof((int)*p)`.
- Precedence is ignored in the conditional operator between `?` and `:`:
  `c ? a = 1, y = 2 : foo();` is parsed as `c ? (a = 1, y = 2) : foo();`.
- `llU` is a valid (non-user-defined) integer suffix
- `(void)` cast
```c
void foo(int x) {
    (void)x; // useful for suppressing unused parameter warnings
	// C++ only: (will be a warning with -Wpedantic)
    return (void)"You can also return anything from a void function";
}
```
- You cannot augment a typedef (or `using` alias) with `unsigned`:
```c
typedef long long ll
void foo(unsigned ll) {} // unsigned implies unsigned int, ll here is a parameter name
```
- `typedef` is a storage class specifier and can appear before, after, or in the middle of a type in a declaration
```c
unsigned typedef int u32;
```
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
- C and C++ support a set of
  [digraph tokens and trigraphs](https://en.wikipedia.org/wiki/Digraphs_and_trigraphs_(programming)#C) and [alternative tokens](https://en.wikipedia.org/wiki/C_alternative_tokens) to
  accommodate certain [archaic character sets](https://en.wikipedia.org/wiki/ISO/IEC_646) which rendered some ASCII characters differently. `not` and `not_eq` also exist because some
  [EBCDIC character sets](https://en.wikipedia.org/wiki/EBCDIC) didn't have a character that rendered as an exclamation mark. Trigraphs were removed from C++ in C++17 and C in C23,
  because they were replaced before tokenization which caused some surprising behavior:
  ```cpp
  puts("??("); // when trigraphs are supported, this outputs [ instead of ??(
  puts("<:"); // outputs <:, there is no way to use digraphs in character constants or string literals
  ```
- ISO C forbids conversion between function and object pointers, and ISO C++ allows implementations to forbid such conversions:
  ```cpp
  void (*func_ptr)() = dlsym(mylib, "func"); // gcc and clang yield a warning in pedantic mode
  ```
  However, if taking the address to the function pointer first, then casting to `void**` and finally dereferencing this pointer again, makes it (usually) work without warnings:
  ```cpp
  void (*func_ptr)();
  *(void**)&func_ptr = dlsym(mylib, "func");
  ```
  Though this trick gets around the warning, the behavior is undefined due to strict aliasing so it may not work.
- It's possible to declare multiple functions at once and use typedefs / using decllarations for signatures:
```cpp
// declares void foo(int); void* baz(float);
void foo(int), * bar(float);
// declares void foo(); void bar();
typedef void fn(); // or using fn = void();
fn foo, bar;
```
### "Special operators"
- ["`-->` operator"](https://stackoverflow.com/q/1642028/15675011), really just a combination of two operators
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
- Boolean identity: `!-!b`

### Bugs and Implementation Quirks
- `0XE+2` should evaluate to `16`, however, both gcc and clang give an error: `invalid suffix "+2"
  on integer constant`. Both bugs are known:
  [gcc](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=63337),
  [clang](https://bugs.llvm.org/show_bug.cgi?id=26910). MSVC handles it correctly. This may be due
  to the definition of `pp-number`s and is mentioned in the standard
  [https://eel.is/c++draft/lex.pptoken#example-2](https://eel.is/c++draft/lex.pptoken#example-2).
- Clang / LLVM internally can start doing non-multiple of 8 arithmetic in its internal representation (even without the
  use of `_ExtInt` or `_BitInt`). For example, [this code](https://godbolt.org/z/v49P6W38r) results in 33-bit arithmetic
  as a result of the optimizer identifying the loop induction.

## C++

- The size of an empty struct is `1`. This is because the C++ memory model guarantees disjoint
  storages (and thus disjoint addresses) for all distinct objects.
  [https://eel.is/c++draft/basic.memobj#intro.object-9.sentence-2](https://eel.is/c++draft/basic.memobj#intro.object-9.sentence-2)
- All types must be deduced the same in an `auto` declarator list. I.e. `auto x = 1, y = 1.5;` is
  not allowed.
- What would be idiomatic uses of `malloc` in C are UB in C++ prior to C++23, [more details here](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0593r6.html#idiomatic-c-code-as-c)
```cpp
struct S { int x; };
S* s = malloc(sizeof(S));
s->x = 1; // an object S hasn't been created and its lifetime hasn't started, placement new is required to make this well-formed
```
- C++ supports a set of alternative tokens such as `and`, `or`, `bitand`, `compl`, etc. which are
  equivalent to their primary counterparts. Truly, equivalent:
```cpp
struct S {
    S() = default;
    S(const S bitand) = delete;
    S(S and) = delete;
    compl S() = default;
}
void foo() {
    char b[sizeof(S)];
    new (&b) S();
    ((S*)b)->compl S();
}
```
- Vexing parse:
```cpp
// Vexing parse: This isn't a variable, it's a function declaration
T foo();
// Most vexing parse: This is still a function declaration (taking a T(*)())
T foo(T());
// "More vexing parse":
T foo(T((()))); // This is also a function declaration taking a T(*)()
T foo(T (((a)))); // this is a function declaration taking a T
// This is a variable definition
T foo((T()));
```
- C++ structs can have stray semicolons:
```cpp
struct S { ;;;;; };
```
- The following is The following code is probably, technically, well-formed in the current working draft of the
  standard (and may have been before too):
```cpp
template<typename T> void main(T) {}
int main() {}
```
  This is related to changes in P1787. Sadly, no compiler supports this.
- [Function try-blocks](https://en.cppreference.com/w/cpp/language/function-try-block) are a
  convenient way to wrap an entire function body with exception handlers and the only way to catch
  exceptions in member initializer lists:
```cpp
template<typename T> struct S {
    T t;
    S(const T& t) try : t(t) {
        ...
    } catch(...) {
        ...
    }
};
```
- `noexcept` is both a specifier and operator
```cpp
void foo() noexcept(noexcept(noexcept(true))) {}
```
- `throw()` is the same as `noexcept` since C++17.
- You can write `extern "C++"` as well as `extern "C"`, these are the only two standard linkage
  languages, but others can be defined by the implementation. Give us `extern "Python"` and
  `extern "Java"`!
- A declaration can have arbitrarily many linkage language specifiers:
```cpp
extern "C" extern "C++" extern "C" extern "C++" void foo(int) {}
```
The innermost specification is used. [https://eel.is/c++draft/dcl.link#5.sentence-2](https://eel.is/c++draft/dcl.link#5.sentence-2)
- The language grammar allows `for`-style `init-statement`s in `switch` and `if` statements, Since
  C++17:
```cpp
switch(int x = foo(); t[x]) { ... }
if(auto [a, b, c] = foo(); c) { ... }
// ranged for allows an init-statement too (just no iteration-expression)
for(auto [vec, map] = foo.bar(); const auto& item : vec) { ... }
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
- `goto` is disallowed in `constexpr` functions until C++23
- `static` storage local variables are not permitted in constexpr functions until C++23
- Structured bindings can't be used in constexpr declarations
- The following is a valid "hello world" implementation
```cpp
auto& hello_world = std::cout<<"Hello World"<<std::endl;
int main() {}
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
- `final`, `override`, `import`, and `module` aren't keywords but have special meaning in certain
  contexts. Thus, this is valid C++:
```cpp
struct final final {
    virtual final override() final { return {}; }
};
void final() {
    struct final typedef override;
    struct final final = override().override();
}
```
[https://eel.is/c++draft/lex.name#2](https://eel.is/c++draft/lex.name#2)
- There are special rules for lexing `<:` digraphs so that `std::vector<::std::string>` is lexed
  correctly and not as `std::vector[:std::string>`:
> Otherwise, if the next three characters are <​::​ and the subsequent character is neither : nor >,
> the < is treated as a preprocessing token by itself and not as the first character of the
> alternative token <:

[https://eel.is/c++draft/lex.pptoken#3.2](https://eel.is/c++draft/lex.pptoken#3.2)
- `std::numeric_limits<T>::max` and related functions are functions because there was originally
  concern that some values may not be available at compile time. E.g.
  `std::numeric_limits<float>::min` which was dependant on rounding mode. These functions are
  `constexpr` since C++11 but at that point it was too late to change them from functions.
- The original proposed syntax for lambdas looked like `<>(int x) : [y] (x + y)`
  (what's now `[y](int x) { return x + y; }`). `<&>(x) ( x * y )` or
  `<&>(x) -> int { return x * y; }` would have been the syntax for `[&](auto x) { return x * y; }`.
  Also, in the original proposal there was no mutable keyword for lambdas. Instead the call operator
  was always const and captures were always marked mutable. Initial proposal papers:
  [N1958](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1958.pdf),
  [N1968](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1968.pdf),
  [N2329 (N1968 rev 1)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2329.pdf).
- `std::string('0', '0')` is a string of 48 `'0'`'s, `std::string{'0', '0'}` is the string `"00"`
- James Bond was [added to the C++ standard in C++17](https://github.com/cplusplus/draft/commit/703d892264af814a64140b17ffe2bf6ae9274dde)
- The C++ standard contains a small poem:
  > When writing a specialization, be careful about its location; or to make it compile will be such a trial as to
  > kindle its self-immolation.

  https://eel.is/c++draft/temp.spec#temp.expl.spec-8
- CV qualifiers don't apply to objects [their construction is complete](https://eel.is/c++draft/class.ctor.general#5.sentence-2),
  and relatedly there are no cv-qualified constructors
- Array elements, and objects in general, are always destroyed in reverse order of construction. Standard quote for [arrays](https://eel.is/c++draft/class.dtor#14.sentence-5)
- A lambda's `operator()` is automatically `constexpr` if it meets the requirements for a constexpr function [https://eel.is/c++draft/expr.prim.lambda.closure#5.sentence-6](https://eel.is/c++draft/expr.prim.lambda.closure#5.sentence-6)

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
- Compiler can't decide which is correct, both are rejected by gcc:
```cpp
extern extern "C" extern "C++" int x; // accepted by clang (with warning)
extern "C" extern "C++" extern int x; // accepted by cland (with warning) and msvc (no warning)
```
GCC is correct. The second is more correct due to `linkage-specification`s, but, it's disallowed to
specify a storage class in a `linkage-specificaiton`
[https://eel.is/c++draft/dcl.link#8.sentence-2](https://eel.is/c++draft/dcl.link#8.sentence-2).
- Double `[[gnu::constructor]]`'s are ignored but they are still allowed on `main` so hello world
  prints twice here.
```cpp
[[gnu::constructor]] [[gnu::constructor]] int main() {
    puts("Hello, World!");
}
```

## C

- Source code of [the very first C compiler](https://github.com/mortdeus/legacy-cc).
- An empty struct is UB in C. Standard quote: 6.7.2.1.8 (C11-C23).
- A significant subset of possible identifiers are reserved in C. These include identifiers which
  begin with `is` or `to`, `str`, or `mem` followed by a lowercase letter in the global scope. It's
  undefined to declare/define a one of these reserved identifiers in the global scope. So, the
  following program may 1) print 1, 2) wipe your hard drive, 3) summon cthulhu, 4) other. All are
  behaviors are equally correct.
```c
#include <stdio.h>
int iseven(int n) {
    return n % 2 == 0;
}
int main() {
    printf("%d", iseven(2));
}
```
- Expressions in parameter declarations are evaluated by gcc/clang. Due to sequencing this prints
  number 1-10:
```c
#include <stdio.h>
int first = 0;
int main();
int main(int a, char *b[(first++ > 8) ? 1 : main()]) {
    printf("%d\n", first--);
}
```
- Similarly this is a valid "hello world" program in C
```c
int main(int, char*[puts("Hello World")]) {}
```
- `auto` is a keyword in C. Not to be confused with C++ `auto`, C `auto` does absolutely nothing.
- `extern const void x;` is valid a valid declaration in C for the same reason `extern struct S s;` is valid - `void` is
  an incomplete type
  - This is not valid in C++ because incomplete types in general are not allowed in extern declarations, incomplete
    class types are specifically explicitly permitted in [[dcl.stc]/7](https://eel.is/c++draft/dcl.stc#7)
- The following is valid C:
```c
signed _Noreturn const long volatile long static _Atomic inline f(void);
```

### Bugs and Implementation Quirks
- gcc allows completely empty case labels (C only):
```c
switch(x) { case 1: }
```
- gcc allows labels to be applied to declarations
```c
switch(x) { default: int y; }
switch(x) { default:; int y; } // must be this in clang
```
- This compiles [without error](https://godbolt.org/z/471Eh7sGc) in TCC
```c
static inline int foo(void) {
    [[[[[[[[{{(}));
}
int main(void) {
    return _Generic(1, int:0, float:((}}]]]);
}
```

## Talks
Some talks about C++ oddities:
- [Fun with (user-defined) attributes](https://youtu.be/Pt6oeIpzue4)
- [Can I has grammar?](https://youtu.be/tsG95Y-C14k)
- [C++ WAT](https://youtu.be/rNNnPrMHsAA)
- [Non-conforming C++: the Secrets the Committee Is Hiding From You](https://youtu.be/IAdLwUXRUvg)
