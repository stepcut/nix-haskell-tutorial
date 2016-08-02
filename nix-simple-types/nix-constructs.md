Nix Language Constructs
=======================

So far we have looked at various ways of creating data values. That might be enough for creating config files, but we want a programming language!

let-expression
==============

The `let` expression is used to bind an expression to a variable:

    nix-repl> let x = "1"; in "foo-${x}"
    "foo-1"

    nix-repl> let x = 1; y = 2; in x + y
    3

The `let` keyword is followed by 1 or more bindings `<var> = <expr>;` followed by `in <expr>`. The variables are only bound in the expression following the `in` clause.

    nix-repl> let x = "1"; in x
    "1"

    nix-repl> x
    error: undefined variable ‘x’ at (string):1:1

`let` is lazy, which means we can do the following:

    nix-repl> let x = "1"; y = throw "die!"; in x
    "1"

Since we never evaluate `y` the exception is not thrown.

Unlike sets, let expressions are recursive by default. So we can write:

    nix-repl> let x = y; y = 1; in x
    1

functions
=========

A function has the form:

    pattern: body

In Nix functions are always anonymous, have one argument, and return a value.

variable pattern
----------------

We can write the id function as follows:


    nix-repl> (x: x) 1
    1

That is the same as the Haskell code:

    GHCi> (\x -> x) 1
    1

We can use let to bind the function to a name:

    nix-repl> let id = x: x; in id 1
    1

That is the same as the Haskell code:

    GHCi> let id = \x -> x in id 1
    1

To simulate a function of multiple arguments we can write:

    nix-repl> let add = x: y: x + y; in add 1 2
    3

That is the same as the Haskell code:

    Prelude> let add = \x -> \y -> x + y in add 1 2
    3

Like the Haskell code, it works because it is defining a function which takes one argument, and returns a function which takes the second argument:

    nix-repl> let add = x: (y: x + y); in add 1
    «lambda»

set pattern
-----------

The set pattern matches on a set with the named attributes:

    nix-repl> let sumXYZ = {x, y, z}: x + y + z; in sumXYZ { x = 1 ; y = 2; z = 3; }
    6

Because we are matching on a set, and the order of elements in a set does not matter, the following is also valid:

    nix-repl> let sumXYZ = {x, y, z}: x + y + z; in sumXYZ { z = 3; y = 2; x = 1 ; }
    6

Note that the value passed to sumXYZ *must* be a set with exactly 3 attributes, and the attributes *must* be named `x`, `y`, and `z`.

Calling `sumXYZ` with three integer arguments will not work:

    nix-repl> let sumXYZ = {x, y, z}: x + y + z; in sumXYZ 1 2 3
    error: value is an integer while a set was expected, at (string):1:39

Changing the name of the `z` attribute to `zz` will not work:

    nix-repl> let sumXYZ = {x, y, z}: x + y + z; in sumXYZ { x = 1 ; y = 2; zz = 3; }
    error: ‘sumXYZ’ at (string):1:14 called without required argument ‘z’, at (string):1:39

Adding an extra attribute `w` will also not work:

    nix-repl> let sumXYZ = {x, y, z}: x + y + z; in sumXYZ { w = 0 ; x = 1 ; y = 2; z = 3; }
    error: ‘sumXYZ’ at (string):1:14 called with unexpected argument ‘w’, at (string):1:39

Sometimes we may not mind if the set has extra attributes, such as `w = 0;`. The `...` pattern allows for extra attributes:

    nix-repl> let sumXYZ = {x, y, z, ...}: x + y + z; in sumXYZ { w = 0 ; x = 1 ; y = 2; z = 3; }
    6

Other times we may want to make some attributes optional and provide
default values for missing attributes. We can do that by using `<name>
? <expr>` in the pattern to provide a default:

   nix-repl> let sumXYZ = {x, y ? 0 , z ? 0}: x + y + z; in sumXYZ { x = 1; }
   1

   nix-repl> let sumXYZ = {x, y ? 0 , z ? 0}: x + y + z; in sumXYZ { x = 1; y = 2; }
   3

