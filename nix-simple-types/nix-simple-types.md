Simple Types
=============

nix has 5 primitive types.

 - String
 - Path
 - Integer
 - Boolean
 - null

An easy way to experiment with expressions is to use the `nix-repl` program. Simple run:

    $ nix-shell -p nix-repl

And you will be dropped into a shell where the `nix-repl` program has
been built and is in the $PATH. Then run nix-repl at the command line
to start the repl:

    [nix-shell:~]$ nix-repl

    Welcome to Nix version 1.11.2. Type :? for help.
    
    nix-repl> 

Strings
=======

Double Quoted
-------------

There are three ways we can create a string literal in Nix. The first is to surround the text in double quotes:

    nix-repl> "hello, world!"
    "hello, world!"

We can use the nix-repl command :t to see the type of an expression

    nix-repl> :t "hello, world!"
    a string

The `+` operator is used to concatenate two strings:

    nix-repl> "hello, " + "world!"
    "hello, world!"


As you may expect from other languages, " and \ are special characters that must be escaped using a \. The character sequence `${` is also reserved and must be escaped using \. Like other languages, you can use \n for newline, \r for carriage return, and \t for tab. In this example we escape the double quotes and insert a newline in the middle:

    nix-repl> "hello,\n\"world\""
    "hello,\n\"world\""

However, this example is not very illuminating. Since nix-repl shows us the escaped form of the string, it is hard to tell that the \ sequences have any special meaning at all. What we really want to do is print the string and see what it looks like printed. But, since nix is pure -- it does not have a print function. However, like Haskell, Nix does have a trace function:

    builtins.trace e1 e2

`builtins.trace` will evaluate its first argument and print it on stderr and will return the value of the second expression. `builtins.trace` is intended to be used for debugging purposes only. Using `builtins.trace` we can now see what \ really does:

    nix-repl> builtins.trace "hello,\n\"world\"" ""
    trace: hello,
    "world"
    ""

In a string, the syntax ${...} is used for antiquotation. antiquotation is a really fancy word for string substitution. The `...` is any expression which evaluations to a string, path, or derivation. For example we could write:

    nix-repl> "hello, ${"world" + "!"}"
    "hello, world!"

In practice, it would be more common to see antiquotation used with a variable like this:

    nix-repl> let object = "world"; in "hello, ${object}"
    "hello, world"

We will cover the `let expression` later.


indented strings
----------------

A second type of string literal in Nix is `indented strings`. An `indented string` is surrounded by *double single-quotes*. For example:

    nix-repl> ''hello, world!''
    "hello, world!"

In `indented strings` only `''` and `${` have special meaning. So we can use double quotes and \ with out having to escape them:

    nix-repl> builtins.trace ''hello \n "world"'' ""
    trace: hello \n "world"
    ""

If we wanted the \n to actually insert a newline then we would need to write it is `''\n`

    nix-repl> builtins.trace ''hello ''\n "world"'' ""
    trace: hello 
    "world"
    ""

