Before we get into using the nix package management tools we are going to peek at how the magic under the hood works. There are two key parts -- the nix expression language, and the nix store.

The nix expression language is a pure, lazy, functional language for
describing packages, collections of packages, variations of packages,
etc. It is not intended to be a general-purpose programming language.

Alas, as bug #14 says, "Nix won't be complete until it has static
typing." nix is, for now anyway, a dynamically typed language.

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

We can start playing around with values using `nix-repl`. The `nix-repl` is not installed by default, but we can simply run,

    nix-shell -p nix-repl

$ nix-repl
Welcome to Nix version 1.10. Type :? for help.

nix-repl>

The `let`-expression binds a name to a value. That is just a fancy way
of saying that let allows us to name an expression. For example:

let x = 1 ;
    y = 2 ;
in x + y

nix-repl> let x = 1 ; y = 2 ; in x + y
3

As you can see, we can bind multiple names at once in a let expression. The bindings are only valid in the expression after the `in`.

The expression after the `in` is evaluated by first replacing the names with the values they are currently bound to.

so x + y because 1 + 2.

let expressions are pretty simple & predictable. But, to get a real
feel for let expressions, let's look at some of the ways they can be
used and how they behave.

A let expression is, as the name suggests, an expression. So it can be
used anywhere an expression is valid. This means, for example, that we
can nest let expressions:

let x = 1 ; in (let y = 2 ; in x + y)

Or we could write:

    nix-repl> let x = (let y = 1 ; in y) ; in x
    1

A value is only bound inside the `in` expression. That means we can write this:

nix-repl> let x = 1 ; in (let y = 2 ; in y) + x
3

But not this:

nix-repl> let x = 1 ; in (let y = 2 ; in y) + y
error: undefined variable ‘y’ at (string):1:37

A let expression can shadow names that are already bound:

let x = 1 ; in (let x = 2 ; in x)

At first glance, let expressions may look similar to variables found
in common imperative languages like C, javascript, php, etc.

Consider the following expression:

    nix-repl> let x = 1 ; in (let x = 2 ; in x) + x
    3

let bindings are lazy. If we write:

    nix-repl> let x = x ; in 1
    1

Everything is fine because we never evaluate x. But if we actually try
to evaluate x:

    nix-repl> let x = x; in x
    error: infinite recursion encountered, at (string):1:9

nix detects that we have infinite recursion.

let expressions are also recursive. This means that the values being bound can refer to each other. For example:

  nix-repl> let y = x; x = 1 ; in y
  1



let name1 = expr1;
    name2 = expr2;
    nameN = exprN;
in expr4


Integer:

 maximum:
  9223372036854775807
 toString

