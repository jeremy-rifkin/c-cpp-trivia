This repository is a collection of neat C & C++ trivia and oddities.

# Both languages

- `0` is technically tokenized as an octal literal.
- `0XE+2` should evaluate to `16`, however, both gcc and clang give an error: `invalid suffix "+2"
  on integer constant`. Both bugs are known: [gcc][gcc-int-suffix] [clang][clang-int-suffix]. MSVC
  handles it correctly. This may be due to the definition of `pp-number`s and is mentioned in the
  [standard][standard-int-suffix].
- `sizeof(0)["abcd"]` is `1`.

[gcc-int-suffix]: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=63337
[clang-int-suffix]: https://bugs.llvm.org/show_bug.cgi?id=26910
[standard-int-suffix]: https://eel.is/c++draft/lex.pptoken#example-2

# C

# C++
