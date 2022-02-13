# TESA: Terse, Elegant, Symbolic Ascii

## Table of Contents

 - [Table of Contents](#table-of-contents) (See Also: recursion)
 - [Manifesto](#manifesto)
   - [Terse](#terse)
   - [Elegant](#elegant)
   - [Symbolic](#symbolic)
   - [Ascii](#ascii)

## Manifesto

I've worked with a lot of programming languages, and like some features not others. This section will outline the four design principles of TESA, for which it is named. Note the following should not be taken as a condemnation of certain styles of programming but rather as an expression of my personal preference.

### Terse

Some languages are long:
```java
public class FizzBuzz {
    public static void main(String[] args) {
        for (int number = 1; number <= 100; number++) {
            if (number % 15 == 0) {
                System.out.println("FizzBuzz");
            } else if (number % 3 == 0) {
                System.out.println("Fizz");
            } else if (number % 5 == 0) {
                System.out.println("Buzz");
            } else {
                System.out.println(number);
            }
        }
    }
}
```
([Java](https://www.oracle.com/java/))

Some languages are short to the point of unreadability:
```jelly
³µ3,5ḍTị“¡Ṭ4“Ụp»ȯµ€G
```
([Jelly](https://github.com/DennisMitchell/jellylanguage)) (Example from [here](https://codegolf.stackexchange.com/a/70216))

Some languages are long *and* unreadable:
```shakespearelang
The Elucidation of Foul Multiples.

Romeo, the hopeless romantic.
Mercutio, the grave man.
Prince Henry, the noble.
Ophelia, the drowned.

                    Act I: The Revelation Of Wretched Multiples.

                    Scene I: Romeo The Sweet Talker.

[Enter Prince Henry and Romeo]

Romeo: 
  You are as rich as the sum of a handsome happy honest horse and a lovely fellow. 
  Thou art the square of thyself.

[Exit Prince Henry]

[Enter Ophelia]

 ... (Continues for 91 lines)
```
([The Shakespeare Programming Language](http://shakespearelang.sourceforge.net/)) (Example from [here](https://github.com/HenryDangPRG/FizzBuzzShakespeare))


TESA aims to be short *while still being legible*:
```tesa
({{each [n] ->
  ((((n % 3) => "" "Fizz") ++ ((n % 5) => "" "Buzz")) lor {repr n})
} ({<-> 100} + 1)} join "\n")
```

This is, of course, a highly subjective metric, but I believe TESA does a good job of fulfilling it.

### Elegant

TESA strives to be elegant. Given its highly work-in-progress state its success in this may vary. Tesa specifically aims to be elegant in a number of key ways:

 - **Unambiguity**
   - Bracketing is used to indicate order instead of operator precedence which must be memorized. (There are a few exceptions to this for brevity's sake but the rule holds in general.)
   - Mandatory whitespace separates different tokens (so nothing like the classic `while (i-->0)` is possible) but since this is always used other noisy separators, such as `,` or `;`, are not required.
 - **Different things are different and Equivalent things are equivalent**
   This is largely a function of the type system but extends into other areas.
   
   - Fundamentally different objects are recognized as such.
     `123` is a number and `"123"` is a string. The two types are fundamentally different.
     
     You will never find that your `123` has suddenly turned into a `"123"` or vice versa or that `(123 matches "123")` (I'm looking at you Javascript)
     
     If you do need to turn a string into a number or a number into a string you can do so explicitly (`{repr 123}` or `{num "123"}`) or implicitly when and only when a type cast is the only logically desired effect (ie `{print 123}`)
   
   - Fundamentally equivalent objects are recognized as such.
     `"Abc"` is a string and `['A' 'b' 'c']` is an array of characters. The two objects are fundamentally the same.
     
     That is: there isn't generally a reason to distinguish between the two.
     
     Therefore in TESA a string *is* an array of characters making `"Abc"` and `['A' 'b' 'c']` just different ways of constructing the same object in the same way that `3.14` and `(3 + 0.1 + 0.04)` are different ways of constructing the same object.
 - **Providing Powerful, Generalized, Features**
   - Multidimensional and Infinite Arrays
     Multidimensional arrays can be constructed in a variety of ways
     ```tesa
     {merge [
       [1 2 3]
       [4 5 6]
       [7 8 9]
     ]}
     ```
     constructs the true two dimensional array
     ```
     ┌─
     ╵ 1 2 3
       4 5 6
       7 8 9
             ┘
     ```
     (by the way if you like this multi-dimensional array representation it was taken from [BQN](https://mlochbaum.github.io/BQN/))
     
     Similarly, infinite arrays are supported
     ```tesa
     (2 ^ {<-> oo})
     ```
     constructs the infinite list
     ```
     [ 1 2 4 8 16 32 64 128 256 512 1024 2048 4096 8192 16384 32768 ··· ]
     ```
   - Functional Programming
     Functions are values and thereby can be passed around allowing you to do things like:
     ```tesa
     \/ => (-< - + *)
     ```
     which is equivalent to the more traditional but lengthier
     ```tesa
     \/ => [a b] -> ((a + b) - (a * b))
     ```
   - Pervasive and Character Arithmetic
     Arithmetic can be done on arrays too:
     ```tesa
     ([1 2 3] + 1)
     ```
     is
     ```
     [2 3 4]
     ```
     and
     ```tesa
     ([6 5 4] - [3 2 1])
     ```
     is
     ```
     [3 3 3]
     ```
     and
     ```tesa
     X => {merge [
       [1 2 3]
       [4 5 6]
       [7 8 9]
     ]}
     Y => [1 10 100]
     (X * Y)
     ```
     is
     ```
     ┌─
     ╵   1   2   3
        40  50  60
       700 800 900
                   ┘
     ```
     
     And a limited subset of arithmetic (namely `+` and `-`) also work for characters:
     ```tesa
     'C' - 'A'
     ```
     is
     ```
     2
     ```
     and
     ```tesa
     "The quick brown fox jumps over the lazy dog." - '\0'
     ```
     is
     ```
     [ 84 104 101 32 113 117 105 99 107 32 98 114 111 119 110 32 ··· (28 more) ]
     ```
     and this system can be used to do things like
     ```tesa
     ({<-> 26} + 'a')
     ```
     which is
     ```
     "abcdefghijklmnopqrstuvwxyz"
     ```
   - Generalizations of Common Functions
     A number of common functions are generalized
     
     `+`, `*`, and others can take any number of arguments:
     ```tesa
     {+ 3 14 159}
     ```
     is valid and results in
     ```
     176
     ```
     
     Functions like `each` and `over` are generalized beyond their standard definitions.
     For instance `each` can act like a traditional `each` or `map` function or like a `zipWith` function depending on the number of arguments.
     ```tesa
     {{each print} [1 2 3]}
     ```
     prints
     ```
     1
     2
     3
     ```
     ```tesa
     {{each ,} [1 2 3] [4 5 6]}
     ```
     is
     ```
     [ [ 1 4 ] [ 2 5 ] [ 3 6 ] ]
     ```
     
     Furthermore functions like `each` and `foldr` take their functional arguments and list arguments in two separate calls allowing partial application for pointfree code:
     ```tesa
     printall => {each print}
     ```
     is the same as
     ```tesa
     printall => [arr] -> {{each print} arr}
     ```

### Symbolic

TESA makes use of multi-character symbols or "ligatures" in order to represent a variety of functions, constants, and others more succinctly without having to use non-ascii characters.

 - `oo` is infinity
 - `<->` is range
 - `\/` is or
 - `-<` is a fork
   and many more

While the term ligature is not strictly accurate to describe simply a multi-character sequence it is recommended that you edit TESA code with a font that renders common multi-character sequences as true ligatures. The ligatures are however designed to be readable without any fancy fonts for those cases where they cannot be used.

### Ascii

This is largely a manner of accessibility.

While much of unicode can be rendered by most modern computers it is still unsupported or only partially supported by:
 - Legacy systems
 - Many monospace fonts
 - Automatic tooling
   and others.

It is also more difficult to input non-ascii characters into a computer, making writing code in a language that uses them slower and more annoying.

That's not to say that TESA is opposed to non-ascii characters as it in fact quite happily supports the entirety of unicode:
 - Characters in TESA can represent any 32 bit unicode code-point
 - Character and string literals can include unicode characters just fine:
   `'ñ'` and `"cos(α) + cos(β)"` are both valid.
 - Unicode characters are valid in identifiers
   `Δt`, `x̄`, and `∫` are all valid variable names
 - TESA uses UTF-8 as it's primary text encoding method for source files

All this restriction means is that no TESA builtin will include non-ascii characters nor will the standard library.