We can also provide a name for the set pattern by using `@`:

   nix-repl> let sumXYZ = triple@{x, y, ...}: x + y + triple.z; in sumXYZ { z = 3; y = 2; x = 1 ; }
   6

functions, `let` and `set patterns`
-----------------------------------

Sometimes you may see a fancy looking expression like:

    {x, y, z}:
    let foo = "foo";
    bar = "bar";
    in { a = x; b = y; c = z; d = foo; e = bar; }

At first this may look like some other function form -- but it really just a combination of a function, a `set pattern` and a `let`-expression.

For `pattern: expr` we have:

    pattern = {x, y, z}
    expr    = let foo = "foo"; bar = "bar"; in { a = x; b = y; c = z; d = foo; e = bar; }

This can be especially confusing when you see something like:

    {x, y, z}: { a = x; b = y; c = z; }

Because in languages like C and Javascript the `{ }` are used to denotate the bounds of a function body. But, remember, in Nix that just means we are creating an attribute set.

inherit
=======

Based on our knowledge of `let` and `set` the following should not be surprising:

    nix-repl> let x = 1; y = 2; in { z = x; }
    { z = 1; }

We can even write this:

    nix-repl> let x = 1; y = 2; in { x = x; }
    { x = 1; }

Since `let` is not recursive, that is ok! The name `x` in the set will be assigned to the value of `x` from the let expression. Of course, if we had a recursive set, then we would get in trouble:

    nix-repl> let x = 1; y = 2; in rec { x = x; }
    { x = error: infinite recursion encountered, at (string):1:32

Instead of writing `x = x;` we can use the `inherit` keyword:

    nix-repl> let x = 1; y = 2; in rec { inherit x; inherit y; }
    { x = 1; y = 2; }

Or shorter:

    nix-repl> let x = 1; y = 2; in rec { inherit x y; }
    { x = 1; y = 2; }

We can also use inherit to inherit attributes from another set:

    nix-repl> let integers = { a = 1; b = 2; c = 3; } ; in { inherit (integers) a b; }
    { a = 1; b = 2; }

There is no wildcard `inherit`. If you do not specify any attributes to inherit, then you don't get anything:

    nix-repl> let integers = { a = 1; b = 2; c = 3; } ; in { inherit (integers); }
    { }


with-expressions
================

Imagine if we have the following:

    nix-repl> let integers = { a = 1; b = 2; c = 3; } ; in integers.a + integers.b + integers.c
    6

The repeated use of `integers.` can be rather verbose. A `with-expression`

    with e1; e2

makes life easy!

    nix-repl> let integers = { a = 1; b = 2; c = 3; } ; in with integers; a + b + c
    6

the `e1` must be an expression that evaluates to a set. `with` turns all the `attributes` in the set in variable bindings in `e2`.

`with` is commonly used with `import`:

    with (import ./foo.nix); ...

In this case, `foo.nix` must contain an expression that evaluates to a set.


assert
======

assert has the syntax:

    assert e1; e2

`e1` must be an expression that evaluates to a Boolean. If it is `true` then `e2` is evaluated. If it is false, then the evaluator aborts and prints a backtrace.



comments
========

Nix uses # for single line comments and /* ... */ for multiline comments.


Review!
=======

That is the essence of the Nix expression language! You might be wondering how this relates to building, configuring, and installing packages. If we can't even print a string, how can we possibly build packages!

Next time we will start looking at derivations and the Nix store -- the magic that makes things happen! This magic is built on top of Nix expressions -- so that's why we had to learn about the Nix expression language first.

Nothing beats hands on experience. In addition to playing around with `nix-repl` I highly recommend working your way through this tutorial:

https://nixcloud.io/tour/?id=1e

It covers the same information we just learned, and presents you with an series of simple challenges to solve. The whole thing is self-contained in the browser, so you don't even need Nix installed.

