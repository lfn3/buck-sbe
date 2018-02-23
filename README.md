# Using [buck](https://buckbuild.com/) and [Simple Binary Encoding](https://github.com/real-logic/simple-binary-encoding) together

This is something I cooked up in the process of figuring out how to glue together
[buck](https://buckbuild.com/) (Facebook's build tool), and [Simple Binary Encoding](https://github.com/real-logic/simple-binary-encoding) (SBE).

We use buck at my employer [LMAX](https://www.lmax.com/), and want to try and
use SBE in the future, so figuring out to to make SBE's code generation tool
work in buck is something I wanted to figure out how to do.
Some knowledge of how buck works is probably required for this to be useful.

### Running it

This assumes you have buck installed: https://buckbuild.com/setup/getting_started.html,
and on the path: `buck --version` in any random directory should return something.

```
$ buck run :result-bin
# A
# B
# C
```

This will run the SBE generator over [example-schema.xml](https://github.com/lfn3/buck-sbe/blob/master/example-schema.xml),
build all the generated files, then our `results-bin` jar, and run it.
The main method is in [Core](https://github.com/lfn3/buck-sbe/blob/master/src/main/java/net/lfn3/buck_sbe_tool/Core.java), 
and you can see the high quality code that gives you the output shown above.

The more useful part happens if you do `buck project --ide intellij`. 
The important part for us is that the java source files created by the 
[sbe-tool](https://github.com/real-logic/simple-binary-encoding/wiki/Sbe-Tool-Guide)
are available to intellij, so can use the normal navigation options in intellij to find them.

Also, you can't edit the Java files. This is also probably a good thing. Some way of pointing out
where exactly the xml file they're sourced from is would be nice, but I think (hope?) people will
be able to figure that out.

### How does it work?

Most of the work happens in the [generate-sbe-stubs](https://github.com/lfn3/buck-sbe/blob/master/BUCK#L39)
macro in the buckfile. This macro contains a bunch of genrules:

```python
genrule(
        name = name + '-sbe-tool-gen',
        cmd = 'java -jar -Dsbe.target.namespace=' + namespace + ' -Dsbe.output.dir=$OUT/../ $(location :sbe-tool-bin) $SRCS',
        srcs = srcs,
        out = namespace.split('.')[0]
    )
```

This one actually runs the SBE tool. There's a bit of trickery there involving the namespace variable - 
we use the leading chunk of the namespace (i.e. the `net` in `net.lfn3.buck_sbe_tool`) as the output directory,
so that when we later zip everything up for use as the 'source-jar' all the directories match up.
This is also why the output directory (`-Dsbe.output.dir=$OUT/../`) involves a step back up the path.



Some of this is a bit nasty - I have to zip the java files produced by the SBE
code generator in order to have 'results' build step pick them up, which is
unnecessary work. I can't find a way of telling buck to 'use all the java files
produced as a result of this build target', however.

I'm also unhappy with the copy step a little bit, but I don't have a good idea
of what a better approach would be.

I will probably wrap all this up in a single 'SBE' target at some point.
