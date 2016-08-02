Collection Types
================

Nix has two collection types list and set.

list
====

The syntax for a list in Nix is a whitespace separated list of values surrounded by square brackets:

    nix-repl> [ 1 2 3 4 ]
    [ 1 2 3 4 ]

    nix-repl> :t [ 1 2 3 4]
    a list

Lists are concatenated using `++`:

    nix-repl> [1 2 3] ++ [4 5 6]
    [ 1 2 3 4 5 6 ]

Nix lists are heterogeneous. That means each element of the list can have a different type:

    nix-repl> [ 1 "hello" /foo true ]
    [ 1 "hello" /foo true ]


    nix-repl> :t [ 1 "hello" /foo true ]
    a list

Nix lists have lazy values but a strict length. That means the elements are not evaluated until you force them. So, for example, we can use builtins.elemAt to retrieve an element from this list:


    nix-repl> builtins.elemAt [ 1 (throw "die") ] 0
    1

Even though the second element would throw an exception if evaluated. This sort of laziness is useful in Nix because many values represent packages that need to be built. It would be a waste of time if we built all the packages in a list and then only used one element from the list.

Nix lists are strict in length, however, which means we can not have infinite lists.

The nix standard library includes many of the standard list operations include folds, maps, and filters.

set
===

The Nix set type is an attribute set -- similar to what other languages call a hashmap, dictionary, extensible record, etc. Sets contain name/value pairs aka, attributes. Each name can appear only once in a set, and the order of attributes is irrelevant. The set type is heart and soul of most Nix expressions.

creating a set
--------------

An example of a set is as follows:

    nix-repl> { x = 123; str = "hello, world!";}
    { foo = "foo"; foo-2 = true; hello world = "!"; str = "hello, world!"; x = 123; }

    nix-repl> :t { x = 123; str = "hello, world!"; }
    a set

A set is enclosed in curly braces, and attributes are defined as `<name> = <expr>;`. The empty set is just `{}`:

    nix-repl> {}
    { }

The <name> is a string value. In most cases you do not need to use double quotes around the string. In this example:

    nix-repl> { x = 123; "x" = 123; }
    error: attribute ‘x’ at (string):1:12 already defined at (string):1:3

We see that `x` and `"x"` are both the same name.

In some cases, such as this one, the double quotes are required:

    nix-repl> { 0 = 123; }
    error: syntax error, unexpected INT, at (string):1:3

    nix-repl> { "0" = 123; }
    { 0 = 123; }

Using anti-quotation we could create a name that depends on value of a Boolean:

    nix-repl> { ${ if true then "foo" else "bar"} = 0; }
    { foo = 0; }

    nix-repl> { ${ if false then "foo" else "bar"} = 0; }
    { bar = 0; }

It is also possible to use antiquotation inside a string:

    nix-repl> { "foo-${toString(1+1)}" = 123; }
    { foo-2 = 123; }

We can also use antiquotation to create an optional attribute. If the antiquotation evalutes to `null` then the attribute will not be added:

    nix-repl> { ${if true then "foo" else null} = 123; }
    { foo = 123; }

    nix-repl> { ${if false then "foo" else null} = 123; }
    { }

selecting from a set
--------------------

To select an attribute from a set, you use the dot operator:

    nix-repl> { foo-1 = "abc"; foo-2 = "def"; }.foo-1
    "abc"

    nix-repl> { foo-1 = "abc"; foo-2 = "def"; }.foo-2
    "def"

The same rules regarding names apply to selecting as they did in creating. Instead of `.foo-2` we could have written:

    nix-repl> { foo-1 = "abc"; foo-2 = "def"; }."foo-${toString(1+1)}"
    "def"

`e ? attrpath` is used to check if a set contains an attribute:

    nix-repl>  {a = 1; b = 2; c = 3;} ? a
    true
    
    nix-repl>  {a = 1; b = 2; c = 3;} ? d
    false

recursive sets
--------------

A set is not recursive by default -- meaning that attributes can not refer to other attributes in the set. If we try we get an error like this:

    nix-repl> { foo-1 = foo-2; foo-2 = "def"; }
    error: undefined variable ‘foo-2’ at (string):1:11

To make a set recursive you use the `rec` keyword:

    nix-repl> rec { foo-1 = foo-2; foo-2 = "def"; }
    { foo-1 = "def"; foo-2 = "def"; }

merging and updating sets
-------------------------

Sets can be merged using the // operator:

    nix-repl> {a = 1; b = 2;} // {b = 3; c = 4; }
    { a = 1; b = 3; c = 4; }

In the event that a name occurs twice the value from the set on the right will be selected.

There is no way to mutate a set -- Nix is pure -- aka immutable:

    nix-repl> {a = 1; b = 2;}.a=2
    error: syntax error, unexpected '=', expecting $end, at (string):1:18

But we can use // to create a *new* set from an existing set.

    nix-repl> {a = 1; b = 2;} // { a=2; }
    { a = 2; b = 2; }

and more!
---------

The prelude includes functions like `builtins.getAttr`, `builtins.hasAttr`, `builtins.intersectAttrs`, `removeAttrs`, and `listToAttrs`.

