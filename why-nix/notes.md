Hello!

This video series will teach you how to develop, test, and deploy
Haskell web services to the cloud using Nix.

Now you may be wondering -- what is nix and why would I want to use
it?

The term Nix is a bit overloaded, but in the most general sense, it is
a package management system built around its own purely functional
expression language which is simply called the Nix expression
language.

The nix package system is self-contained which means you can use it on
a system that is already using apt-get, yum, etc. In fact, it even
runs on OS X. Sadly, Windows is not supported. Mostly because nobody
is working on it.

There is a full ecosystem built up around the core nix language and
tools.

The most essential component is nixpkgs, the package database. Nix has
proven popular in the Haskell community, and so Haskell is well
supported in nixpkgs. In fact, nearly 100% of the packages on Hackage
are available in nixpkgs by default. nixpkgs also supports multiple
different versions of GHC and even GHCJS.

The nixpkg collection is built using Hydra. This allows users to
download prebuilt binary packages. Developers can also use it to
perform continuous integration testing.

NixOS is a Linux distribution built from the ground up around Nix.

NixOS servers can be deployed using the nixops tool. It supports cloud
services including EC2 and GCE, or NixOS servers run on the hosting
company of your choice, or even deployment to VirtualBox instances.

So we see that Nix has all the tools we need to develop, test, and
deploy web applications. But what makes these tools better than
anything else? In a nutshell: reproducability.

The secret sauce is nix's purely functional approach to package and
system management. Instead of saying that Nix is purely functional, it
might be better to say that it is declarative. You write a declaration
of what the machine should be like. This declaration is feed into the
nix package tools which then produce the desired machine state as
their output.

The traditional approach to package and system management is based on
mutation. You start with a fresh installation, and then you install a
bunch of packages, and edit a bunch of config files by hand, and
hopefully end up with a working system.

While this model is simple to understand, it is difficult to manage in
the long run. It can be difficult to recreate the working
configuration because you need to remember the correct set of
steps. Before using Nix, my company had a policy that you had to make
a wiki page describing the set of steps you used to install and
configure a service on the server in the hope that someone would be
able to reproduce the setup later.

In Nix, we instead have a complete specification of the machine
written in the Nix expression language. This expression can be
evaluated to produce the intended server state. This includes not only
which packages to be installed, but what versions of libraries those
packages should be built against, what the contents of the config
files should be, what user accounts should exist on the system, etc.

Because the expression captures every critical piece of information
that goes into building the server, it is possible to wipe out the
server and rebuild it from source and be sure that all the files and
binaries that are recreated have the same sha256 checksums as before.

Because the expressions are just text files, the server configuration
can be stored in github.

Imagine getting a bug report that says something mysterious is broken
on your website, but it used to work 6 months ago. You could actually
check out the configuration from 6 months ago and recreate the exact
environment on a test server.

Another issue with the traditional mutation based system management is
the lack of atomic upgrades. When a new version of some application
comes out, you may decide to upgrade it on the server. What has often
happened to me in the past is that the newer version has updated
dependencies, so it brings in a whole bunch of libraries and other
packages. But, halfway through, some pacakge has an installation error
and the installation aborts. Unfortunately, things are now a giant
mess. Now the old version doesn't work because some of the old
dependencies have been upgraded, and the new version doesn't work
because not all the new dependencies installed. Trying to get the
system back to the known working state can be a nightmare --
especially with no record of what exactly that working state was.

In Nix, upgrades are atomic -- the upgrade either entirely suceeds or
no changes are made.

Another problem with traditional systems is lack of isolation. For
example, other times I have upgraded a pacakge, only to find out that
it pulled in some new library that breaks a different application.

Nix's purely functional model dictates that once a library,
executable, or configuration file is built and installed, it is never
modified. In fact, on NixOS, the package store is mounted
read-only and even the root user can not trivially modify the files.

If multiple versions of something built, they are stored in
unique output directories. This means that upgrading one package will
not accidentally break another package. Both can coexist at the same
time even though they are using different versions of the same
library.

