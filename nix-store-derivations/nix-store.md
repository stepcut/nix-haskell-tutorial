The nix store & derivations
===========================

We started down this path with the idea of configuring, building, and
installing packages on our system in a pure, declarative way. But then
we went down the rabbit hole of learning the nix expression language,
which seems unrelated to the problem we are trying to solve.

So, let's review what we are actually trying to achieve. Our big two goals are:

 1. deterministic builds -- building the same thing twice should result in the exact same output
 
 2. once a package is installed, it won't break due to dependencies being upgraded

determinstic builds
===================

The essence of a 'pure' or 'determinstic' system is that a function
should always produce the same outputs when given the same inputs.

For a build system like Nix, the output is the files that are built --
the executables, the libraries, the config files, etc.

The inputs are anything that can affect the output. That includes the
obvious things like the source code for a package, what dependencies
it is built against, and the configuration settings. But it also
includes things like environment variables, the timestamps on files,
etc. To make a build deterministic -- every aspect of the build which could
change needs to be identified and locked down.

Obviously, nix can only do so much -- if your code uses Template
Haskell to randomly generate different output everytime it builds --
then stop doing that!

Rather than try to track a bunch of unimportant factors that don't
actually matter -- like unused environment variables, Nix starts with
a pristine build environment. Anything which should affect the build,
such as enviroment variables, the source code, the dependencies, etc,
must be specified as an input to the build.

The expression which describes all the inputs to the build action is
called a `derivation`. A `derivation` is just a special type of
attribute set. Before we dig into `derivations` we need to learn about
the nix store.

nix store
=========

Another promise of Nix is that once a package is installed, it will never break because you have upgraded a dependency. But, what if you want to install two different packages which depend on two different versions of the same library -- how does that work?

The answer lies in the nix store. Everything that nix builds is stored in `/nix/store`. Let's say we need to create a file 'greeting.txt' which contains a friendly greeting. We can run the following in nix-repl,

    nix-repl> builtins.toFile "greeting.txt" "hello, world."
    "/nix/store/ipanzazapikrzb82z0qzl6rh1vb22l6j-greeting.txt"

The first argument to `toFile` is the name of the file and the second argument is the text to write to the file. We see that Nix created the file for us in `/nix/store`.

    $ cat /nix/store/ipanzazapikrzb82z0qzl6rh1vb22l6j-greeting.txt
    hello, world.

If we run the same nix command again it returns the same path:

    nix-repl> builtins.toFile "greeting.txt" "hello, world."
    "/nix/store/ipanzazapikrzb82z0qzl6rh1vb22l6j-greeting.txt"

Let's change the period to an exclamation point and add a trailing newline:

    nix-repl> builtins.toFile "greeting.txt" "hello, world!\n"
    "/nix/store/lh23wjc4h9f9a4bsi3rckyra3xnbzfgm-greeting.txt"

    $ cat /nix/store/lh23wjc4h9f9a4bsi3rckyra3xnbzfgm-greeting.txt
    hello, world!

Now we get a new file with the new contents. If you are thinking that string of letters prefixing the file name is some sort of cryptographic hash of the file contents, you are right! In our Nix config we might write something like:

    nix-repl> let greetingFile = builtins.toFile "greeting.txt" "hello, world." ; in { greeting = greetingFile; }
    { greeting = "/nix/store/ipanzazapikrzb82z0qzl6rh1vb22l6j-greeting.txt"; }

Later we might update it to:

    nix-repl> let greetingFile = builtins.toFile "greeting.txt" "hello, world!\n" ; in { greeting = greetingFile; }
    { greeting = "/nix/store/lh23wjc4h9f9a4bsi3rckyra3xnbzfgm-greeting.txt"; }

So, now we are starting to get a picture of how things work in Nix. In the Nix expression we work with nice human readable names like "greeting.txt". But, under the hood, Nix calculates the hash of the file contents and uses that to make the file name unique based on its contents. If the file already exists in the nix store, then it can return the existing file, otherwise it will create the new file with the new name.

When we modify the contents of `greeting.txt` in the expression, we are not modifying the contents of the file on disk, but creating a new file. Both versions of `greeting.txt` can co-exist because they are store in different files on the disk.

derivations
===========

A derivation is just a (special) set which contains all the attributes of a build. To create a derivation we can use the builtin function `derivation`.

The `derivation` function takes a set as an argument. The set has many optional arguments, but the three required arguments are:

  name - the name of the package
  builder - name of the script that will build the package
  system - a string specifying the Nix platform identifier, e.g. x86_64-linux, x86_64-darwin, etc.

So, let's create a derivation:

let simpleDerivation = derivation { name = "foo" ; builder = "foo.sh"; system = "x86_64-linux"; } ; in simpleDerivation

    nix-repl> :t let simpleDerivation = derivation { name = "foo" ; builder = "foo.sh"; system = "x86_64-linux"; } ; in simpleDerivation
    a set

using `builtins.trace` we can see all the attributes of the set:

    nix-repl> let simpleDerivation = derivation { name = "foo" ; builder = "foo.sh"; system = "x86_64-linux"; } ; in builtins.trace simpleDerivation ""
    trace: { all = <CODE>; builder = "foo.sh"; drvAttrs = { builder = "foo.sh"; name = "foo"; system = "x86_64-linux"; }; drvPath = <CODE>; name = "foo"; out = { all = <CODE>; builder = "foo.sh"; drvAttrs = { builder = "foo.sh"; name = "foo"; system = "x86_64-linux"; }; drvPath = <CODE>; name = "foo"; out = <CYCLE>; outPath = <CODE>; outputName = "out"; system = "x86_64-linux"; type = "derivation"; }; outPath = <CODE>; outputName = "out"; system = "x86_64-linux"; type = "derivation"; }
""

The `derivation` automatically serializes the inputs and stores them in the nix store in a `.drv` file:

    nix-repl> let simpleDerivation = derivation { name = "foo" ; builder = "foo.sh"; system = "x86_64-linux"; } ; in simpleDerivation
    «derivation /nix/store/gahaap071chaynabdz6a5incfwsv11pv-foo.drv»

Remember that a derivation only specifies the inputs to a build -- we have not actually built anything yet. Of course, if we did try to build this, it would fail because we have not created `foo.sh` yet. So, let's fix that!



