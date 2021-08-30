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

# C

- Source code of [the very first C compiler](https://github.com/mortdeus/legacy-cc).

# C++

- `decltype(std)` is an `int` in gcc. Bug reports:
  [#1](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=100482),
  [#2](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=101213).
- Prior to gcc 10, `decltype(decltype(decltype))` could be used to generate [exponential error
  messages](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=92105).