Another missing feature of traditional systems is the ability to
rollback to a previous working configuration. Often times, I have
upgraded a server only to realize the upgrade contains some horrendous
bug. Because Nix performs upgrades by installing new files rather than
modifying the old files, it is trivial to simply rollback to a
previous configuration. In fact, by default all previous
configurations are preserved and it is possible to switch freely
between them.

Now, you might be getting concerned about running out of diskspace if
nothing is ever deleted. Fortunately, nix does have a garbage
collector. We will cover that later.

Nix is great for developers because it allows you to create sandboxed
developement environments. While cabal sandbox allows you to sandbox
things at the Haskell library level, nix extends this functionality to
all packages. For example, you can easily be using GHC 7.8 in one
shell and GHC 7.10 in another.

Using the complete nix stack brings other advantages as well. Because
nix, hydra, nixops, etc are all based around the Nix expression
language, they can also share common configuration files. For example,
if you need to apply a patch to some library so it will build, you can
specify this patch in a shared configuration file. That information
will then be used to build for your local system, or in hydra, or for
deploying to the server. You won't run into those annoying cases where
'it works for me' because you forgot that you had some patch applied
locally.

Being able to store the complete configuration including the entire
nixpkg repository in git is a huge benefit when working with a
team. It allows you to have very precious control over when the
package database is upgraded, makes it easy to pin packages, makes it
easy to apply patches to libraries, and to do this in a way that keeps
all developer's machines insync.

Because Nix tracks all the inputs required to build a package, it can
often avoid building a package at all. If the package has already been
built with the exact same configuration, it can be downloaded from a
binary store.

Hydra simulatenously functions as a build farm and continuous
integration server. If you are working on a project with a large
number of dependencies, this can be a huge time saver. A teammate can
commit a patch to some low-level library, and hydra will rebuild that
library and everything that depends on it, and if that succeeds, it
will automatically be available via the binary cache. If you then
update your project directory, nix will automatically detect which
packages need to be rebuilt, and if they match a configuration that
was already built it can simply download the prebuilt binaries. If
there is only a partial match it will automatically download some
packages and build others from source.

Nix is also portable, making it easy to develop on system that is
different from your deployment machine. For example, I deploy on Linux
servers, but I do all my development work under OS X. Because nix
behaves the same, it is incredibly transparent. I can use the same nix
configuration files on my OS X machine as I do on the Linux boxes.

NixOps builds on top of Nix and NixOS to deploy machines to the cloud.

NixOps is built around two core pieces, a logical machine
specification and an environment specification.

With NixOps you use the nix expression to create a declariation of a
logical machine. This declaration includes all the information about
what packages should be installed, what user accounts should be
created, what services should be run, how they should be configured,
etc.

The environment specification details a location where a logical
machine can be deployed. It can be real physical hardware, a
virtualbox instance, an EC2 or GCE instance, etc. This specification
includes information about the amount of RAM available, how to access
the machine, etc. In the case of EC2 and VirtualBox, nixops can even
automatically provision the machine if it does not already exist.

The same logical machine can be deployed to multiple
environments. This makes it easy to test a new configuration by first
deploying it to a VirtualBox instance. It also makes it easy to run a
load-balanced cluster where a similar configuration needs to be
deployed to multiple machines, but with minor tweaks for each server.

I've only touched on a few of the advantages Nix and a small number of
use cases. But, hopefully you are now excited about its potential to
make your life easier!

Fortunately, it is easy to try out Nix! No commitment required! As I
mentioned earlier, Nix can be installed along existing package
managers. Nix stores all its files under /nix/store -- so it won't
interfer with any other files already on your system. Using Nix only
requires a single line to be added to your .bashrc file. If you
comment out that line, it will be as if you never installed Nix. If
you decide Nix is not right for you can just comment out that line and
remove /nix/store.

So, let's get started! Installing nix is very easy. All you need to do is run this one line:

curl https://nixos.org/nix/install | sh

