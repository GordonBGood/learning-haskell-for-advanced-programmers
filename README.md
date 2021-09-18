# Learning Haskell for Advanced Programmers

There are no end of tuturials and self-help documents on the Integet to walk inexperienced beginners who want to dabble in Haskell up to being able to easily write their own basic to more advanced code.  This document is for programmers who already know several reasonably advanced languages such as Rust or Nim or even DotNet languages such as C# or F# or also JavaVirtual languages such as Kotlin, Clojure, or Scala, who want to dabble in the most advanced Functional Programming concepts which are a part of Haskell.  As GHC Haskell is the only active Haskell compiler development at this time (mid 2021), everything taught will be related to newer GHC Haskell compilers.

First, to dispel a common fallacy that one may have seen on the Internet:  Haskell is a dying language because its last stable version was 2010.  **This is wrong**.  Haskell 2010 is a committee generated specification describing the minimum features that all Haskell implementations conforming to the specification must provide and extensions that they mey optionally support, and at some point in the next few years will likely be superceded, but since there is only one active Haskell development as the in Glorious Glasgow Haskell Compiler (GHC), there isn't a lot of point as GHC is the specification of what Haskell represents.  GHC is in active development with long term support versions (latest version 8.8.4 from 2019), an ordinary stable version currently as version 8.19.7 (from August 2021), and an advanced stable version with all the latest features as in version 9.0.1 released at the beginning of 2021. There are new minor versions released roughly every year and minor "bugfix" realease more often as the minor versions are improved.  So no, Haskell is not dead or even close to it.

There are a few things that Haskell will likely never be and one is that it will likely never be a successful mainstream language.  That is because it was never planned to be such, as it was designed by mostly academics as mostly a reseach vehicle investigating Functional Programming paradigms; that said, it is used by some companies commercially.  The thing that will likely prevent it from ever becoming a mainstream language is that for beginners and programmers only experienced in interpretive dynamically typed languages such as Python, the learning barrier to entry is quite steep and requires a large investment in effort and time.  For those, I say learn another FP language first, and recommend Elm as the simplest pure FP language out there with a reduced compulsory feature set that makes it easy to learn.  Other options might be F# or a Lisp/Scheme base language such as Racket.  Then, come back here after you are a reasonably advanced programmer in one of these languages.

## Introduction

This section explains the learning approach for advanced programmers; unlike in tutorials for beginners, it doesn't start with a "Hello World" program (the student can find those on the Internet themselves) but rather will build a knowledge from the "ground up" that we demontrate GHC Haskell's relationship to the CPU machine, and through understanding that relationship, the relationship it may share with other languages and those features which are entirely different.  Haskell is a very high level language embracing esoteric features, and thus many programmers using it never even consider the low level machine operations that have been hidden by multiple levels of abstractions.  That is a problem, as they then have no concept of the cost in resources and or execution time in using such abstractions.  This series of documents seeks to give a better understanding of how the high level abstractions relate to the lowest level CPU instructions.

Since this approach is completely opposed to most Haskell tutorials that teach the mechanics of syntax and higher order abstractions, then only maybe mention some of how these relate to low level operations, the order of this instruction will be completely reversed from most Haskell tutorials and perhaps a little more like more conventional imperative languages in starting with the lowest level "primitive" machine types and advancing to the higher order abstractions step by step.

Haskell is all about statically controlling and manipulating types, so we'll start there.

## Primitive Types in Haskell

As mentioned in the Introduction, this is about starting from the lowest and advancing, the lowest type that needs to be understood in Haskell doesn't take any memory at all and only exists as a concept:  the "bottom" type which represents voidness/nullness/emptiness - thus "bottom" - and can only be written both as a type and an instance of that type as `()`;  In other C-like languages, this is sometimes thought of ambiguously as being "void" but with a "void" pointer being a pointer to "what's at address zero" which for most CPU's and Operating Systems isn't anything one can use or can legally access.  That isn't what it represents in Haskell as there is no such thing as a null pointer (other than for purposes of exchanging data with those other languages, in which case artificial to Haskell means are used to represent this).  So that's the first and most important thing about Haskell - there are types that can never be represented on the "real" machine.  Later, we will also see types that **could** be represented on a real machine but in Haskell likely never are (except maybe for debugging purposes) as in ephimeral "guost" types that may conceptually be a kind of container of other types but which, at least for production code, the compilere can optimize entirely away.  Keep that in mind.

Next, let's consider types that represent the real bytes and words that take physical storage:  In ordinary Haskell code these aren't accessible directly at all!  All "ordinary" types in Haskell are "lifted" types, which means they aren't something that can be represented by a simple range of one or more bytes in memory, but represent the intention that eventually one will be able to access something that represents those bytes in memory - this is Haskell's non-strictness or laziness; types may exist only as functions of that are able to create them until they are actually required.  This applies to all types, even a type as basic as an integer.  So what is an `Int` then?  Well, first we have a standard the the primitive that it represents has a size, and that size is the word size of the CPU architecture on which the code is run, so for 32-bit architectures thats 32 bits and for 64-bit architectures it's 64; we also knkow it is signed (as opposed to unsigned) so the Most Significant Bit (MSB) is reserved for the two's complent sign (for most machine implementations) being one for negative numbers.  But, being "lifted" it isn't the contends of those four or eight bytes; the "lifted" type puts the representation into a container, and that container is `Int`.  Now in some lesser implemtnations (and perhaps for interpreted Haskell), that container may be some sort of class, structure, or record in memory, but it doesn't have to be - it could be one of those ephimeral "ghost types that disappear when the code is compiled.

