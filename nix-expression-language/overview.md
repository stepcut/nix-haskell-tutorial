At the core of anything nix does there lies a nix
expression. So understanding the nix expression language is essential
if we want to build our development process on top of nix.

The nix expression language is not a general-purpose programming
language. It lacks many features you'd expect to find in a typical
programming language such as floating point numbers. It is a language
designed specifically for describing packages, collections of
packages, variations of packages, system configuration, and
deployments.

Let's start with an overview of what the language does provide. We'll go into more detail later.

Nix Evaluation Model
--------------------

First of all, Nix is declarative.

 - not bash -- a step-by-step set of instructions with control flow

 - we describe attributes about the package -- where to find the
   source, what the sha256 of the source should be, what other
   packages it depends on, whether tests should be run or not, and
   things like that.

Nix is functional.

 - first class functions - functions are normal values that can be passed around and manipulated
 - functions can return functions
 - partial application

Nix is pure.

 - for the same inputs, you get the same outputs
 - any value a function depends on must be supplied as an arguement to the function -- if it needs the contents of a file, then that file must be supplied as an input
 - means builds are reproducable

Nix is lazy.

 - arguments/values are not evaluated unless required.

 - do not need to calculate the value of all the packages in the nix
   package collection -- only the packages we depend on to evaluate
   the expression

Nix Type System
---------------

Nix is dynamically typed.

 - not as bad as it seems

 - no automatic type coersion
   - 1 + '1' is an error

 - errors generally result in build failures, not a successful build with hidden bugs

 - does lack transparency and introspection -- hard to figure out what attributes exist

 - Alas, as bug #14 says, "Nix won't be complete until it has static typing". Possibly mu types with row polymorphism

Nix has 5 simple types:

 - String
 - Path
 - Integer
 - Boolean
 - null

Nix has two collection types:

 - lists
 - sets
   - attribute sets -- name/value pairs

Nix does not have support for algebraic types, type classes, an object
system, or any other advanced type system features.

Though as the zen koan says, "Closures are a poor mans object system."

Language Constructs
-------------------

Nix has also a number of built-in language constructs such as:

 - let expressions
 - conditionals
 - assertions
 - comments
 - other nix specific constructs

Nix Syntax
----------

Nix is not whitespace sensitive.

 - uses {} and ; instead of whitespace layout


nix standard library
--------------------

And, lastly, Nix has a collection of built-in functions and operators.

In the next segment we will examine the simple types.


#######################################



https://nixcloud.io/tour/?id=1e





The nix expression language is a pure, lazy, functional language for
describing packages, collections of packages, variations of packages,
etc. It is not intended to be a general-purpose programming language.


Nix is pure. Meaning that functions do not have side-effects. It is
functional, meaning that functions can be pass to other functions or
returned from functions just like any other value. Nix is also lazy,
which means that values are not evaluated until they are actually
needed.

nix is not whitespace sensitive, except in the case of multiline
strings. Instead it uses curly braces and semicolons.


The nix language has X simple values:

  String
  Integer
  Path
  Boolean,
  null.

It also has two collection types:

  List
  attribute set

There are no mechanims for creating your own datatypes.


Before we get into using the nix package management tools we are going
to peek at how the magic under the hood works. There are two key parts
-- the nix expression language, and the nix store.
