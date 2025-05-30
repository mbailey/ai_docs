---
created: 2025-02-28T13:49:59 (UTC +11:00)
tags: []
source: https://docs.python.org/3/library/optparse.html#choosing-an-argument-parser
author: 
---

# optparse — Parser for command line options — Python 3.13.2 documentation

> ## Excerpt
> Source code: Lib/optparse.py Choosing an argument parsing library: The standard library includes three argument parsing libraries: getopt: a module that closely mirrors the procedural C getopt API....

---
**Source code:** [Lib/optparse.py](https://github.com/python/cpython/tree/3.13/Lib/optparse.py)

___

## Choosing an argument parsing library

The standard library includes three argument parsing libraries:

-   [`getopt`](https://docs.python.org/3/library/getopt.html#module-getopt "getopt: Portable parser for command line options; support both short and long option names."): a module that closely mirrors the procedural C `getopt` API. Included in the standard library since before the initial Python 1.0 release.
    
-   [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library."): a declarative replacement for `getopt` that provides equivalent functionality without requiring each application to implement its own procedural option parsing logic. Included in the standard library since the Python 2.3 release.
    
-   [`argparse`](https://docs.python.org/3/library/argparse.html#module-argparse "argparse: Command-line option and argument parsing library."): a more opinionated alternative to `optparse` that provides more functionality by default, at the expense of reduced application flexibility in controlling exactly how arguments are processed. Included in the standard library since the Python 2.7 and Python 3.2 releases.
    

In the absence of more specific argument parsing design constraints, [`argparse`](https://docs.python.org/3/library/argparse.html#module-argparse "argparse: Command-line option and argument parsing library.") is the recommended choice for implementing command line applications, as it offers the highest level of baseline functionality with the least application level code.

[`getopt`](https://docs.python.org/3/library/getopt.html#module-getopt "getopt: Portable parser for command line options; support both short and long option names.") is retained almost entirely for backwards compatibility reasons. However, it also serves a niche use case as a tool for prototyping and testing command line argument handling in `getopt`\-based C applications.

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") should be considered as an alternative to [`argparse`](https://docs.python.org/3/library/argparse.html#module-argparse "argparse: Command-line option and argument parsing library.") in the following cases:

-   an application is already using [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") and doesn’t want to risk the subtle behavioural changes that may arise when migrating to [`argparse`](https://docs.python.org/3/library/argparse.html#module-argparse "argparse: Command-line option and argument parsing library.")
    
-   the application requires additional control over the way options and positional parameters are interleaved on the command line (including the ability to disable the interleaving feature completely)
    
-   the application requires additional control over the incremental parsing of command line elements (while `argparse` does support this, the exact way it works in practice is undesirable for some use cases)
    
-   the application requires additional control over the handling of options which accept parameter values that may start with `-` (such as delegated options to be passed to invoked subprocesses)
    
-   the application requires some other command line parameter processing behavior which `argparse` does not support, but which can be implemented in terms of the lower level interface offered by `optparse`
    

These considerations also mean that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") is likely to provide a better foundation for library authors writing third party command line argument processing libraries.

As a concrete example, consider the following two command line argument parsing configurations, the first using `optparse`, and the second using `argparse`:

```
<span></span><span>import</span><span> </span><span>optparse</span>

<span>if</span> <span>__name__</span> <span>==</span> <span>'__main__'</span><span>:</span>
    <span>parser</span> <span>=</span> <span>optparse</span><span>.</span><span>OptionParser</span><span>()</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>'-o'</span><span>,</span> <span>'--output'</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>'-v'</span><span>,</span> <span>dest</span><span>=</span><span>'verbose'</span><span>,</span> <span>action</span><span>=</span><span>'store_true'</span><span>)</span>
    <span>opts</span><span>,</span> <span>args</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>()</span>
    <span>process</span><span>(</span><span>args</span><span>,</span> <span>output</span><span>=</span><span>opts</span><span>.</span><span>output</span><span>,</span> <span>verbose</span><span>=</span><span>opts</span><span>.</span><span>verbose</span><span>)</span>
```

```
<span></span><span>import</span><span> </span><span>argparse</span>

<span>if</span> <span>__name__</span> <span>==</span> <span>'__main__'</span><span>:</span>
    <span>parser</span> <span>=</span> <span>argparse</span><span>.</span><span>ArgumentParser</span><span>()</span>
    <span>parser</span><span>.</span><span>add_argument</span><span>(</span><span>'-o'</span><span>,</span> <span>'--output'</span><span>)</span>
    <span>parser</span><span>.</span><span>add_argument</span><span>(</span><span>'-v'</span><span>,</span> <span>dest</span><span>=</span><span>'verbose'</span><span>,</span> <span>action</span><span>=</span><span>'store_true'</span><span>)</span>
    <span>parser</span><span>.</span><span>add_argument</span><span>(</span><span>'rest'</span><span>,</span> <span>nargs</span><span>=</span><span>'*'</span><span>)</span>
    <span>args</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>()</span>
    <span>process</span><span>(</span><span>args</span><span>.</span><span>rest</span><span>,</span> <span>output</span><span>=</span><span>args</span><span>.</span><span>output</span><span>,</span> <span>verbose</span><span>=</span><span>args</span><span>.</span><span>verbose</span><span>)</span>
```

The most obvious difference is that in the `optparse` version, the non-option arguments are processed separately by the application after the option processing is complete. In the `argparse` version, positional arguments are declared and processed in the same way as the named options.

However, the `argparse` version will also handle some parameter combination differently from the way the `optparse` version would handle them. For example (amongst other differences):

-   supplying `-o -v` gives `output="-v"` and `verbose=False` when using `optparse`, but a usage error with `argparse` (complaining that no value has been supplied for `-o/--output`, since `-v` is interpreted as meaning the verbosity flag)
    
-   similarly, supplying `-o --` gives `output="--"` and `args=()` when using `optparse`, but a usage error with `argparse` (also complaining that no value has been supplied for `-o/--output`, since `--` is interpreted as terminating the option processing and treating all remaining values as positional arguments)
    
-   supplying `-o=foo` gives `output="=foo"` when using `optparse`, but gives `output="foo"` with `argparse` (since `=` is special cased as an alternative separator for option parameter values)
    

Whether these differing behaviors in the `argparse` version are considered desirable or a problem will depend on the specific command line application use case.

See also

[click](https://pypi.org/project/click/) is a third party argument processing library (originally based on `optparse`), which allows command line applications to be developed as a set of decorated command implementation functions.

Other third party libraries, such as [typer](https://pypi.org/project/typer/) or [msgspec-click](https://pypi.org/project/msgspec-click/), allow command line interfaces to be specified in ways that more effectively integrate with static checking of Python type annotations.

## Introduction

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") is a more convenient, flexible, and powerful library for parsing command-line options than the minimalist [`getopt`](https://docs.python.org/3/library/getopt.html#module-getopt "getopt: Portable parser for command line options; support both short and long option names.") module. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") uses a more declarative style of command-line parsing: you create an instance of [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser"), populate it with options, and parse the command line. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") allows users to specify options in the conventional GNU/POSIX syntax, and additionally generates usage and help messages for you.

Here’s an example of using [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") in a simple script:

```
<span></span><span>from</span><span> </span><span>optparse</span><span> </span><span>import</span> <span>OptionParser</span>
<span>...</span>
<span>parser</span> <span>=</span> <span>OptionParser</span><span>()</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--file"</span><span>,</span> <span>dest</span><span>=</span><span>"filename"</span><span>,</span>
                  <span>help</span><span>=</span><span>"write report to FILE"</span><span>,</span> <span>metavar</span><span>=</span><span>"FILE"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>"--quiet"</span><span>,</span>
                  <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span> <span>default</span><span>=</span><span>True</span><span>,</span>
                  <span>help</span><span>=</span><span>"don't print status messages to stdout"</span><span>)</span>

<span>(</span><span>options</span><span>,</span> <span>args</span><span>)</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>()</span>
```

With these few lines of code, users of your script can now do the “usual thing” on the command-line, for example:

```
<span></span><span>&lt;</span><span>yourscript</span><span>&gt;</span> <span>--</span><span>file</span><span>=</span><span>outfile</span> <span>-</span><span>q</span>
```

As it parses the command line, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") sets attributes of the `options` object returned by [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args") based on user-supplied command-line values. When [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args") returns from parsing this command line, `options.filename` will be `"outfile"` and `options.verbose` will be `False`. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") supports both long and short options, allows short options to be merged together, and allows options to be associated with their arguments in a variety of ways. Thus, the following command lines are all equivalent to the above example:

```
<span></span><span>&lt;</span><span>yourscript</span><span>&gt;</span> <span>-</span><span>f</span> <span>outfile</span> <span>--</span><span>quiet</span>
<span>&lt;</span><span>yourscript</span><span>&gt;</span> <span>--</span><span>quiet</span> <span>--</span><span>file</span> <span>outfile</span>
<span>&lt;</span><span>yourscript</span><span>&gt;</span> <span>-</span><span>q</span> <span>-</span><span>foutfile</span>
<span>&lt;</span><span>yourscript</span><span>&gt;</span> <span>-</span><span>qfoutfile</span>
```

Additionally, users can run one of the following

```
<span></span><span>&lt;</span><span>yourscript</span><span>&gt;</span> <span>-</span><span>h</span>
<span>&lt;</span><span>yourscript</span><span>&gt;</span> <span>--</span><span>help</span>
```

and [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will print out a brief summary of your script’s options:

```
<span></span>Usage: &lt;yourscript&gt; [options]

Options:
  -h, --help            show this help message and exit
  -f FILE, --file=FILE  write report to FILE
  -q, --quiet           don't print status messages to stdout
```

where the value of _yourscript_ is determined at runtime (normally from `sys.argv[0]`).

## Background

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") was explicitly designed to encourage the creation of programs with straightforward command-line interfaces that follow the conventions established by the `getopt()` family of functions available to C developers. To that end, it supports only the most common command-line syntax and semantics conventionally used under Unix. If you are unfamiliar with these conventions, reading this section will allow you to acquaint yourself with them.

### Terminology

argument

a string entered on the command-line, and passed by the shell to `execl()` or `execv()`. In Python, arguments are elements of `sys.argv[1:]` (`sys.argv[0]` is the name of the program being executed). Unix shells also use the term “word”.

It is occasionally desirable to substitute an argument list other than `sys.argv[1:]`, so you should read “argument” as “an element of `sys.argv[1:]`, or of some other list provided as a substitute for `sys.argv[1:]`”.

option

an argument used to supply extra information to guide or customize the execution of a program. There are many different syntaxes for options; the traditional Unix syntax is a hyphen (“-”) followed by a single letter, e.g. `-x` or `-F`. Also, traditional Unix syntax allows multiple options to be merged into a single argument, e.g. `-x -F` is equivalent to `-xF`. The GNU project introduced `--` followed by a series of hyphen-separated words, e.g. `--file` or `--dry-run`. These are the only two option syntaxes provided by [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.").

Some other option syntaxes that the world has seen include:

-   a hyphen followed by a few letters, e.g. `-pf` (this is _not_ the same as multiple options merged into a single argument)
    
-   a hyphen followed by a whole word, e.g. `-file` (this is technically equivalent to the previous syntax, but they aren’t usually seen in the same program)
    
-   a plus sign followed by a single letter, or a few letters, or a word, e.g. `+f`, `+rgb`
    
-   a slash followed by a letter, or a few letters, or a word, e.g. `/f`, `/file`
    

These option syntaxes are not supported by [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library."), and they never will be. This is deliberate: the first three are non-standard on any environment, and the last only makes sense if you’re exclusively targeting Windows or certain legacy platforms (e.g. VMS, MS-DOS).

option argument

an argument that follows an option, is closely associated with that option, and is consumed from the argument list when that option is. With [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library."), option arguments may either be in a separate argument from their option:

or included in the same argument:

Typically, a given option either takes an argument or it doesn’t. Lots of people want an “optional option arguments” feature, meaning that some options will take an argument if they see it, and won’t if they don’t. This is somewhat controversial, because it makes parsing ambiguous: if `-a` takes an optional argument and `-b` is another option entirely, how do we interpret `-ab`? Because of this ambiguity, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") does not support this feature.

positional argument

something leftover in the argument list after options have been parsed, i.e. after options and their arguments have been parsed and removed from the argument list.

required option

an option that must be supplied on the command-line; note that the phrase “required option” is self-contradictory in English. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") doesn’t prevent you from implementing required options, but doesn’t give you much help at it either.

For example, consider this hypothetical command-line:

```
<span></span><span>prog</span> <span>-</span><span>v</span> <span>--</span><span>report</span> <span>report</span><span>.</span><span>txt</span> <span>foo</span> <span>bar</span>
```

`-v` and `--report` are both options. Assuming that `--report` takes one argument, `report.txt` is an option argument. `foo` and `bar` are positional arguments.

### What are options for?

Options are used to provide extra information to tune or customize the execution of a program. In case it wasn’t clear, options are usually _optional_. A program should be able to run just fine with no options whatsoever. (Pick a random program from the Unix or GNU toolsets. Can it run without any options at all and still make sense? The main exceptions are `find`, `tar`, and `dd`—all of which are mutant oddballs that have been rightly criticized for their non-standard syntax and confusing interfaces.)

Lots of people want their programs to have “required options”. Think about it. If it’s required, then it’s _not optional_! If there is a piece of information that your program absolutely requires in order to run successfully, that’s what positional arguments are for.

As an example of good command-line interface design, consider the humble `cp` utility, for copying files. It doesn’t make much sense to try to copy files without supplying a destination and at least one source. Hence, `cp` fails if you run it with no arguments. However, it has a flexible, useful syntax that does not require any options at all:

```
<span></span><span>cp</span> <span>SOURCE</span> <span>DEST</span>
<span>cp</span> <span>SOURCE</span> <span>...</span> <span>DEST</span><span>-</span><span>DIR</span>
```

You can get pretty far with just that. Most `cp` implementations provide a bunch of options to tweak exactly how the files are copied: you can preserve mode and modification time, avoid following symlinks, ask before clobbering existing files, etc. But none of this distracts from the core mission of `cp`, which is to copy either one file to another, or several files to another directory.

### What are positional arguments for?

Positional arguments are for those pieces of information that your program absolutely, positively requires to run.

A good user interface should have as few absolute requirements as possible. If your program requires 17 distinct pieces of information in order to run successfully, it doesn’t much matter _how_ you get that information from the user—most people will give up and walk away before they successfully run the program. This applies whether the user interface is a command-line, a configuration file, or a GUI: if you make that many demands on your users, most of them will simply give up.

In short, try to minimize the amount of information that users are absolutely required to supply—use sensible defaults whenever possible. Of course, you also want to make your programs reasonably flexible. That’s what options are for. Again, it doesn’t matter if they are entries in a config file, widgets in the “Preferences” dialog of a GUI, or command-line options—the more options you implement, the more flexible your program is, and the more complicated its implementation becomes. Too much flexibility has drawbacks as well, of course; too many options can overwhelm users and make your code much harder to maintain.

## Tutorial

While [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") is quite flexible and powerful, it’s also straightforward to use in most cases. This section covers the code patterns that are common to any [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")\-based program.

First, you need to import the OptionParser class; then, early in the main program, create an OptionParser instance:

```
<span></span><span>from</span><span> </span><span>optparse</span><span> </span><span>import</span> <span>OptionParser</span>
<span>...</span>
<span>parser</span> <span>=</span> <span>OptionParser</span><span>()</span>
```

Then you can start defining options. The basic syntax is:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>opt_str</span><span>,</span> <span>...</span><span>,</span>
                  <span>attr</span><span>=</span><span>value</span><span>,</span> <span>...</span><span>)</span>
```

Each option has one or more option strings, such as `-f` or `--file`, and several option attributes that tell [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") what to expect and what to do when it encounters that option on the command line.

Typically, each option will have one short option string and one long option string, e.g.:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--file"</span><span>,</span> <span>...</span><span>)</span>
```

You’re free to define as many short option strings and as many long option strings as you like (including zero), as long as there is at least one option string overall.

The option strings passed to [`OptionParser.add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option") are effectively labels for the option defined by that call. For brevity, we will frequently refer to _encountering an option_ on the command line; in reality, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") encounters _option strings_ and looks up options from them.

Once all of your options are defined, instruct [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") to parse your program’s command line:

```
<span></span><span>(</span><span>options</span><span>,</span> <span>args</span><span>)</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>()</span>
```

(If you like, you can pass a custom argument list to [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args"), but that’s rarely necessary: by default it uses `sys.argv[1:]`.)

[`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args") returns two values:

-   `options`, an object containing values for all of your options—e.g. if `--file` takes a single string argument, then `options.file` will be the filename supplied by the user, or `None` if the user did not supply that option
    
-   `args`, the list of positional arguments leftover after parsing options
    

This tutorial section only covers the four most important option attributes: [`action`](https://docs.python.org/3/library/optparse.html#optparse.Option.action "optparse.Option.action"), [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type"), [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") (destination), and [`help`](https://docs.python.org/3/library/optparse.html#optparse.Option.help "optparse.Option.help"). Of these, [`action`](https://docs.python.org/3/library/optparse.html#optparse.Option.action "optparse.Option.action") is the most fundamental.

### Understanding option actions

Actions tell [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") what to do when it encounters an option on the command line. There is a fixed set of actions hard-coded into [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library."); adding new actions is an advanced topic covered in section [Extending optparse](https://docs.python.org/3/library/optparse.html#optparse-extending-optparse). Most actions tell [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") to store a value in some variable—for example, take a string from the command line and store it in an attribute of `options`.

If you don’t specify an option action, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") defaults to `store`.

### The store action

The most common option action is `store`, which tells [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") to take the next argument (or the remainder of the current argument), ensure that it is of the correct type, and store it to your chosen destination.

For example:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--file"</span><span>,</span>
                  <span>action</span><span>=</span><span>"store"</span><span>,</span> <span>type</span><span>=</span><span>"string"</span><span>,</span> <span>dest</span><span>=</span><span>"filename"</span><span>)</span>
```

Now let’s make up a fake command line and ask [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") to parse it:

```
<span></span><span>args</span> <span>=</span> <span>[</span><span>"-f"</span><span>,</span> <span>"foo.txt"</span><span>]</span>
<span>(</span><span>options</span><span>,</span> <span>args</span><span>)</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>(</span><span>args</span><span>)</span>
```

When [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") sees the option string `-f`, it consumes the next argument, `foo.txt`, and stores it in `options.filename`. So, after this call to [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args"), `options.filename` is `"foo.txt"`.

Some other option types supported by [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") are `int` and `float`. Here’s an option that expects an integer argument:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-n"</span><span>,</span> <span>type</span><span>=</span><span>"int"</span><span>,</span> <span>dest</span><span>=</span><span>"num"</span><span>)</span>
```

Note that this option has no long option string, which is perfectly acceptable. Also, there’s no explicit action, since the default is `store`.

Let’s parse another fake command-line. This time, we’ll jam the option argument right up against the option: since `-n42` (one argument) is equivalent to `-n 42` (two arguments), the code

```
<span></span><span>(</span><span>options</span><span>,</span> <span>args</span><span>)</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>([</span><span>"-n42"</span><span>])</span>
<span>print</span><span>(</span><span>options</span><span>.</span><span>num</span><span>)</span>
```

will print `42`.

If you don’t specify a type, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") assumes `string`. Combined with the fact that the default action is `store`, that means our first example can be a lot shorter:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--file"</span><span>,</span> <span>dest</span><span>=</span><span>"filename"</span><span>)</span>
```

If you don’t supply a destination, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") figures out a sensible default from the option strings: if the first long option string is `--foo-bar`, then the default destination is `foo_bar`. If there are no long option strings, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") looks at the first short option string: the default destination for `-f` is `f`.

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") also includes the built-in `complex` type. Adding types is covered in section [Extending optparse](https://docs.python.org/3/library/optparse.html#optparse-extending-optparse).

### Handling boolean (flag) options

Flag options—set a variable to true or false when a particular option is seen—are quite common. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") supports them with two separate actions, `store_true` and `store_false`. For example, you might have a `verbose` flag that is turned on with `-v` and off with `-q`:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
```

Here we have two different options with the same destination, which is perfectly OK. (It just means you have to be a bit careful when setting default values—see below.)

When [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") encounters `-v` on the command line, it sets `options.verbose` to `True`; when it encounters `-q`, `options.verbose` is set to `False`.

### Other actions

Some other actions supported by [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") are:

`"store_const"`

store a constant value, pre-set via [`Option.const`](https://docs.python.org/3/library/optparse.html#optparse.Option.const "optparse.Option.const")

`"append"`

append this option’s argument to a list

`"count"`

increment a counter by one

`"callback"`

call a specified function

These are covered in section [Reference Guide](https://docs.python.org/3/library/optparse.html#optparse-reference-guide), and section [Option Callbacks](https://docs.python.org/3/library/optparse.html#optparse-option-callbacks).

### Default values

All of the above examples involve setting some variable (the “destination”) when certain command-line options are seen. What happens if those options are never seen? Since we didn’t supply any defaults, they are all set to `None`. This is usually fine, but sometimes you want more control. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") lets you supply a default value for each destination, which is assigned before the command line is parsed.

First, consider the verbose/quiet example. If we want [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") to set `verbose` to `True` unless `-q` is seen, then we can do this:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span> <span>default</span><span>=</span><span>True</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
```

Since default values apply to the _destination_ rather than to any particular option, and these two options happen to have the same destination, this is exactly equivalent:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span> <span>default</span><span>=</span><span>True</span><span>)</span>
```

Consider this:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span> <span>default</span><span>=</span><span>False</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span> <span>default</span><span>=</span><span>True</span><span>)</span>
```

Again, the default value for `verbose` will be `True`: the last default value supplied for any particular destination is the one that counts.

A clearer way to specify default values is the `set_defaults()` method of OptionParser, which you can call at any time before calling [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args"):

```
<span></span><span>parser</span><span>.</span><span>set_defaults</span><span>(</span><span>verbose</span><span>=</span><span>True</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>...</span><span>)</span>
<span>(</span><span>options</span><span>,</span> <span>args</span><span>)</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>()</span>
```

As before, the last value specified for a given option destination is the one that counts. For clarity, try to use one method or the other of setting default values, not both.

### Generating help

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s ability to generate help and usage text automatically is useful for creating user-friendly command-line interfaces. All you have to do is supply a [`help`](https://docs.python.org/3/library/optparse.html#optparse.Option.help "optparse.Option.help") value for each option, and optionally a short usage message for your whole program. Here’s an OptionParser populated with user-friendly (documented) options:

```
<span></span><span>usage</span> <span>=</span> <span>"usage: %prog [options] arg1 arg2"</span>
<span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>usage</span><span>=</span><span>usage</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>"--verbose"</span><span>,</span>
                  <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span> <span>default</span><span>=</span><span>True</span><span>,</span>
                  <span>help</span><span>=</span><span>"make lots of noise [default]"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>"--quiet"</span><span>,</span>
                  <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span>
                  <span>help</span><span>=</span><span>"be vewwy quiet (I'm hunting wabbits)"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--filename"</span><span>,</span>
                  <span>metavar</span><span>=</span><span>"FILE"</span><span>,</span> <span>help</span><span>=</span><span>"write output to FILE"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-m"</span><span>,</span> <span>"--mode"</span><span>,</span>
                  <span>default</span><span>=</span><span>"intermediate"</span><span>,</span>
                  <span>help</span><span>=</span><span>"interaction mode: novice, intermediate, "</span>
                       <span>"or expert [default: </span><span>%d</span><span>efault]"</span><span>)</span>
```

If [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") encounters either `-h` or `--help` on the command-line, or if you just call `parser.print_help()`, it prints the following to standard output:

```
<span></span>Usage: &lt;yourscript&gt; [options] arg1 arg2

Options:
  -h, --help            show this help message and exit
  -v, --verbose         make lots of noise [default]
  -q, --quiet           be vewwy quiet (I'm hunting wabbits)
  -f FILE, --filename=FILE
                        write output to FILE
  -m MODE, --mode=MODE  interaction mode: novice, intermediate, or
                        expert [default: intermediate]
```

(If the help output is triggered by a help option, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") exits after printing the help text.)

There’s a lot going on here to help [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") generate the best possible help message:

-   the script defines its own usage message:
    
    ```
    <span></span><span>usage</span> <span>=</span> <span>"usage: %prog [options] arg1 arg2"</span>
    ```
    
    [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") expands `%prog` in the usage string to the name of the current program, i.e. `os.path.basename(sys.argv[0])`. The expanded string is then printed before the detailed option help.
    
    If you don’t supply a usage string, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") uses a bland but sensible default: `"Usage: %prog [options]"`, which is fine if your script doesn’t take any positional arguments.
    
-   every option defines a help string, and doesn’t worry about line-wrapping—[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") takes care of wrapping lines and making the help output look good.
    
-   options that take a value indicate this fact in their automatically generated help message, e.g. for the “mode” option:
    
    Here, “MODE” is called the meta-variable: it stands for the argument that the user is expected to supply to `-m`/`--mode`. By default, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") converts the destination variable name to uppercase and uses that for the meta-variable. Sometimes, that’s not what you want—for example, the `--filename` option explicitly sets `metavar="FILE"`, resulting in this automatically generated option description:
    
    This is important for more than just saving space, though: the manually written help text uses the meta-variable `FILE` to clue the user in that there’s a connection between the semi-formal syntax `-f FILE` and the informal semantic description “write output to FILE”. This is a simple but effective way to make your help text a lot clearer and more useful for end users.
    
-   options that have a default value can include `%default` in the help string—[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will replace it with [`str()`](https://docs.python.org/3/library/stdtypes.html#str "str") of the option’s default value. If an option has no default value (or the default value is `None`), `%default` expands to `none`.
    

#### Grouping Options

When dealing with many options, it is convenient to group these options for better help output. An [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser") can contain several option groups, each of which can contain several options.

An option group is obtained using the class [`OptionGroup`](https://docs.python.org/3/library/optparse.html#optparse.OptionGroup "optparse.OptionGroup"):

_class_ optparse.OptionGroup(_parser_, _title_, _description\=None_)

where

-   parser is the [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser") instance the group will be inserted in to
    
-   title is the group title
    
-   description, optional, is a long description of the group
    

[`OptionGroup`](https://docs.python.org/3/library/optparse.html#optparse.OptionGroup "optparse.OptionGroup") inherits from `OptionContainer` (like [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser")) and so the `add_option()` method can be used to add an option to the group.

Once all the options are declared, using the [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser") method `add_option_group()` the group is added to the previously defined parser.

Continuing with the parser defined in the previous section, adding an [`OptionGroup`](https://docs.python.org/3/library/optparse.html#optparse.OptionGroup "optparse.OptionGroup") to a parser is easy:

```
<span></span><span>group</span> <span>=</span> <span>OptionGroup</span><span>(</span><span>parser</span><span>,</span> <span>"Dangerous Options"</span><span>,</span>
                    <span>"Caution: use these options at your own risk.  "</span>
                    <span>"It is believed that some of them bite."</span><span>)</span>
<span>group</span><span>.</span><span>add_option</span><span>(</span><span>"-g"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>help</span><span>=</span><span>"Group option."</span><span>)</span>
<span>parser</span><span>.</span><span>add_option_group</span><span>(</span><span>group</span><span>)</span>
```

This would result in the following help output:

```
<span></span>Usage: &lt;yourscript&gt; [options] arg1 arg2

Options:
  -h, --help            show this help message and exit
  -v, --verbose         make lots of noise [default]
  -q, --quiet           be vewwy quiet (I'm hunting wabbits)
  -f FILE, --filename=FILE
                        write output to FILE
  -m MODE, --mode=MODE  interaction mode: novice, intermediate, or
                        expert [default: intermediate]

  Dangerous Options:
    Caution: use these options at your own risk.  It is believed that some
    of them bite.

    -g                  Group option.
```

A bit more complete example might involve using more than one group: still extending the previous example:

```
<span></span><span>group</span> <span>=</span> <span>OptionGroup</span><span>(</span><span>parser</span><span>,</span> <span>"Dangerous Options"</span><span>,</span>
                    <span>"Caution: use these options at your own risk.  "</span>
                    <span>"It is believed that some of them bite."</span><span>)</span>
<span>group</span><span>.</span><span>add_option</span><span>(</span><span>"-g"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>help</span><span>=</span><span>"Group option."</span><span>)</span>
<span>parser</span><span>.</span><span>add_option_group</span><span>(</span><span>group</span><span>)</span>

<span>group</span> <span>=</span> <span>OptionGroup</span><span>(</span><span>parser</span><span>,</span> <span>"Debug Options"</span><span>)</span>
<span>group</span><span>.</span><span>add_option</span><span>(</span><span>"-d"</span><span>,</span> <span>"--debug"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span>
                 <span>help</span><span>=</span><span>"Print debug information"</span><span>)</span>
<span>group</span><span>.</span><span>add_option</span><span>(</span><span>"-s"</span><span>,</span> <span>"--sql"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span>
                 <span>help</span><span>=</span><span>"Print all SQL statements executed"</span><span>)</span>
<span>group</span><span>.</span><span>add_option</span><span>(</span><span>"-e"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>help</span><span>=</span><span>"Print every action done"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option_group</span><span>(</span><span>group</span><span>)</span>
```

that results in the following output:

```
<span></span>Usage: &lt;yourscript&gt; [options] arg1 arg2

Options:
  -h, --help            show this help message and exit
  -v, --verbose         make lots of noise [default]
  -q, --quiet           be vewwy quiet (I'm hunting wabbits)
  -f FILE, --filename=FILE
                        write output to FILE
  -m MODE, --mode=MODE  interaction mode: novice, intermediate, or expert
                        [default: intermediate]

  Dangerous Options:
    Caution: use these options at your own risk.  It is believed that some
    of them bite.

    -g                  Group option.

  Debug Options:
    -d, --debug         Print debug information
    -s, --sql           Print all SQL statements executed
    -e                  Print every action done
```

Another interesting method, in particular when working programmatically with option groups is:

OptionParser.get\_option\_group(_opt\_str_)

Return the [`OptionGroup`](https://docs.python.org/3/library/optparse.html#optparse.OptionGroup "optparse.OptionGroup") to which the short or long option string _opt\_str_ (e.g. `'-o'` or `'--option'`) belongs. If there’s no such [`OptionGroup`](https://docs.python.org/3/library/optparse.html#optparse.OptionGroup "optparse.OptionGroup"), return `None`.

### Printing a version string

Similar to the brief usage string, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") can also print a version string for your program. You have to supply the string as the `version` argument to OptionParser:

```
<span></span><span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>usage</span><span>=</span><span>"%prog [-f] [-q]"</span><span>,</span> <span>version</span><span>=</span><span>"%prog 1.0"</span><span>)</span>
```

`%prog` is expanded just like it is in `usage`. Apart from that, `version` can contain anything you like. When you supply it, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") automatically adds a `--version` option to your parser. If it encounters this option on the command line, it expands your `version` string (by replacing `%prog`), prints it to stdout, and exits.

For example, if your script is called `/usr/bin/foo`:

```
<span></span><span>$ </span>/usr/bin/foo<span> </span>--version
<span>foo 1.0</span>
```

The following two methods can be used to print and get the `version` string:

OptionParser.print\_version(_file\=None_)

Print the version message for the current program (`self.version`) to _file_ (default stdout). As with [`print_usage()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.print_usage "optparse.OptionParser.print_usage"), any occurrence of `%prog` in `self.version` is replaced with the name of the current program. Does nothing if `self.version` is empty or undefined.

OptionParser.get\_version()

Same as [`print_version()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.print_version "optparse.OptionParser.print_version") but returns the version string instead of printing it.

### How [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") handles errors

There are two broad classes of errors that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") has to worry about: programmer errors and user errors. Programmer errors are usually erroneous calls to [`OptionParser.add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option"), e.g. invalid option strings, unknown option attributes, missing option attributes, etc. These are dealt with in the usual way: raise an exception (either [`optparse.OptionError`](https://docs.python.org/3/library/optparse.html#optparse.OptionError "optparse.OptionError") or [`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError "TypeError")) and let the program crash.

Handling user errors is much more important, since they are guaranteed to happen no matter how stable your code is. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") can automatically detect some user errors, such as bad option arguments (passing `-n 4x` where `-n` takes an integer argument), missing arguments (`-n` at the end of the command line, where `-n` takes an argument of any type). Also, you can call `OptionParser.error()` to signal an application-defined error condition:

```
<span></span><span>(</span><span>options</span><span>,</span> <span>args</span><span>)</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>()</span>
<span>...</span>
<span>if</span> <span>options</span><span>.</span><span>a</span> <span>and</span> <span>options</span><span>.</span><span>b</span><span>:</span>
    <span>parser</span><span>.</span><span>error</span><span>(</span><span>"options -a and -b are mutually exclusive"</span><span>)</span>
```

In either case, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") handles the error the same way: it prints the program’s usage message and an error message to standard error and exits with error status 2.

Consider the first example above, where the user passes `4x` to an option that takes an integer:

```
<span></span><span>$ </span>/usr/bin/foo<span> </span>-n<span> </span>4x
<span>Usage: foo [options]</span>

<span>foo: error: option -n: invalid integer value: '4x'</span>
```

Or, where the user fails to pass a value at all:

```
<span></span><span>$ </span>/usr/bin/foo<span> </span>-n
<span>Usage: foo [options]</span>

<span>foo: error: -n option requires an argument</span>
```

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")\-generated error messages take care always to mention the option involved in the error; be sure to do the same when calling `OptionParser.error()` from your application code.

If [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s default error-handling behaviour does not suit your needs, you’ll need to subclass OptionParser and override its `exit()` and/or `error()` methods.

### Putting it all together

Here’s what [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")\-based scripts usually look like:

```
<span></span><span>from</span><span> </span><span>optparse</span><span> </span><span>import</span> <span>OptionParser</span>
<span>...</span>
<span>def</span><span> </span><span>main</span><span>():</span>
    <span>usage</span> <span>=</span> <span>"usage: %prog [options] arg"</span>
    <span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>usage</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--file"</span><span>,</span> <span>dest</span><span>=</span><span>"filename"</span><span>,</span>
                      <span>help</span><span>=</span><span>"read data from FILENAME"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>"--verbose"</span><span>,</span>
                      <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>"--quiet"</span><span>,</span>
                      <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
    <span>...</span>
    <span>(</span><span>options</span><span>,</span> <span>args</span><span>)</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>()</span>
    <span>if</span> <span>len</span><span>(</span><span>args</span><span>)</span> <span>!=</span> <span>1</span><span>:</span>
        <span>parser</span><span>.</span><span>error</span><span>(</span><span>"incorrect number of arguments"</span><span>)</span>
    <span>if</span> <span>options</span><span>.</span><span>verbose</span><span>:</span>
        <span>print</span><span>(</span><span>"reading </span><span>%s</span><span>..."</span> <span>%</span> <span>options</span><span>.</span><span>filename</span><span>)</span>
    <span>...</span>

<span>if</span> <span>__name__</span> <span>==</span> <span>"__main__"</span><span>:</span>
    <span>main</span><span>()</span>
```

## Reference Guide

### Creating the parser

The first step in using [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") is to create an OptionParser instance.

_class_ optparse.OptionParser(_..._)

The OptionParser constructor has no required arguments, but a number of optional keyword arguments. You should always pass them as keyword arguments, i.e. do not rely on the order in which the arguments are declared.

`usage` (default: `"%prog [options]"`)

The usage summary to print when your program is run incorrectly or with a help option. When [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") prints the usage string, it expands `%prog` to `os.path.basename(sys.argv[0])` (or to `prog` if you passed that keyword argument). To suppress a usage message, pass the special value `optparse.SUPPRESS_USAGE`.

`option_list` (default: `[]`)

A list of Option objects to populate the parser with. The options in `option_list` are added after any options in `standard_option_list` (a class attribute that may be set by OptionParser subclasses), but before any version or help options. Deprecated; use [`add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option") after creating the parser instead.

`option_class` (default: optparse.Option)

Class to use when adding options to the parser in [`add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option").

`version` (default: `None`)

A version string to print when the user supplies a version option. If you supply a true value for `version`, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") automatically adds a version option with the single option string `--version`. The substring `%prog` is expanded the same as for `usage`.

`conflict_handler` (default: `"error"`)

Specifies what to do when options with conflicting option strings are added to the parser; see section [Conflicts between options](https://docs.python.org/3/library/optparse.html#optparse-conflicts-between-options).

`description` (default: `None`)

A paragraph of text giving a brief overview of your program. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") reformats this paragraph to fit the current terminal width and prints it when the user requests help (after `usage`, but before the list of options).

`formatter` (default: a new `IndentedHelpFormatter`)

An instance of optparse.HelpFormatter that will be used for printing help text. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") provides two concrete classes for this purpose: IndentedHelpFormatter and TitledHelpFormatter.

`add_help_option` (default: `True`)

If true, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will add a help option (with option strings `-h` and `--help`) to the parser.

`prog`

The string to use when expanding `%prog` in `usage` and `version` instead of `os.path.basename(sys.argv[0])`.

`epilog` (default: `None`)

A paragraph of help text to print after the option help.

### Populating the parser

There are several ways to populate the parser with options. The preferred way is by using [`OptionParser.add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option"), as shown in section [Tutorial](https://docs.python.org/3/library/optparse.html#optparse-tutorial). `add_option()` can be called in one of two ways:

-   pass it an Option instance (as returned by `make_option()`)
    
-   pass it any combination of positional and keyword arguments that are acceptable to `make_option()` (i.e., to the Option constructor), and it will create the Option instance for you
    

The other alternative is to pass a list of pre-constructed Option instances to the OptionParser constructor, as in:

```
<span></span><span>option_list</span> <span>=</span> <span>[</span>
    <span>make_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--filename"</span><span>,</span>
                <span>action</span><span>=</span><span>"store"</span><span>,</span> <span>type</span><span>=</span><span>"string"</span><span>,</span> <span>dest</span><span>=</span><span>"filename"</span><span>),</span>
    <span>make_option</span><span>(</span><span>"-q"</span><span>,</span> <span>"--quiet"</span><span>,</span>
                <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>),</span>
    <span>]</span>
<span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>option_list</span><span>=</span><span>option_list</span><span>)</span>
```

(`make_option()` is a factory function for creating Option instances; currently it is an alias for the Option constructor. A future version of [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") may split Option into several classes, and `make_option()` will pick the right class to instantiate. Do not instantiate Option directly.)

### Defining options

Each Option instance represents a set of synonymous command-line option strings, e.g. `-f` and `--file`. You can specify any number of short or long option strings, but you must specify at least one overall option string.

The canonical way to create an [`Option`](https://docs.python.org/3/library/optparse.html#optparse.Option "optparse.Option") instance is with the `add_option()` method of [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser").

OptionParser.add\_option(_option_)

OptionParser.add\_option(_\*opt\_str_, _attr=value_, _..._)

To define an option with only a short option string:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>attr</span><span>=</span><span>value</span><span>,</span> <span>...</span><span>)</span>
```

And to define an option with only a long option string:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--foo"</span><span>,</span> <span>attr</span><span>=</span><span>value</span><span>,</span> <span>...</span><span>)</span>
```

The keyword arguments define attributes of the new Option object. The most important option attribute is [`action`](https://docs.python.org/3/library/optparse.html#optparse.Option.action "optparse.Option.action"), and it largely determines which other attributes are relevant or required. If you pass irrelevant option attributes, or fail to pass required ones, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") raises an [`OptionError`](https://docs.python.org/3/library/optparse.html#optparse.OptionError "optparse.OptionError") exception explaining your mistake.

An option’s _action_ determines what [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") does when it encounters this option on the command-line. The standard option actions hard-coded into [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") are:

`"store"`

store this option’s argument (default)

`"store_const"`

store a constant value, pre-set via [`Option.const`](https://docs.python.org/3/library/optparse.html#optparse.Option.const "optparse.Option.const")

`"store_true"`

store `True`

`"store_false"`

store `False`

`"append"`

append this option’s argument to a list

`"append_const"`

append a constant value to a list, pre-set via [`Option.const`](https://docs.python.org/3/library/optparse.html#optparse.Option.const "optparse.Option.const")

`"count"`

increment a counter by one

`"callback"`

call a specified function

`"help"`

print a usage message including all options and the documentation for them

(If you don’t supply an action, the default is `"store"`. For this action, you may also supply [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") and [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") option attributes; see [Standard option actions](https://docs.python.org/3/library/optparse.html#optparse-standard-option-actions).)

As you can see, most actions involve storing or updating a value somewhere. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") always creates a special object for this, conventionally called `options`, which is an instance of [`optparse.Values`](https://docs.python.org/3/library/optparse.html#optparse.Values "optparse.Values").

_class_ optparse.Values

An object holding parsed argument names and values as attributes. Normally created by calling when calling [`OptionParser.parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args"), and can be overridden by a custom subclass passed to the _values_ argument of [`OptionParser.parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args") (as described in [Parsing arguments](https://docs.python.org/3/library/optparse.html#optparse-parsing-arguments)).

Option arguments (and various other values) are stored as attributes of this object, according to the [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") (destination) option attribute.

For example, when you call

one of the first things [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") does is create the `options` object:

If one of the options in this parser is defined with

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>,</span> <span>"--file"</span><span>,</span> <span>action</span><span>=</span><span>"store"</span><span>,</span> <span>type</span><span>=</span><span>"string"</span><span>,</span> <span>dest</span><span>=</span><span>"filename"</span><span>)</span>
```

and the command-line being parsed includes any of the following:

```
<span></span><span>-</span><span>ffoo</span>
<span>-</span><span>f</span> <span>foo</span>
<span>--</span><span>file</span><span>=</span><span>foo</span>
<span>--</span><span>file</span> <span>foo</span>
```

then [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library."), on seeing this option, will do the equivalent of

The [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") and [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") option attributes are almost as important as [`action`](https://docs.python.org/3/library/optparse.html#optparse.Option.action "optparse.Option.action"), but [`action`](https://docs.python.org/3/library/optparse.html#optparse.Option.action "optparse.Option.action") is the only one that makes sense for _all_ options.

### Option attributes

_class_ optparse.Option

A single command line argument, with various attributes passed by keyword to the constructor. Normally created with [`OptionParser.add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option") rather than directly, and can be overridden by a custom class via the _option\_class_ argument to [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser").

The following option attributes may be passed as keyword arguments to [`OptionParser.add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option"). If you pass an option attribute that is not relevant to a particular option, or fail to pass a required option attribute, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") raises [`OptionError`](https://docs.python.org/3/library/optparse.html#optparse.OptionError "optparse.OptionError").

Option.action

(default: `"store"`)

Determines [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s behaviour when this option is seen on the command line; the available options are documented [here](https://docs.python.org/3/library/optparse.html#optparse-standard-option-actions).

Option.type

(default: `"string"`)

The argument type expected by this option (e.g., `"string"` or `"int"`); the available option types are documented [here](https://docs.python.org/3/library/optparse.html#optparse-standard-option-types).

Option.dest

(default: derived from option strings)

If the option’s action implies writing or modifying a value somewhere, this tells [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") where to write it: [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") names an attribute of the `options` object that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") builds as it parses the command line.

Option.default

The value to use for this option’s destination if the option is not seen on the command line. See also [`OptionParser.set_defaults()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.set_defaults "optparse.OptionParser.set_defaults").

Option.nargs

(default: 1)

How many arguments of type [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") should be consumed when this option is seen. If > 1, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will store a tuple of values to [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest").

Option.const

For actions that store a constant value, the constant value to store.

Option.choices

For options of type `"choice"`, the list of strings the user may choose from.

Option.callback

For options with action `"callback"`, the callable to call when this option is seen. See section [Option Callbacks](https://docs.python.org/3/library/optparse.html#optparse-option-callbacks) for detail on the arguments passed to the callable.

Option.callback\_args

Option.callback\_kwargs

Additional positional and keyword arguments to pass to `callback` after the four standard callback arguments.

Option.help

Help text to print for this option when listing all available options after the user supplies a [`help`](https://docs.python.org/3/library/optparse.html#optparse.Option.help "optparse.Option.help") option (such as `--help`). If no help text is supplied, the option will be listed without help text. To hide this option, use the special value `optparse.SUPPRESS_HELP`.

Option.metavar

(default: derived from option strings)

Stand-in for the option argument(s) to use when printing help text. See section [Tutorial](https://docs.python.org/3/library/optparse.html#optparse-tutorial) for an example.

### Standard option actions

The various option actions all have slightly different requirements and effects. Most actions have several relevant option attributes which you may specify to guide [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s behaviour; a few have required attributes, which you must specify for any option using that action.

-   `"store"` \[relevant: [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type"), [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest"), [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs"), [`choices`](https://docs.python.org/3/library/optparse.html#optparse.Option.choices "optparse.Option.choices")\]
    
    The option must be followed by an argument, which is converted to a value according to [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") and stored in [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest"). If [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs") > 1, multiple arguments will be consumed from the command line; all will be converted according to [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") and stored to [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") as a tuple. See the [Standard option types](https://docs.python.org/3/library/optparse.html#optparse-standard-option-types) section.
    
    If [`choices`](https://docs.python.org/3/library/optparse.html#optparse.Option.choices "optparse.Option.choices") is supplied (a list or tuple of strings), the type defaults to `"choice"`.
    
    If [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") is not supplied, it defaults to `"string"`.
    
    If [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") is not supplied, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") derives a destination from the first long option string (e.g., `--foo-bar` implies `foo_bar`). If there are no long option strings, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") derives a destination from the first short option string (e.g., `-f` implies `f`).
    
    Example:
    
    ```
    <span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-f"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-p"</span><span>,</span> <span>type</span><span>=</span><span>"float"</span><span>,</span> <span>nargs</span><span>=</span><span>3</span><span>,</span> <span>dest</span><span>=</span><span>"point"</span><span>)</span>
    ```
    
    As it parses the command line
    
    ```
    <span></span><span>-</span><span>f</span> <span>foo</span><span>.</span><span>txt</span> <span>-</span><span>p</span> <span>1</span> <span>-</span><span>3.5</span> <span>4</span> <span>-</span><span>fbar</span><span>.</span><span>txt</span>
    ```
    
    [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will set
    
    ```
    <span></span><span>options</span><span>.</span><span>f</span> <span>=</span> <span>"foo.txt"</span>
    <span>options</span><span>.</span><span>point</span> <span>=</span> <span>(</span><span>1.0</span><span>,</span> <span>-</span><span>3.5</span><span>,</span> <span>4.0</span><span>)</span>
    <span>options</span><span>.</span><span>f</span> <span>=</span> <span>"bar.txt"</span>
    ```
    
-   `"store_const"` \[required: [`const`](https://docs.python.org/3/library/optparse.html#optparse.Option.const "optparse.Option.const"); relevant: [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest")\]
    
    The value [`const`](https://docs.python.org/3/library/optparse.html#optparse.Option.const "optparse.Option.const") is stored in [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest").
    
    Example:
    
    ```
    <span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-q"</span><span>,</span> <span>"--quiet"</span><span>,</span>
                      <span>action</span><span>=</span><span>"store_const"</span><span>,</span> <span>const</span><span>=</span><span>0</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>"--verbose"</span><span>,</span>
                      <span>action</span><span>=</span><span>"store_const"</span><span>,</span> <span>const</span><span>=</span><span>1</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--noisy"</span><span>,</span>
                      <span>action</span><span>=</span><span>"store_const"</span><span>,</span> <span>const</span><span>=</span><span>2</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>)</span>
    ```
    
    If `--noisy` is seen, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will set
    
-   `"store_true"` \[relevant: [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest")\]
    
    A special case of `"store_const"` that stores `True` to [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest").
    
-   `"store_false"` \[relevant: [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest")\]
    
    Like `"store_true"`, but stores `False`.
    
    Example:
    
    ```
    <span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--clobber"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"clobber"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--no-clobber"</span><span>,</span> <span>action</span><span>=</span><span>"store_false"</span><span>,</span> <span>dest</span><span>=</span><span>"clobber"</span><span>)</span>
    ```
    
-   `"append"` \[relevant: [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type"), [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest"), [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs"), [`choices`](https://docs.python.org/3/library/optparse.html#optparse.Option.choices "optparse.Option.choices")\]
    
    The option must be followed by an argument, which is appended to the list in [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest"). If no default value for [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") is supplied, an empty list is automatically created when [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") first encounters this option on the command-line. If [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs") > 1, multiple arguments are consumed, and a tuple of length [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs") is appended to [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest").
    
    The defaults for [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") and [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") are the same as for the `"store"` action.
    
    Example:
    
    ```
    <span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-t"</span><span>,</span> <span>"--tracks"</span><span>,</span> <span>action</span><span>=</span><span>"append"</span><span>,</span> <span>type</span><span>=</span><span>"int"</span><span>)</span>
    ```
    
    If `-t3` is seen on the command-line, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") does the equivalent of:
    
    ```
    <span></span><span>options</span><span>.</span><span>tracks</span> <span>=</span> <span>[]</span>
    <span>options</span><span>.</span><span>tracks</span><span>.</span><span>append</span><span>(</span><span>int</span><span>(</span><span>"3"</span><span>))</span>
    ```
    
    If, a little later on, `--tracks=4` is seen, it does:
    
    ```
    <span></span><span>options</span><span>.</span><span>tracks</span><span>.</span><span>append</span><span>(</span><span>int</span><span>(</span><span>"4"</span><span>))</span>
    ```
    
    The `append` action calls the `append` method on the current value of the option. This means that any default value specified must have an `append` method. It also means that if the default value is non-empty, the default elements will be present in the parsed value for the option, with any values from the command line appended after those default values:
    
    \>>>
    
    ```
    <span></span><span>&gt;&gt;&gt; </span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--files"</span><span>,</span> <span>action</span><span>=</span><span>"append"</span><span>,</span> <span>default</span><span>=</span><span>[</span><span>'~/.mypkg/defaults'</span><span>])</span>
    <span>&gt;&gt;&gt; </span><span>opts</span><span>,</span> <span>args</span> <span>=</span> <span>parser</span><span>.</span><span>parse_args</span><span>([</span><span>'--files'</span><span>,</span> <span>'overrides.mypkg'</span><span>])</span>
    <span>&gt;&gt;&gt; </span><span>opts</span><span>.</span><span>files</span>
    <span>['~/.mypkg/defaults', 'overrides.mypkg']</span>
    ```
    
-   `"append_const"` \[required: [`const`](https://docs.python.org/3/library/optparse.html#optparse.Option.const "optparse.Option.const"); relevant: [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest")\]
    
    Like `"store_const"`, but the value [`const`](https://docs.python.org/3/library/optparse.html#optparse.Option.const "optparse.Option.const") is appended to [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest"); as with `"append"`, [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") defaults to `None`, and an empty list is automatically created the first time the option is encountered.
    
-   `"count"` \[relevant: [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest")\]
    
    Increment the integer stored at [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest"). If no default value is supplied, [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") is set to zero before being incremented the first time.
    
    Example:
    
    ```
    <span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>action</span><span>=</span><span>"count"</span><span>,</span> <span>dest</span><span>=</span><span>"verbosity"</span><span>)</span>
    ```
    
    The first time `-v` is seen on the command line, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") does the equivalent of:
    
    ```
    <span></span><span>options</span><span>.</span><span>verbosity</span> <span>=</span> <span>0</span>
    <span>options</span><span>.</span><span>verbosity</span> <span>+=</span> <span>1</span>
    ```
    
    Every subsequent occurrence of `-v` results in
    
-   `"callback"` \[required: [`callback`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback "optparse.Option.callback"); relevant: [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type"), [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs"), [`callback_args`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_args "optparse.Option.callback_args"), [`callback_kwargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_kwargs "optparse.Option.callback_kwargs")\]
    
    Call the function specified by [`callback`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback "optparse.Option.callback"), which is called as
    
    ```
    <span></span><span>func</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>,</span> <span>*</span><span>args</span><span>,</span> <span>**</span><span>kwargs</span><span>)</span>
    ```
    
    See section [Option Callbacks](https://docs.python.org/3/library/optparse.html#optparse-option-callbacks) for more detail.
    
-   `"help"`
    
    Prints a complete help message for all the options in the current option parser. The help message is constructed from the `usage` string passed to OptionParser’s constructor and the [`help`](https://docs.python.org/3/library/optparse.html#optparse.Option.help "optparse.Option.help") string passed to every option.
    
    If no [`help`](https://docs.python.org/3/library/optparse.html#optparse.Option.help "optparse.Option.help") string is supplied for an option, it will still be listed in the help message. To omit an option entirely, use the special value `optparse.SUPPRESS_HELP`.
    
    [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") automatically adds a [`help`](https://docs.python.org/3/library/optparse.html#optparse.Option.help "optparse.Option.help") option to all OptionParsers, so you do not normally need to create one.
    
    Example:
    
    ```
    <span></span><span>from</span><span> </span><span>optparse</span><span> </span><span>import</span> <span>OptionParser</span><span>,</span> <span>SUPPRESS_HELP</span>
    
    <span># usually, a help option is added automatically, but that can</span>
    <span># be suppressed using the add_help_option argument</span>
    <span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>add_help_option</span><span>=</span><span>False</span><span>)</span>
    
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-h"</span><span>,</span> <span>"--help"</span><span>,</span> <span>action</span><span>=</span><span>"help"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-v"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"verbose"</span><span>,</span>
                      <span>help</span><span>=</span><span>"Be moderately verbose"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--file"</span><span>,</span> <span>dest</span><span>=</span><span>"filename"</span><span>,</span>
                      <span>help</span><span>=</span><span>"Input file to read data from"</span><span>)</span>
    <span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--secret"</span><span>,</span> <span>help</span><span>=</span><span>SUPPRESS_HELP</span><span>)</span>
    ```
    
    If [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") sees either `-h` or `--help` on the command line, it will print something like the following help message to stdout (assuming `sys.argv[0]` is `"foo.py"`):
    
    ```
    <span></span>Usage: foo.py [options]
    
    Options:
      -h, --help        Show this help message and exit
      -v                Be moderately verbose
      --file=FILENAME   Input file to read data from
    ```
    
    After printing the help message, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") terminates your process with `sys.exit(0)`.
    
-   `"version"`
    
    Prints the version number supplied to the OptionParser to stdout and exits. The version number is actually formatted and printed by the `print_version()` method of OptionParser. Generally only relevant if the `version` argument is supplied to the OptionParser constructor. As with [`help`](https://docs.python.org/3/library/optparse.html#optparse.Option.help "optparse.Option.help") options, you will rarely create `version` options, since [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") automatically adds them when needed.
    

### Standard option types

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") has five built-in option types: `"string"`, `"int"`, `"choice"`, `"float"` and `"complex"`. If you need to add new option types, see section [Extending optparse](https://docs.python.org/3/library/optparse.html#optparse-extending-optparse).

Arguments to string options are not checked or converted in any way: the text on the command line is stored in the destination (or passed to the callback) as-is.

Integer arguments (type `"int"`) are parsed as follows:

-   if the number starts with `0x`, it is parsed as a hexadecimal number
    
-   if the number starts with `0`, it is parsed as an octal number
    
-   if the number starts with `0b`, it is parsed as a binary number
    
-   otherwise, the number is parsed as a decimal number
    

The conversion is done by calling [`int()`](https://docs.python.org/3/library/functions.html#int "int") with the appropriate base (2, 8, 10, or 16). If this fails, so will [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library."), although with a more useful error message.

`"float"` and `"complex"` option arguments are converted directly with [`float()`](https://docs.python.org/3/library/functions.html#float "float") and [`complex()`](https://docs.python.org/3/library/functions.html#complex "complex"), with similar error-handling.

`"choice"` options are a subtype of `"string"` options. The [`choices`](https://docs.python.org/3/library/optparse.html#optparse.Option.choices "optparse.Option.choices") option attribute (a sequence of strings) defines the set of allowed option arguments. `optparse.check_choice()` compares user-supplied option arguments against this master list and raises [`OptionValueError`](https://docs.python.org/3/library/optparse.html#optparse.OptionValueError "optparse.OptionValueError") if an invalid string is given.

### Parsing arguments

The whole point of creating and populating an OptionParser is to call its [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args") method.

OptionParser.parse\_args(_args\=None_, _values\=None_)

Parse the command-line options found in _args_.

The input parameters are

`args`

the list of arguments to process (default: `sys.argv[1:]`)

`values`

a [`Values`](https://docs.python.org/3/library/optparse.html#optparse.Values "optparse.Values") object to store option arguments in (default: a new instance of [`Values`](https://docs.python.org/3/library/optparse.html#optparse.Values "optparse.Values")) – if you give an existing object, the option defaults will not be initialized on it

and the return value is a pair `(options, args)` where

`options`

the same object that was passed in as _values_, or the `optparse.Values` instance created by [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")

`args`

the leftover positional arguments after all options have been processed

The most common usage is to supply neither keyword argument. If you supply `values`, it will be modified with repeated [`setattr()`](https://docs.python.org/3/library/functions.html#setattr "setattr") calls (roughly one for every option argument stored to an option destination) and returned by [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args").

If [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args") encounters any errors in the argument list, it calls the OptionParser’s `error()` method with an appropriate end-user error message. This ultimately terminates your process with an exit status of 2 (the traditional Unix exit status for command-line errors).

### Querying and manipulating your option parser

The default behavior of the option parser can be customized slightly, and you can also poke around your option parser and see what’s there. OptionParser provides several methods to help you out:

OptionParser.disable\_interspersed\_args()

Set parsing to stop on the first non-option. For example, if `-a` and `-b` are both simple options that take no arguments, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") normally accepts this syntax:

and treats it as equivalent to

To disable this feature, call [`disable_interspersed_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.disable_interspersed_args "optparse.OptionParser.disable_interspersed_args"). This restores traditional Unix syntax, where option parsing stops with the first non-option argument.

Use this if you have a command processor which runs another command which has options of its own and you want to make sure these options don’t get confused. For example, each command might have a different set of options.

OptionParser.enable\_interspersed\_args()

Set parsing to not stop on the first non-option, allowing interspersing switches with command arguments. This is the default behavior.

OptionParser.get\_option(_opt\_str_)

Returns the Option instance with the option string _opt\_str_, or `None` if no options have that option string.

OptionParser.has\_option(_opt\_str_)

Return `True` if the OptionParser has an option with option string _opt\_str_ (e.g., `-q` or `--verbose`).

OptionParser.remove\_option(_opt\_str_)

If the [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser") has an option corresponding to _opt\_str_, that option is removed. If that option provided any other option strings, all of those option strings become invalid. If _opt\_str_ does not occur in any option belonging to this [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser"), raises [`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError "ValueError").

### Conflicts between options

If you’re not careful, it’s easy to define options with conflicting option strings:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-n"</span><span>,</span> <span>"--dry-run"</span><span>,</span> <span>...</span><span>)</span>
<span>...</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-n"</span><span>,</span> <span>"--noisy"</span><span>,</span> <span>...</span><span>)</span>
```

(This is particularly true if you’ve defined your own OptionParser subclass with some standard options.)

Every time you add an option, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") checks for conflicts with existing options. If it finds any, it invokes the current conflict-handling mechanism. You can set the conflict-handling mechanism either in the constructor:

```
<span></span><span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>...</span><span>,</span> <span>conflict_handler</span><span>=</span><span>handler</span><span>)</span>
```

or with a separate call:

```
<span></span><span>parser</span><span>.</span><span>set_conflict_handler</span><span>(</span><span>handler</span><span>)</span>
```

The available conflict handlers are:

> `"error"` (default)
> 
> assume option conflicts are a programming error and raise [`OptionConflictError`](https://docs.python.org/3/library/optparse.html#optparse.OptionConflictError "optparse.OptionConflictError")
> 
> `"resolve"`
> 
> resolve option conflicts intelligently (see below)

As an example, let’s define an [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser") that resolves conflicts intelligently and add conflicting options to it:

```
<span></span><span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>conflict_handler</span><span>=</span><span>"resolve"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-n"</span><span>,</span> <span>"--dry-run"</span><span>,</span> <span>...</span><span>,</span> <span>help</span><span>=</span><span>"do no harm"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-n"</span><span>,</span> <span>"--noisy"</span><span>,</span> <span>...</span><span>,</span> <span>help</span><span>=</span><span>"be noisy"</span><span>)</span>
```

At this point, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") detects that a previously added option is already using the `-n` option string. Since `conflict_handler` is `"resolve"`, it resolves the situation by removing `-n` from the earlier option’s list of option strings. Now `--dry-run` is the only way for the user to activate that option. If the user asks for help, the help message will reflect that:

```
<span></span><span>Options</span><span>:</span>
  <span>--</span><span>dry</span><span>-</span><span>run</span>     <span>do</span> <span>no</span> <span>harm</span>
  <span>...</span>
  <span>-</span><span>n</span><span>,</span> <span>--</span><span>noisy</span>   <span>be</span> <span>noisy</span>
```

It’s possible to whittle away the option strings for a previously added option until there are none left, and the user has no way of invoking that option from the command-line. In that case, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") removes that option completely, so it doesn’t show up in help text or anywhere else. Carrying on with our existing OptionParser:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--dry-run"</span><span>,</span> <span>...</span><span>,</span> <span>help</span><span>=</span><span>"new dry-run option"</span><span>)</span>
```

At this point, the original `-n`/`--dry-run` option is no longer accessible, so [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") removes it, leaving this help text:

```
<span></span><span>Options</span><span>:</span>
  <span>...</span>
  <span>-</span><span>n</span><span>,</span> <span>--</span><span>noisy</span>   <span>be</span> <span>noisy</span>
  <span>--</span><span>dry</span><span>-</span><span>run</span>     <span>new</span> <span>dry</span><span>-</span><span>run</span> <span>option</span>
```

### Cleanup

OptionParser instances have several cyclic references. This should not be a problem for Python’s garbage collector, but you may wish to break the cyclic references explicitly by calling `destroy()` on your OptionParser once you are done with it. This is particularly useful in long-running applications where large object graphs are reachable from your OptionParser.

### Other methods

OptionParser supports several other public methods:

OptionParser.set\_usage(_usage_)

Set the usage string according to the rules described above for the `usage` constructor keyword argument. Passing `None` sets the default usage string; use `optparse.SUPPRESS_USAGE` to suppress a usage message.

OptionParser.print\_usage(_file\=None_)

Print the usage message for the current program (`self.usage`) to _file_ (default stdout). Any occurrence of the string `%prog` in `self.usage` is replaced with the name of the current program. Does nothing if `self.usage` is empty or not defined.

OptionParser.get\_usage()

Same as [`print_usage()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.print_usage "optparse.OptionParser.print_usage") but returns the usage string instead of printing it.

OptionParser.set\_defaults(_dest=value_, _..._)

Set default values for several option destinations at once. Using [`set_defaults()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.set_defaults "optparse.OptionParser.set_defaults") is the preferred way to set default values for options, since multiple options can share the same destination. For example, if several “mode” options all set the same destination, any one of them can set the default, and the last one wins:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--advanced"</span><span>,</span> <span>action</span><span>=</span><span>"store_const"</span><span>,</span>
                  <span>dest</span><span>=</span><span>"mode"</span><span>,</span> <span>const</span><span>=</span><span>"advanced"</span><span>,</span>
                  <span>default</span><span>=</span><span>"novice"</span><span>)</span>    <span># overridden below</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--novice"</span><span>,</span> <span>action</span><span>=</span><span>"store_const"</span><span>,</span>
                  <span>dest</span><span>=</span><span>"mode"</span><span>,</span> <span>const</span><span>=</span><span>"novice"</span><span>,</span>
                  <span>default</span><span>=</span><span>"advanced"</span><span>)</span>  <span># overrides above setting</span>
```

To avoid this confusion, use [`set_defaults()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.set_defaults "optparse.OptionParser.set_defaults"):

```
<span></span><span>parser</span><span>.</span><span>set_defaults</span><span>(</span><span>mode</span><span>=</span><span>"advanced"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--advanced"</span><span>,</span> <span>action</span><span>=</span><span>"store_const"</span><span>,</span>
                  <span>dest</span><span>=</span><span>"mode"</span><span>,</span> <span>const</span><span>=</span><span>"advanced"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--novice"</span><span>,</span> <span>action</span><span>=</span><span>"store_const"</span><span>,</span>
                  <span>dest</span><span>=</span><span>"mode"</span><span>,</span> <span>const</span><span>=</span><span>"novice"</span><span>)</span>
```

## Option Callbacks

When [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s built-in actions and types aren’t quite enough for your needs, you have two choices: extend [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") or define a callback option. Extending [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") is more general, but overkill for a lot of simple cases. Quite often a simple callback is all you need.

There are two steps to defining a callback option:

-   define the option itself using the `"callback"` action
    
-   write the callback; this is a function (or method) that takes at least four arguments, as described below
    

### Defining a callback option

As always, the easiest way to define a callback option is by using the [`OptionParser.add_option()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.add_option "optparse.OptionParser.add_option") method. Apart from [`action`](https://docs.python.org/3/library/optparse.html#optparse.Option.action "optparse.Option.action"), the only option attribute you must specify is `callback`, the function to call:

```
<span></span><span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-c"</span><span>,</span> <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>my_callback</span><span>)</span>
```

`callback` is a function (or other callable object), so you must have already defined `my_callback()` when you create this callback option. In this simple case, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") doesn’t even know if `-c` takes any arguments, which usually means that the option takes no arguments—the mere presence of `-c` on the command-line is all it needs to know. In some circumstances, though, you might want your callback to consume an arbitrary number of command-line arguments. This is where writing callbacks gets tricky; it’s covered later in this section.

[`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") always passes four particular arguments to your callback, and it will only pass additional arguments if you specify them via [`callback_args`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_args "optparse.Option.callback_args") and [`callback_kwargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_kwargs "optparse.Option.callback_kwargs"). Thus, the minimal callback function signature is:

```
<span></span><span>def</span><span> </span><span>my_callback</span><span>(</span><span>option</span><span>,</span> <span>opt</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>):</span>
```

The four arguments to a callback are described below.

There are several other option attributes that you can supply when you define a callback option:

[`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type")

has its usual meaning: as with the `"store"` or `"append"` actions, it instructs [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") to consume one argument and convert it to [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type"). Rather than storing the converted value(s) anywhere, though, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") passes it to your callback function.

[`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs")

also has its usual meaning: if it is supplied and > 1, [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will consume [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs") arguments, each of which must be convertible to [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type"). It then passes a tuple of converted values to your callback.

[`callback_args`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_args "optparse.Option.callback_args")

a tuple of extra positional arguments to pass to the callback

[`callback_kwargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_kwargs "optparse.Option.callback_kwargs")

a dictionary of extra keyword arguments to pass to the callback

### How callbacks are called

All callbacks are called as follows:

```
<span></span><span>func</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>,</span> <span>*</span><span>args</span><span>,</span> <span>**</span><span>kwargs</span><span>)</span>
```

where

`option`

is the Option instance that’s calling the callback

`opt_str`

is the option string seen on the command-line that’s triggering the callback. (If an abbreviated long option was used, `opt_str` will be the full, canonical option string—e.g. if the user puts `--foo` on the command-line as an abbreviation for `--foobar`, then `opt_str` will be `"--foobar"`.)

`value`

is the argument to this option seen on the command-line. [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") will only expect an argument if [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") is set; the type of `value` will be the type implied by the option’s type. If [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") for this option is `None` (no argument expected), then `value` will be `None`. If [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs") > 1, `value` will be a tuple of values of the appropriate type.

`parser`

is the OptionParser instance driving the whole thing, mainly useful because you can access some other interesting data through its instance attributes:

`parser.largs`

the current list of leftover arguments, ie. arguments that have been consumed but are neither options nor option arguments. Feel free to modify `parser.largs`, e.g. by adding more arguments to it. (This list will become `args`, the second return value of [`parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args").)

`parser.rargs`

the current list of remaining arguments, ie. with `opt_str` and `value` (if applicable) removed, and only the arguments following them still there. Feel free to modify `parser.rargs`, e.g. by consuming more arguments.

`parser.values`

the object where option values are by default stored (an instance of optparse.OptionValues). This lets callbacks use the same mechanism as the rest of [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") for storing option values; you don’t need to mess around with globals or closures. You can also access or modify the value(s) of any options already encountered on the command-line.

`args`

is a tuple of arbitrary positional arguments supplied via the [`callback_args`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_args "optparse.Option.callback_args") option attribute.

`kwargs`

is a dictionary of arbitrary keyword arguments supplied via [`callback_kwargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.callback_kwargs "optparse.Option.callback_kwargs").

### Raising errors in a callback

The callback function should raise [`OptionValueError`](https://docs.python.org/3/library/optparse.html#optparse.OptionValueError "optparse.OptionValueError") if there are any problems with the option or its argument(s). [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") catches this and terminates the program, printing the error message you supply to stderr. Your message should be clear, concise, accurate, and mention the option at fault. Otherwise, the user will have a hard time figuring out what they did wrong.

### Callback example 1: trivial callback

Here’s an example of a callback option that takes no arguments, and simply records that the option was seen:

```
<span></span><span>def</span><span> </span><span>record_foo_seen</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>):</span>
    <span>parser</span><span>.</span><span>values</span><span>.</span><span>saw_foo</span> <span>=</span> <span>True</span>

<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--foo"</span><span>,</span> <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>record_foo_seen</span><span>)</span>
```

Of course, you could do that with the `"store_true"` action.

### Callback example 2: check option order

Here’s a slightly more interesting example: record the fact that `-a` is seen, but blow up if it comes after `-b` in the command-line.

```
<span></span><span>def</span><span> </span><span>check_order</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>):</span>
    <span>if</span> <span>parser</span><span>.</span><span>values</span><span>.</span><span>b</span><span>:</span>
        <span>raise</span> <span>OptionValueError</span><span>(</span><span>"can't use -a after -b"</span><span>)</span>
    <span>parser</span><span>.</span><span>values</span><span>.</span><span>a</span> <span>=</span> <span>1</span>
<span>...</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-a"</span><span>,</span> <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>check_order</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-b"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"b"</span><span>)</span>
```

### Callback example 3: check option order (generalized)

If you want to reuse this callback for several similar options (set a flag, but blow up if `-b` has already been seen), it needs a bit of work: the error message and the flag that it sets must be generalized.

```
<span></span><span>def</span><span> </span><span>check_order</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>):</span>
    <span>if</span> <span>parser</span><span>.</span><span>values</span><span>.</span><span>b</span><span>:</span>
        <span>raise</span> <span>OptionValueError</span><span>(</span><span>"can't use </span><span>%s</span><span> after -b"</span> <span>%</span> <span>opt_str</span><span>)</span>
    <span>setattr</span><span>(</span><span>parser</span><span>.</span><span>values</span><span>,</span> <span>option</span><span>.</span><span>dest</span><span>,</span> <span>1</span><span>)</span>
<span>...</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-a"</span><span>,</span> <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>check_order</span><span>,</span> <span>dest</span><span>=</span><span>'a'</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-b"</span><span>,</span> <span>action</span><span>=</span><span>"store_true"</span><span>,</span> <span>dest</span><span>=</span><span>"b"</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-c"</span><span>,</span> <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>check_order</span><span>,</span> <span>dest</span><span>=</span><span>'c'</span><span>)</span>
```

### Callback example 4: check arbitrary condition

Of course, you could put any condition in there—you’re not limited to checking the values of already-defined options. For example, if you have options that should not be called when the moon is full, all you have to do is this:

```
<span></span><span>def</span><span> </span><span>check_moon</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>):</span>
    <span>if</span> <span>is_moon_full</span><span>():</span>
        <span>raise</span> <span>OptionValueError</span><span>(</span><span>"</span><span>%s</span><span> option invalid when moon is full"</span>
                               <span>%</span> <span>opt_str</span><span>)</span>
    <span>setattr</span><span>(</span><span>parser</span><span>.</span><span>values</span><span>,</span> <span>option</span><span>.</span><span>dest</span><span>,</span> <span>1</span><span>)</span>
<span>...</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--foo"</span><span>,</span>
                  <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>check_moon</span><span>,</span> <span>dest</span><span>=</span><span>"foo"</span><span>)</span>
```

(The definition of `is_moon_full()` is left as an exercise for the reader.)

### Callback example 5: fixed arguments

Things get slightly more interesting when you define callback options that take a fixed number of arguments. Specifying that a callback option takes arguments is similar to defining a `"store"` or `"append"` option: if you define [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type"), then the option takes one argument that must be convertible to that type; if you further define [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs"), then the option takes [`nargs`](https://docs.python.org/3/library/optparse.html#optparse.Option.nargs "optparse.Option.nargs") arguments.

Here’s an example that just emulates the standard `"store"` action:

```
<span></span><span>def</span><span> </span><span>store_value</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>):</span>
    <span>setattr</span><span>(</span><span>parser</span><span>.</span><span>values</span><span>,</span> <span>option</span><span>.</span><span>dest</span><span>,</span> <span>value</span><span>)</span>
<span>...</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"--foo"</span><span>,</span>
                  <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>store_value</span><span>,</span>
                  <span>type</span><span>=</span><span>"int"</span><span>,</span> <span>nargs</span><span>=</span><span>3</span><span>,</span> <span>dest</span><span>=</span><span>"foo"</span><span>)</span>
```

Note that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") takes care of consuming 3 arguments and converting them to integers for you; all you have to do is store them. (Or whatever; obviously you don’t need a callback for this example.)

### Callback example 6: variable arguments

Things get hairy when you want an option to take a variable number of arguments. For this case, you must write a callback, as [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") doesn’t provide any built-in capabilities for it. And you have to deal with certain intricacies of conventional Unix command-line parsing that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") normally handles for you. In particular, callbacks should implement the conventional rules for bare `--` and `-` arguments:

-   either `--` or `-` can be option arguments
    
-   bare `--` (if not the argument to some option): halt command-line processing and discard the `--`
    
-   bare `-` (if not the argument to some option): halt command-line processing but keep the `-` (append it to `parser.largs`)
    

If you want an option that takes a variable number of arguments, there are several subtle, tricky issues to worry about. The exact implementation you choose will be based on which trade-offs you’re willing to make for your application (which is why [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") doesn’t support this sort of thing directly).

Nevertheless, here’s a stab at a callback for an option with variable arguments:

```
<span></span><span>def</span><span> </span><span>vararg_callback</span><span>(</span><span>option</span><span>,</span> <span>opt_str</span><span>,</span> <span>value</span><span>,</span> <span>parser</span><span>):</span>
    <span>assert</span> <span>value</span> <span>is</span> <span>None</span>
    <span>value</span> <span>=</span> <span>[]</span>

    <span>def</span><span> </span><span>floatable</span><span>(</span><span>str</span><span>):</span>
        <span>try</span><span>:</span>
            <span>float</span><span>(</span><span>str</span><span>)</span>
            <span>return</span> <span>True</span>
        <span>except</span> <span>ValueError</span><span>:</span>
            <span>return</span> <span>False</span>

    <span>for</span> <span>arg</span> <span>in</span> <span>parser</span><span>.</span><span>rargs</span><span>:</span>
        <span># stop on --foo like options</span>
        <span>if</span> <span>arg</span><span>[:</span><span>2</span><span>]</span> <span>==</span> <span>"--"</span> <span>and</span> <span>len</span><span>(</span><span>arg</span><span>)</span> <span>&gt;</span> <span>2</span><span>:</span>
            <span>break</span>
        <span># stop on -a, but not on -3 or -3.0</span>
        <span>if</span> <span>arg</span><span>[:</span><span>1</span><span>]</span> <span>==</span> <span>"-"</span> <span>and</span> <span>len</span><span>(</span><span>arg</span><span>)</span> <span>&gt;</span> <span>1</span> <span>and</span> <span>not</span> <span>floatable</span><span>(</span><span>arg</span><span>):</span>
            <span>break</span>
        <span>value</span><span>.</span><span>append</span><span>(</span><span>arg</span><span>)</span>

    <span>del</span> <span>parser</span><span>.</span><span>rargs</span><span>[:</span><span>len</span><span>(</span><span>value</span><span>)]</span>
    <span>setattr</span><span>(</span><span>parser</span><span>.</span><span>values</span><span>,</span> <span>option</span><span>.</span><span>dest</span><span>,</span> <span>value</span><span>)</span>

<span>...</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-c"</span><span>,</span> <span>"--callback"</span><span>,</span> <span>dest</span><span>=</span><span>"vararg_attr"</span><span>,</span>
                  <span>action</span><span>=</span><span>"callback"</span><span>,</span> <span>callback</span><span>=</span><span>vararg_callback</span><span>)</span>
```

## Extending [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")

Since the two major controlling factors in how [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") interprets command-line options are the action and type of each option, the most likely direction of extension is to add new actions and new types.

### Adding new types

To add new types, you need to define your own subclass of [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s [`Option`](https://docs.python.org/3/library/optparse.html#optparse.Option "optparse.Option") class. This class has a couple of attributes that define [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s types: [`TYPES`](https://docs.python.org/3/library/optparse.html#optparse.Option.TYPES "optparse.Option.TYPES") and [`TYPE_CHECKER`](https://docs.python.org/3/library/optparse.html#optparse.Option.TYPE_CHECKER "optparse.Option.TYPE_CHECKER").

Option.TYPES

A tuple of type names; in your subclass, simply define a new tuple [`TYPES`](https://docs.python.org/3/library/optparse.html#optparse.Option.TYPES "optparse.Option.TYPES") that builds on the standard one.

Option.TYPE\_CHECKER

A dictionary mapping type names to type-checking functions. A type-checking function has the following signature:

```
<span></span><span>def</span><span> </span><span>check_mytype</span><span>(</span><span>option</span><span>,</span> <span>opt</span><span>,</span> <span>value</span><span>)</span>
```

where `option` is an [`Option`](https://docs.python.org/3/library/optparse.html#optparse.Option "optparse.Option") instance, `opt` is an option string (e.g., `-f`), and `value` is the string from the command line that must be checked and converted to your desired type. `check_mytype()` should return an object of the hypothetical type `mytype`. The value returned by a type-checking function will wind up in the OptionValues instance returned by [`OptionParser.parse_args()`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser.parse_args "optparse.OptionParser.parse_args"), or be passed to a callback as the `value` parameter.

Your type-checking function should raise [`OptionValueError`](https://docs.python.org/3/library/optparse.html#optparse.OptionValueError "optparse.OptionValueError") if it encounters any problems. [`OptionValueError`](https://docs.python.org/3/library/optparse.html#optparse.OptionValueError "optparse.OptionValueError") takes a single string argument, which is passed as-is to [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser")’s `error()` method, which in turn prepends the program name and the string `"error:"` and prints everything to stderr before terminating the process.

Here’s a silly example that demonstrates adding a `"complex"` option type to parse Python-style complex numbers on the command line. (This is even sillier than it used to be, because [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") 1.3 added built-in support for complex numbers, but never mind.)

First, the necessary imports:

```
<span></span><span>from</span><span> </span><span>copy</span><span> </span><span>import</span> <span>copy</span>
<span>from</span><span> </span><span>optparse</span><span> </span><span>import</span> <span>Option</span><span>,</span> <span>OptionValueError</span>
```

You need to define your type-checker first, since it’s referred to later (in the [`TYPE_CHECKER`](https://docs.python.org/3/library/optparse.html#optparse.Option.TYPE_CHECKER "optparse.Option.TYPE_CHECKER") class attribute of your Option subclass):

```
<span></span><span>def</span><span> </span><span>check_complex</span><span>(</span><span>option</span><span>,</span> <span>opt</span><span>,</span> <span>value</span><span>):</span>
    <span>try</span><span>:</span>
        <span>return</span> <span>complex</span><span>(</span><span>value</span><span>)</span>
    <span>except</span> <span>ValueError</span><span>:</span>
        <span>raise</span> <span>OptionValueError</span><span>(</span>
            <span>"option </span><span>%s</span><span>: invalid complex value: </span><span>%r</span><span>"</span> <span>%</span> <span>(</span><span>opt</span><span>,</span> <span>value</span><span>))</span>
```

Finally, the Option subclass:

```
<span></span><span>class</span><span> </span><span>MyOption</span> <span>(</span><span>Option</span><span>):</span>
    <span>TYPES</span> <span>=</span> <span>Option</span><span>.</span><span>TYPES</span> <span>+</span> <span>(</span><span>"complex"</span><span>,)</span>
    <span>TYPE_CHECKER</span> <span>=</span> <span>copy</span><span>(</span><span>Option</span><span>.</span><span>TYPE_CHECKER</span><span>)</span>
    <span>TYPE_CHECKER</span><span>[</span><span>"complex"</span><span>]</span> <span>=</span> <span>check_complex</span>
```

(If we didn’t make a [`copy()`](https://docs.python.org/3/library/copy.html#module-copy "copy: Shallow and deep copy operations.") of [`Option.TYPE_CHECKER`](https://docs.python.org/3/library/optparse.html#optparse.Option.TYPE_CHECKER "optparse.Option.TYPE_CHECKER"), we would end up modifying the [`TYPE_CHECKER`](https://docs.python.org/3/library/optparse.html#optparse.Option.TYPE_CHECKER "optparse.Option.TYPE_CHECKER") attribute of [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")’s Option class. This being Python, nothing stops you from doing that except good manners and common sense.)

That’s it! Now you can write a script that uses the new option type just like any other [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.")\-based script, except you have to instruct your OptionParser to use MyOption instead of Option:

```
<span></span><span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>option_class</span><span>=</span><span>MyOption</span><span>)</span>
<span>parser</span><span>.</span><span>add_option</span><span>(</span><span>"-c"</span><span>,</span> <span>type</span><span>=</span><span>"complex"</span><span>)</span>
```

Alternately, you can build your own option list and pass it to OptionParser; if you don’t use `add_option()` in the above way, you don’t need to tell OptionParser which option class to use:

```
<span></span><span>option_list</span> <span>=</span> <span>[</span><span>MyOption</span><span>(</span><span>"-c"</span><span>,</span> <span>action</span><span>=</span><span>"store"</span><span>,</span> <span>type</span><span>=</span><span>"complex"</span><span>,</span> <span>dest</span><span>=</span><span>"c"</span><span>)]</span>
<span>parser</span> <span>=</span> <span>OptionParser</span><span>(</span><span>option_list</span><span>=</span><span>option_list</span><span>)</span>
```

### Adding new actions

Adding new actions is a bit trickier, because you have to understand that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") has a couple of classifications for actions:

“store” actions

actions that result in [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") storing a value to an attribute of the current OptionValues instance; these options require a [`dest`](https://docs.python.org/3/library/optparse.html#optparse.Option.dest "optparse.Option.dest") attribute to be supplied to the Option constructor.

“typed” actions

actions that take a value from the command line and expect it to be of a certain type; or rather, a string that can be converted to a certain type. These options require a [`type`](https://docs.python.org/3/library/optparse.html#optparse.Option.type "optparse.Option.type") attribute to the Option constructor.

These are overlapping sets: some default “store” actions are `"store"`, `"store_const"`, `"append"`, and `"count"`, while the default “typed” actions are `"store"`, `"append"`, and `"callback"`.

When you add an action, you need to categorize it by listing it in at least one of the following class attributes of Option (all are lists of strings):

Option.ACTIONS

All actions must be listed in ACTIONS.

Option.STORE\_ACTIONS

“store” actions are additionally listed here.

Option.TYPED\_ACTIONS

“typed” actions are additionally listed here.

Option.ALWAYS\_TYPED\_ACTIONS

Actions that always take a type (i.e. whose options always take a value) are additionally listed here. The only effect of this is that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") assigns the default type, `"string"`, to options with no explicit type whose action is listed in [`ALWAYS_TYPED_ACTIONS`](https://docs.python.org/3/library/optparse.html#optparse.Option.ALWAYS_TYPED_ACTIONS "optparse.Option.ALWAYS_TYPED_ACTIONS").

In order to actually implement your new action, you must override Option’s `take_action()` method and add a case that recognizes your action.

For example, let’s add an `"extend"` action. This is similar to the standard `"append"` action, but instead of taking a single value from the command-line and appending it to an existing list, `"extend"` will take multiple values in a single comma-delimited string, and extend an existing list with them. That is, if `--names` is an `"extend"` option of type `"string"`, the command line

```
<span></span><span>--</span><span>names</span><span>=</span><span>foo</span><span>,</span><span>bar</span> <span>--</span><span>names</span> <span>blah</span> <span>--</span><span>names</span> <span>ding</span><span>,</span><span>dong</span>
```

would result in a list

```
<span></span><span>[</span><span>"foo"</span><span>,</span> <span>"bar"</span><span>,</span> <span>"blah"</span><span>,</span> <span>"ding"</span><span>,</span> <span>"dong"</span><span>]</span>
```

Again we define a subclass of Option:

```
<span></span><span>class</span><span> </span><span>MyOption</span><span>(</span><span>Option</span><span>):</span>

    <span>ACTIONS</span> <span>=</span> <span>Option</span><span>.</span><span>ACTIONS</span> <span>+</span> <span>(</span><span>"extend"</span><span>,)</span>
    <span>STORE_ACTIONS</span> <span>=</span> <span>Option</span><span>.</span><span>STORE_ACTIONS</span> <span>+</span> <span>(</span><span>"extend"</span><span>,)</span>
    <span>TYPED_ACTIONS</span> <span>=</span> <span>Option</span><span>.</span><span>TYPED_ACTIONS</span> <span>+</span> <span>(</span><span>"extend"</span><span>,)</span>
    <span>ALWAYS_TYPED_ACTIONS</span> <span>=</span> <span>Option</span><span>.</span><span>ALWAYS_TYPED_ACTIONS</span> <span>+</span> <span>(</span><span>"extend"</span><span>,)</span>

    <span>def</span><span> </span><span>take_action</span><span>(</span><span>self</span><span>,</span> <span>action</span><span>,</span> <span>dest</span><span>,</span> <span>opt</span><span>,</span> <span>value</span><span>,</span> <span>values</span><span>,</span> <span>parser</span><span>):</span>
        <span>if</span> <span>action</span> <span>==</span> <span>"extend"</span><span>:</span>
            <span>lvalue</span> <span>=</span> <span>value</span><span>.</span><span>split</span><span>(</span><span>","</span><span>)</span>
            <span>values</span><span>.</span><span>ensure_value</span><span>(</span><span>dest</span><span>,</span> <span>[])</span><span>.</span><span>extend</span><span>(</span><span>lvalue</span><span>)</span>
        <span>else</span><span>:</span>
            <span>Option</span><span>.</span><span>take_action</span><span>(</span>
                <span>self</span><span>,</span> <span>action</span><span>,</span> <span>dest</span><span>,</span> <span>opt</span><span>,</span> <span>value</span><span>,</span> <span>values</span><span>,</span> <span>parser</span><span>)</span>
```

Features of note:

-   `"extend"` both expects a value on the command-line and stores that value somewhere, so it goes in both [`STORE_ACTIONS`](https://docs.python.org/3/library/optparse.html#optparse.Option.STORE_ACTIONS "optparse.Option.STORE_ACTIONS") and [`TYPED_ACTIONS`](https://docs.python.org/3/library/optparse.html#optparse.Option.TYPED_ACTIONS "optparse.Option.TYPED_ACTIONS").
    
-   to ensure that [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") assigns the default type of `"string"` to `"extend"` actions, we put the `"extend"` action in [`ALWAYS_TYPED_ACTIONS`](https://docs.python.org/3/library/optparse.html#optparse.Option.ALWAYS_TYPED_ACTIONS "optparse.Option.ALWAYS_TYPED_ACTIONS") as well.
    
-   `MyOption.take_action()` implements just this one new action, and passes control back to `Option.take_action()` for the standard [`optparse`](https://docs.python.org/3/library/optparse.html#module-optparse "optparse: Command-line option parsing library.") actions.
    
-   `values` is an instance of the optparse\_parser.Values class, which provides the very useful `ensure_value()` method. `ensure_value()` is essentially [`getattr()`](https://docs.python.org/3/library/functions.html#getattr "getattr") with a safety valve; it is called as
    
    ```
    <span></span><span>values</span><span>.</span><span>ensure_value</span><span>(</span><span>attr</span><span>,</span> <span>value</span><span>)</span>
    ```
    
    If the `attr` attribute of `values` doesn’t exist or is `None`, then ensure\_value() first sets it to `value`, and then returns `value`. This is very handy for actions like `"extend"`, `"append"`, and `"count"`, all of which accumulate data in a variable and expect that variable to be of a certain type (a list for the first two, an integer for the latter). Using `ensure_value()` means that scripts using your action don’t have to worry about setting a default value for the option destinations in question; they can just leave the default as `None` and `ensure_value()` will take care of getting it right when it’s needed.
    

## Exceptions

_exception_ optparse.OptionError

Raised if an [`Option`](https://docs.python.org/3/library/optparse.html#optparse.Option "optparse.Option") instance is created with invalid or inconsistent arguments.

_exception_ optparse.OptionConflictError

Raised if conflicting options are added to an [`OptionParser`](https://docs.python.org/3/library/optparse.html#optparse.OptionParser "optparse.OptionParser").

_exception_ optparse.OptionValueError

Raised if an invalid option value is encountered on the command line.

_exception_ optparse.BadOptionError

Raised if an invalid option is passed on the command line.

_exception_ optparse.AmbiguousOptionError

Raised if an ambiguous option is passed on the command line.
