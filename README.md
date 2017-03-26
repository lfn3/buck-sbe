# Using [buck](https://buckbuild.com/) and [Simple Binary Encoding](https://github.com/real-logic/simple-binary-encoding) together

This is something I cooked up in the process of figuring out how to glue together
[buck](https://buckbuild.com/) (Facebook's build tool), and [Simple Binary Encoding](https://github.com/real-logic/simple-binary-encoding) (SBE).

We use buck at my employer [LMAX](https://www.lmax.com/), and want to try and
use SBE in the future, so figuring out to to make SBE's code generation tool
work in buck is something I wanted to figure out how to do.

### Running it

This assumes you have buck installed: https://buckbuild.com/setup/getting_started.html
Some knowledge of how buck works is probably required for this to be useful.

```
$ buck run :result bin
# A
# B
# C
```

If you want to edit the code: `buck build :copy-sbe-genned` so you can actually
see the SBE generated files.

### Commentary

Most of the work happens in the two genrules in the buckfile. The first one is
responsible for actually running the SBE code generator, and the second one
copies the resulting files into the src/generated directory so they're visible
to intellij.

Some of this is a bit nasty - I have to zip the java files produced by the SBE
code generator in order to have 'results' build step pick them up, which is
unnecessary work. I can't find a way of telling buck to 'use all the java files
produced as a result of this build target', however.

I'm also unhappy with the copy step a little bit, but I don't have a good idea
of what a better approach would be.

I will probably wrap all this up in a single 'SBE' target at some point.
