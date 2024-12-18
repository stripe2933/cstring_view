# cpp_util::cstring_view

[![Test with GCC, Clang and MSVC](https://github.com/breyerml/cstring_view/actions/workflows/compiler_test.yml/badge.svg)](https://github.com/breyerml/cstring_view/actions/workflows/compiler_test.yml)
[![Codacy Badge](https://app.codacy.com/project/badge/Grade/86dcbe857c904f91bc84f3d33b3aadb5)](https://app.codacy.com/gh/breyerml/cstring_view/dashboard?utm_source=gh&utm_medium=referral&utm_content=&utm_campaign=Badge_grade)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Implementation of [P1402R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1402r0.pdf) for a `cstring_view`.

Mimics (and delegates most of its functionality to) a `std::basic_string_view`, but removes all functions that could
potentially break the invariant of null termination.

## Changes compared to P1402R0

- The conversion from a `std::string` to a `basic_cstring_view` is implemented using a special constructor, since it is
  not possible to add a function to the `std::basic_string` class.

- Moved the `null_terminated_t` into the `basic_cstring_view` class definition.

- Enabled all `char8_t` functionality only conditionally if the feature test macro `__cpp_char8_t` is defined.

- Renamed the suffix literal for a `basic_cstring_view` from `csv` to `_csv` since only standard library suffixes can
  omit the `_`.

- Corrected `constexpr string_view_type substr(size_type pos = 0, size_type n) const;` and removed the default parameter
  from the first parameter.

- Enabled `starts_with` and `ends_with` delegation to `std::string_view` only if the feature test macro `__cpp_lib_starts_ends_with` is defined, otherwise a
  custom implementation based on *cppreference* is used.

- Added `contains()` functions proposed for C++23.

- Added custom `assert()` to various functions.

- Implemented the comparison functions using `operator==` and `operator<=>` if the feature test macros `__cpp_impl_three_way_comparison` and `__cpp_lib_three_way_comparison` are defined.

- Added ranges helper templates, which are only enabled if the feature test macro `__cpp_lib_ranges` is defined.

- Added support for `std::format` if the feature test macro `__cpp_lib_format` is defined.

- Added `[[nodiscard]]` to almost all functions.

## Prerequisites

Any compiler supporting `C++17` should be sufficient (for more information see [Compiler Support](#compiler-support)).
Additionally, at least [CMake](https://cmake.org/) `3.30` is required.

The tests are implemented using [Catch2](https://github.com/catchorg/Catch2/tree/v2.x), which gets shipped as single header file with this repository.

## Building and Running the Examples

Building with GCC, Clang, or MSVC can be done using CMake presets.

```bash
git clone git@github.com:breyerml/cstring_view.git
cd cstring_view
cmake --preset [preset] -DCPP_UTIL_BUILD_EXAMPLES .
cmake --build --preset [preset]
./build/cstring_view_examples
```

Where `[preset]` is one of `gcc`, `clang`, or `msvc`.

## Building and Running the Tests

To additionally build the tests and run them use:

```bash
cmake --preset [preset] -DCPP_UTIL_ENABLE_TESTS .
cmake --build --preset [preset]
ctest --preset [preset]
```

Tests for all supported `C++` standards starting with `C++17` for the currently used compiler are generated.

## Compiler Support

The `cpp_util::cstring_view` has been tested with the following compilers, all installed using the respective package
manager and GitHub actions (other compilers or compiler versions may also be supported):

| Compiler | Versions                               |
|----------|----------------------------------------|
| GCC      | 9.5.0, 10.5.0, 11.4.0, 12.3.0          |
| Clang    | 11.1.0, 12.0.1, 13.0.1, 14.0.0, 15.0.7 |
| MSVC     | 19.29.30154.0                          |

All tests were run using the following compiler flags:

- GCC: `-Wall -Wextra -Wpedantic -Werror`

- Clang: `-Weverything -Wno-c++98-compat -Wno-c++98-compat-pedantic -Wno-unknown-warning-option -Wno-zero-as-null-pointer-constant -Wno-reserved-identifier -Werror`
  - `-Wno-c++98-compat -Wno-c++98-compat-pedantic`: since `C++98` is not supported
  - `-Wno-unknown-warning-option`: due to `clang-11.1.0` and `clang-12.0.1` not recognizing the Wno-reserved-identifier option
  - `-Wno-zero-as-null-pointer-constant`: due to a warning generated by `clang11.1.0` in the Catch2 header
  - `-Wno-reserved-identifier`: due to a warning generated by `clang-13.0.1` when using the custom `_csv` literals

- MSVC:  `/W4 /EHsc /WX`

## Compiler Standard Version

- `C++17`:
  - the minimum required standard version enabling all functions

- `C++20`:
  - the implementation of the `starts_with` and `ends_with` functions is now delegated to the respective `std::string_view` implementation
  - the relational operators are now implemented in terms of the three-way comparison operator `operator<=>(const basic_cstring_view&, const basic_cstring_view&)`
  - add support for `char8_t` string views
  - add support for std::ranges

The actual features are enabled using the specific features test macros and not the `__cplusplus` macro.
