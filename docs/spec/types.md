# TESA Type System

## Overview

TESA recognizes six (and only six) distinct types.

 - None
 - Number
 - Character
 - Array
 - Function
 - Container

Of these types the first four are immutable[^1] and the last two are mutable[^2].

Certain common types are not included in TESA, but these generally have replacements:

 - Ints and Floats are not separated.
 - Booleans are just the numbers `0` and `1`. The standard library also provides the constants [`FALSE`](./stdlib.md#false) and [`TRUE`](./stdlib.md#true).
 - Strings are just one dimensional arrays of characters.
 - Objects, Maps, and various other types that associate keys to values are represented by the single container type.

To get the type of an object in TESA the [`typeof`](./builtins.md#typeof) function can be used. The exact return set of values for this function is left up to the implementation to determine but it is guaranteed that there will be six distinct values which can be compared using the `=`, `=/=`, [`match`](./builtins.md#match), and [`notMatch`](./builtins.md#notmatch) functions. In practice implementers will want to use some sort of numerical enum or set of representation characters for the distinct values. The [`TYPE`](./stdlib.md#type) container in the standard library contains the variables `NONE`, `NUMBER`, `CHARACTER`, `ARRAY`, `FUNCTION`, and `CONTAINER` which can be compared against. (i.e. `{{typeof x} = TYPE.NUMBER}`) to check if a value is a number.

A few useful cast functions are provided:

 - [`repr`](./builtins.md#repr), short for representation, converts an object of any type into a string. It can optionally take an additional second argument specifying formatting details.
 - [`num`](./stdlib.md#num) converts a string to a number or returns `·` if no such conversion is possible. It can optionally take a second argument specifying parsing rules.
 - [`bool`](./builtins.md#bool) converts an object of any type into either the number `0` or `1`. It cannot fail.

In some contexts an implicit cast to either a string (as is the case in [`print!`](./builtins.md#print)) or a boolean (as in [`lor`](./builtins.md#lor)) is unambiguously desired. In these cases the [`repr`](./builtins.md#repr) and [`bool`](./builtins.md#bool) functions are used internally.

It should also be noted the following underlines the *user-facing* types of the language. Implementations may use more than one internal type per user facing type under the hood for various reasons including convenience and efficiency. However, these extra internal types should not be visible to user-programs.

[^1]: Arrays in TESA are considered immutable in the sense that the elements in an array cannot be changed, however arrays can contain mutable elements which in turn can change.

[^2]: Containers are mutable in the trivial sense that their elements can be updated, functions are mutable in the sense that they can depend on and update mutable variables in an enclosing scope allowing their behavior to change over the course of a program.

## None

None is the simplest type in TESA, having one possible value also called None. None is implicitly the result any time a value is expected but none is provided. It is the value of:
 - An empty expression `()`.
 - An unprovided function parameter.
 - An unset field in a container.
 - The [`NONE`](./stdlib.md#none) constant from the standard library
   
   And more.

As such it is not possible to distinguish between an explicit none provided by user-code or one implicitly generated by the lack of any other value. This is a good thing as it allows setting a value to none to delete it instead of having to have an extra explicit notation for that purpose.

### Literals

There is no literal for None so the most common ways of explicitly obtaining the value are by using an empty expression `()` or using the constant `NONE` from the standard library.

### Type Conversions

 - `{repr ()}` is `"·"` (that's the middle dot).
 - `{bool ()}` is `0`.

## Number

TESA uses its own fancy number format with the following specs:
 - Arbitrary precision integers are supported.
 - Non-integer numbers with up to 53 bits of precision (the same as an IEEE-754 64 bit float) are supported. (An implementation may provide additional precision, but 53 bits is the minimum)
 - Zero is unsigned so both `0` and `-0` produce the same interval value.
 - The two signed infinities `oo` and `-oo` are both supported.
 - There is no NaN value, instead various functions may choose instead choose to:
   - Normalize indeterminate forms. (i.e. `(0 / 0)` gives `0`)
   - Return `·` instead. (i.e. `{num "Abacaba"}` gives `·`)
   - Simply fail.

### Literals

Numbers can be constructed in a variety of ways:

   The usual:
 - Decimal Literals: `123.45`
 - Hex Literals: `0x21.a` (unlike other languages TESA supports radix points in its non decimal literals)
 - Binary Literals: `0b11101010`
 - Octal Literals (both `0o` and `0` prefixed): `012` or `0o12`
 
   And the slightly more unusual:
 - Senary/Seximal (base-6) Literals: `-0s45`
 - Duodecimal/Dozenal (base-12) Literals: `0d9↊.↋1` (`a`/`A`, `t`/`T`, `x`/`X`, and `↊` can all by used as the digit for 10 and `b`/`B`, `e`/`E`, `z`/`Z`, and `↋` as the digit for 11)
 - Base 36 (full-alphabet) Literals: `0aFooBar1`
 
   Note that all bases > 10 are case insensitive
 - Infinity Literals: `oo` and `-oo`

### Type Conversions

 - `{repr n}` is the decimal representation of n. If n is negative this will include a leading `-` and if it is not an integer it will include a radix point `.`. The two infinities have dedicated representations.
   - `{repr 12}` is `"12"`
   - `{repr -81}` is `"-81"`
   - `{repr 3.14}` is `"3.14"`
   - `{repr oo}` is `"∞"` and `{repr -oo}` is `"-∞"`
   
   A second parameter can be provided to `repr` to adjust this behavior, see [`repr`](./builtins.md#repr) for details.
 - `{bool n}` is `0` if and only if `n` is `0`, otherwise it is `1`.

## Character

A character in TESA represents a single unicode code-point. This means that strings can be thought of as being in UTF-32 for convenience.

### Literals

 - Character literals are wrapped in single quotes (`'`) (i.e. `'x'`) and can use escape sequences. (i.e. `'\0'`)
 - String literals represent one dimensional arrays of characters, and are wrapped in double quotes (`"`) (i.e. `"FooBar"`) and can also use escape sequences.

Escape sequences start with a backslash (`\`) and have the following options. (Stealing from C, like everyone else)

 - `\a`, `\b`, `\e`, `\f`, `\n`, `\r`, `\t`, `\v` for ␇, ␈, ␛, ␌, ␊, ␍, ␉, and ␋ respectively.
 - `\\`, `\'`, and `\"` for `\`, `'`, and `"` respectively.
 - `\n`, `\nn`, and `\nnn` for the character with the corresponding 1-3 digit octal code point.
 - `\xdd` for the character with the corresponding 2 digit hexadecimal code point.
 - `\udddd` for the character with the corresponding 4 digit hexadecimal code point.
 - `\u{digits}` for the character with the corresponding n digit hexadecimal code point. (Actually, this one is stolen from Javascript)

### Type Conversions

 - `{repr c}` is generally `"'c'"`. If `c` is non-printable or must be escaped an escape sequence is used prioritizing escape sequences in the order laid out in the literals section.
 - `{bool c}` is `0` if and only if `c` is the null character (`'\0'`), otherwise it is `1`.

## Array

An array is TESA in a rectangular data structure with an arbitrary number of dimensions. Along each dimension or "axis" there can be any non-negative integer number of cells. (Including infinity as a possible number of cells) The elements of an array can be of any type (including other arrays) and can be of mixed type. A number of builtin functions are available to get key properties of an array.

 - `#` gets the number of items in an array.
 - [`length`](./builtins.md#length) gets the number of items in the leading axis of an array.
 - [`shape`](./builtins.md#shape) gets the a list of the lengths of the axes of an array.
 - [`rank`](./builtins.md#rank) gets the number of axes in an array.

### Literals

Lists (1d arrays) can be constructed using square brackets (`[` and `]`).

 - `[1 2 3]` is the list `[ 1 2 3 ]`
 - The elements can be expressions. (i.e `[(2 * 3) ("ABC" + 1) ()]` is `[ 6 "BCD" · ]`)

In order to create arrays of other shapes functions such as [`merge`](./builtins.md#merge) and [`reshape`](./builtins.md#reshape) must be used.

### Type Conversions

 - The `repr` of an array is highly complicated. See [`repr`](./builtins.md#repr) for details.
 - `{bool a}` is `0` if and only if `{# a}` is `0`, otherwise it is `1`.

## Function

A function in TESA is a first class value taking in zero or more values and returning some value as the result of an expression. Functions in TESA may or may not be pure functions (which can be tested with the [`pure?`](./builtins.md#pure) predicate) and they can also be forced to be pure using [`latch`](./builtins.md#latch). A function in TESA is considered to be pure even if it contains side effects within its body as long as those side effects do not leak.

### Literals

Functions are constructed using a single arrow `->` with a pattern on the left side representing the function's arguments as a list, and an expression on the right.

 - `[a b t] -> ((a * t) + (b * 1 - t))` is the lerp function.

### Type Conversions

 - `{repr f}` is `"(->)"`.
 - `{bool f}` is `1`.

## Container

A container in TESA is a mapping from keys (which can be of any type) to values. A container may contain immutable keys, mutable keys, or a mixture and may allow additional keys to be added by making the default key (`·`) mutable.

### Literals

Containers are constructed using square brackets, like arrays, but with double arrows (`=>` or `/=>`) inside them associating keys and values. The keys may be either patterns or expressions (defaulting to the former). If patterns are used the keys are interpreted as strings. Using none as a key in the definition doesn't create an actual pairing but does make it so additional keys can be added if it is set as mutable.

 - `[=>]` is an empty container (using autocompleted pattern `__` and autocompleted expression `()`).
 - `['a' => 0 'b' => 1 'c' => 2]` contains three character keys.
 - `[foo => "bar" baz => 17]` is equivalent to `["foo" => "bar" "baz" => 17]`.
 - In `[(foo) => bar]` both the key and value are determined by expressions. The key is not `"foo"` but the value referenced by `foo`.
 - `[/=>]` is an empty container which allows appending additional keys.

### Type Conversions

 - `{repr C}` is generally similar to a container literal but this can be changed by adding a `USR-REPR-FN` key to the container. See [`repr`](./builtins.md#repr) for details.
 - `{bool C}` is `0` if and only if the container contains no keys set to a non-none value. (As none keys are equivalent to unset keys) It is `1` otherwise.