So, before we can go any further, we need to understand about data and data constructors such as this `Int` representing some primitive bytes in memory.  Data is decribed with a "data" definition, it's as simple as that, but there is a lot going on even with that simple concept: to define `Int` we just need `data Int = I# Int#)`, which is exactly how it is defined in the Haskell source code.  So `data` one can understand as defining a new data type called `Int`, what's going on in the reslt of the line?  The `I#` is what is called the constructor of the `Int` type.  So what is a constructor?  A constructor is a function taking as an argument the thing(s) to be contained in the type, if any; in this case it contains the primitive byte representation of what the CPU actually uses as a machine int, calle `Int#` and that is another Haskell type that looks like it is high level but actually only exists a a low level representation of the bytes, which requires some "compiler magic" in C code to implement.  Okay, that makes sense, but what about all those hash mark symbols?  Hash mark sysbols used like this represent the real machine primitives, but they in turn need their own constructor so that the contents can be handled by the higher level code and converted back and forth between the "lifted" types and these "unlifted" forms.  So the first cpitalized `I#` is the primitive constructor function that lifts the real capitalized `Int#` representing the real bytes into "lifted" space.  This is capitalized as are all Haskell types because it represents a high-level type to high-level Haskell code even though it is actually represented as just low level bytes.  Whew!

As normal high level Haskell code never deals with the lowest primitive representative, one normally doesn't see or use hash mark symbols and the ability to use them is turned off by default for safety.  In order to use them, we need to turn on a GHC Haskell extension called "MagicHash"; this can be done as a compiler command line option as `-XMagicHash` or in the source code itself with a pragma as `{-# LANGUAGE MagicHash #-}`, which can also contain a list of required extensions separated by commas.  Since it makes sense at this point, comments may as well be explained: single line comments are anything that follows two dashes as in `--` to the end of the line; multi-line comments are enclosed in `{- ... -} symbols and can be nested.  Oho, that means the `LANGUAGE` pragma is actually used inside what would otherwise be a regular possibly-multi-line comment and that's for a reason - if there were other dialects that didn't support pragmas then they wouldn't even see them, for GHC that does know pragmas, it looks for the extra enclosing hash marks and then starts looking for the specific pragma, in this case `LANGUAGE` which means language extensions.

There is another neat thing about using multi-line comments like this:  if one wants to comment out multi-lines temporarily, they can just add a line containing `{-` before the commented out block and `-} afterwards, but to be clever let's put `--}` in the following line instead.  Now to uncomment the commented block, we can just add a double dash before the first enclosing mark as `--{-` and the commented out block is revealed because both of the added block comment lines now become single line comments with no comments other than code comments between.  Because block comments can be nested, this does not affect other enclosed block comments which continue to behave as intended.

So back to the subject of our `Int` type enclosing a "real" `i#` primitive as a number of bytes, there is some small amount of code handling these kinds of primitive types that can't be written in Haskell as it's too low level and so is written in C, and that code must be compiled using a C compiler in order to "bootstrap" the Haskell compiler from pure source code, every level past "bootstrap" can be handled by any version of GHC Haskell building the rest of the new version as the rest of the code is in high level Haskell code.  That low level primitive C code is inside the "" file

To complete the basic description of the lowest level of primitives, let's test some code using this, as follows:

```haskell
{-# LANGUAGE MagicHash #-}

-- import parts of the library for low-level primitives;
-- import the Int data class and its constructor,
-- the Int# primitive, and the primitive Int# addition operator...
import GHC.Exts ( Int(I#), Int#, (+#) )

cTESTINT = I# (42# +# 7#) -- makes the Int from low level manipulations

main = print cTESTINT -- prints 49
```
At the end is a line you should understand intuitively as a main function that prints the `Int` result.

Alternatively, this could be tested with the Read/Evaluate/Print/Loop interpreter `ghci` as follows (all lines followed by enter key):
1. With GHC Haskell installed, from a command prompt type `ghci -XMagicHash` to see the following:
```
GHCi, version 9.0.1: https://www.haskell.org/ghc/  :? for help
ghci>
```
2. Type `:m +GHC.Exts` to import all of the "GHC.Exts" library.
3. Type `cTESTINT = I# (42# +# 7#)` for the primitive computation.
4. Type `cTESTINT` to show the result
5. Type `:quit` to leave the REPL as one can find out how to do from the `:?` command.
6. The sequence should look like the following:
```
ghci> cTESTINT = I# (42# +# 7#)
ghci> cTESTINT
49
ghci> :quit
Leaving GHCi.
```
which is exactly as if run as a program above.

Rather than producing the first source file with a `.hs` extension with an editor, compiling it using `ghc <name_of_source_file>`, then running with with `./<>name_of_source_file_without_the_extension>` you can use an on-line IDE [such as Wandbox](https://wandbox.org/permlink/H6Nw3klCU5wGcK78).

To do the calculation at a high level rather than a primitive level, one would just substitute `42 + 7` for `I# (42# +# 7#)`, with the second exactly what the GHC compiler is turning the first into.