To escape ${ we write ''${:

    nix-repl> builtins.trace ''hello ''${ "world"'' ""
    trace: hello ${ "world"
    ""

To escape '' we write '''

    nix-repl> builtins.trace ''hello ''' "world"'' ""
    trace: hello '' "world"
    ""

Thus far, indented strings don't seem worthwhile, and the name doesn't make sense. The power of `indented strings` is that they allow for multi-line string literals. Unfortunately, `nix-repl` does not support the multiline feature of `indented strings`, so you will not be able to test this feature interactively.

The leading indentation is automated striped from multiline strings, which allows you to keep things neatly indented in your code. It strips from each line a number of spaces equal to the minimal indentation of the string as a whole. So if you wrote:

    ''
       This is line one.
      This is line two.
        This is line three.
    ''

Where the first line is indented 3 spaces, the second is indented 2 spaces, and the third is indented 4 spaces, nix would strip two spaces from the start of each line resulting in:

    " This is line one.\nThis is line two.\n  This is line three.\"

Since the remainder of the line after the openning `''` is exclusively whitespace, it is ignored and not included in the string literal.

The combination of multiline support, identation stripping, uncommon reserved character sequences, and escaping rules make `indented strings` ideal for embedding shell scripts and configuration files in your nix expressions. For example, consider the following snippet:

    stdenv.mkDerivation {
      ...
      postInstall =
        ''
          mkdir $out/bin $out/etc
          cp foo $out/bin
          echo "Hello World" > $out/etc/foo.conf
          ${if enableBar then "cp bar $out/bin" else ""}
        '';
      ...
    }

This snippet creates a postInstall shell script. The double quotes in `echo "Hello World"` do not require any special treatment. And, in the following line antiquotation is used to do some metaprogramming.

URI Syntax
----------

The third way of creating a string literal is to write a URI without double quotes:

    nix-repl> http://www.example.org/
    "http://www.example.org/"

This is just a bit of syntactic sugar -- the value is still just a string -- there is no special URI type.

    nix-repl> :t http://www.example.org/
    a string

It is the same as if we had written the URI with double quotes:

    nix-repl> "http://www.example.org/"
    "http://www.example.org/"

    nix-repl> :t "http://www.example.org/"
    a string

There are only a few string functions built into nix. We already saw + for concatenation.

`builtins.replaceStrings from to s` is used for string replace.

    nix-repl> builtins.replaceStrings ["oo" "a"] ["a" "i"] "foobar"
    "fabir"

`builtins.stringLength e` returns the length of a string:

    nix-repl> builtins.stringLength "hello, world!"
    13

`builtins.substring start len s` returns a substring:

    nix-repl> builtins.substring 7 5 "hello, world!"
    "world"

`toString` will attempt to convert a value to a string:

    nix-repl> toString 10
    "10"

Some values, like sets, can not be coerced.

Path
====

A path value represents a path on the local filesystem. A path literal must contain a /.

    nix-repl> /foo
    /foo

    nix-repl> foo
    error: undefined variable ‘foo’ at (string):1:1

In nix, path and string are different types:

    nix-repl> :t /foo
    a path

    nix-repl> :t "/foo"
    a string

A path can be converted to a string by using `toString`:

    nix-repl> toString /foo
    "/foo"

    nix-repl> :t toString /foo
    a string

When a path is used with antiquotation, it is not merely converted to a string:

    nix-repl> "${/etc}"
    "/nix/store/p762cs7s4bxsxmh9lpdxydrpjfj99q06-etc"

We will revist this unexpected output when we learn about the nix store.

A relative path will be automatically converted to a absolute path at parse time, relative to the directory of the Nix expression that contained it. For example, if the file: `/foo/bar/default.nix` refers to `../baz/extra.nix`, the expression would evaluate to the absolute path `/foo/baz/extra.nix`.

If we use nix-repl:

    nix-repl> ./hello.txt
    /Users/stepcut/Documents/nix/nix-simple-types/hello.txt

We can see that the path is made absolute relative to the directory we were in when we started `nix-repl`.

Integer
========

The only numeric type that nix support is integer. On a 64 bit machine, the integer type seems to have a range of -9223372036854775807 to 9223372036854775807. On a 32-bit machine, it likely has a range of -2147483647 to 2147483647.

The only way to express an integer is to write it out as a second of digits:

    nix-repl> 1234
    1234

    nix-repl> -1234
    -1234

There is support for the four basic integer operators, +, -, /, *:

    nix-repl> 1 + 1
    2

    nix-repl> 3 - 4
    -1

    nix-repl> 3 * 3
    9

    nix-repl> 4 / 3
    1

There is also support for the comparison operators, <, >, <=, >=:

    nix-repl> 3 < 4
    true

    nix-repl> 3 >= 4
    false

There does not appear to be support for modulo or remainders.

Boolean
=======

The boolean type has two values, true and false. The boolean literals are always written lower case:

    nix-repl> true
    true

    nix-repl> false
    false

    nix-repl> :t true
    a boolean

    nix-repl> :t false
    a boolean

The basic equality and boolean operators you would expect are available:

    nix-repl> true==true
    true

    nix-repl> true != false
    true

    nix-repl> true && true
    true

    nix-repl> true || false
    true

    nix-repl> !true
    false

There is also a logical implication operator `->`

    nix-repl> false -> false
    true

It is equivalent to `!e1 || e2`

There is no XOR operator.

null
====

The final simple value that nix supports is null. It has the type null and the only value is null:

    nix-repl> null
    null

    nix-repl> :t null
    null

The == and != support the null type:

    nix-repl> null == null
    true
    
    nix-repl> 0 != null
    true

Note that the empty string is *not* null.

    nix-repl> "" == null
    false



