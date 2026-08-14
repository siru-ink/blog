+++
title = "What makes a programming language whole?"
date = "2026-03-31"
+++
_Programming language_ is an immensely overloaded term. It can describe anything
between a certain type of logical structuring (such as object-oriented,
functional, or script programing) or different layers of the abstraction stack
(such as systems, application, or web programming). This long-form post will
create a basic framework for a potential mental-model of what a language is,
alongside clarifying various related terms.

In essence, the term can be split into four major and two minor components. The
major ones being: the _language specification_, the _built-ins_, the
_standard library_, and the _compiler/interpreter_. Whereas the minor ones are:
the _community packages_ and the _writing guidelines_. These terms will all be
explained in the explicit context of the
[Go programming language](https://en.wikipedia.org/wiki/Go_(programming_language)),
but they similarly apply to all other common programming languages.

## Major Programing Language Components

### Programming Language Specification

[This](https://go.dev/ref/spec) is the official specification for the Go
programing language. It is an overview of the correct syntax for the text files
written according to the Go language. It explains things like: what are proper
variable names, how can a variable be initialized, and how to define the return
type of a function. It does not however do any computing based on the content
itself, i.e. it does not demonstrate what machine code should explicitely be run
when a particular syntax token is written in the text. It is merely a high level
overview that specifies things like: unlike the Python programming language Go
uses the `:=` assignment operator.

### The Built-ins

[Built-ins](https://go.dev/ref/spec#Built-in_functions)[^1] are code chunks that
are meant to perform actions that do not explicitely need to be imported into
the code in some way. For example the `len()` built-in function can be used to
figure out the number of characters in a given string. It can be used anywhere,
and at any time in a program without requiring special syntax to first make it
available. In contrast, `fmt.Println()` first has to be imported via
`import fmt` before it can be used in a code piece. These are usually the core
building blocks that can be used to describe logic in new custom software. This
is the first level of actual runnable code provided by the creators of a
language.

### The Standard Library

Other code that is provided by the language developers directly is considered
the [standard library](https://pkg.go.dev/std). This is also runnable code that
is normally provided directly alongside the language specification. It is
generally meant to provide common, useful tools that any program might need,
such as printing out text to the terminal, as mentioned in the above example of
`fmt.Println()`.[^10] Since it is always shipped directly with the language
documentation, it is generally easier to import than other external, community
packages. In the case of Go, packages in the standard library can be directly
imported via their name, such as `import "fmt"`, whereas community packages have
to be imported with a path specifier, such as `github.com/modernc`. However,
simply because they are easily accessible does not indicate them to  necessarily
be the best solution to all problems. Often times the standard library is
limited in update frequency by the updates made to the language specification
itself. I.e. sometimes community packages have solved a problem better than the
implementation in the standard library has.[^2]

### The Compiler/Interpreter

The compiler/interpreter[^3] is the part of a language that allows the actual
execution of code written according to the language specification. Since the Go
language is based on compilation,[^4] I will stick to compilers in this section,
but interpreters laregely cover the same concepts. Many different programing
languages exist, however any one specific CPU architecture only has a specific
set of commands it knows how to execute, known as
[machine code](/understanding-the-executable-stack/#lowest-level-the-a-hrefhttpsenwikipediaorgwikicentral_processing_unitcpua-central-processing-unit).
In order to convert the text written based on the language specification into
this machine code, the compiler is used. Most programming languages have their
official reference compiler. In the case of Go, this is the
[Go compiler](https://go.dev/src/cmd/compile/README). However, one a language
specification exists, other compilers can be created which also comprehend this
specification. For example, the
[GCC compiler](https://en.wikipedia.org/wiki/GNU_Compiler_Collection) has also
added [a backend specific to the Go language](https://gcc.gnu.org/onlinedocs/gccgo/).[^5]

## Minor Programing Language Components

### The Community Packages

Beyond the standard library, given time, other programers will also write and
publish code written according to a particular language specification. This
turns into the community packages for the programing language. In the case of Go
a limited overview can be found at the [Go package index](https://pkg.go.dev).
Unlike some other programming languages, the Go index does not store packages
directly, but rather maintains a reference of the location for code repositories
storing the code for community packages. However, the working principle remains
the same. It allows coders to download code published (and licensed[^7]) by
other creators to then incorporate it into their own source code. The only
difference between this code and the standard library code is that it is not
directly published along side the specification and compiler, but rather has to
be specially downloaded on an as-needed basis.[^6]

### The Writing Guidelines

Lastly, programing languages come with varying levels of rigor in relation to
their writing guidelines. Go has a fairly strict system, largely enforced via
the `go fmt` program, but not all languages have such a strict set of rules.[^8]
These guidelines can easily be confused with the language specification itself,
but technically they are design choices that should have no effect on the output
created by the compiler.[^9] To give an example, it is considered more proper to
write the following in Go code:

```go
import (
    "fmt"
    "net/http"
)
```

whereas writing this instead, would still be completely valid according to the
language specification alone:

```go
import "fmt"
import "net/http"
```

___

[^1]: A perhaps easier to digest listing of the Go language's built-ins can be
      found [here](https://pkg.go.dev/builtin). However, it should be noted,
      that the canonical definition is found in the language specification, and
      not in this package.

[^2]: Different programming languages have starkly differing opinions about what
      constitutes a good standard library. For example, Go focusses on having
      [a lot of functionality built into the standard library directly](https://dev.to/godofgeeks/working-with-the-standard-library-in-go-io-fmt-etc-31o1)
      to prevent the need for external packages. In contrast,
      [JavaScript](https://en.wikipedia.org/wiki/JavaScript) and
      [Rust](https://en.wikipedia.org/wiki/Rust_(programming_language)) adhere
      to the principal that most functionality should be provided by external
      packages and only absolute necessities should be provided by the official
      standard library. There is a lot of literature on the question of which
      approach is best ([link](https://news.ycombinator.com/item?id=35286221),
      [link](https://alexgaynor.net/2025/may/19/standard-libraries/)); I do not
      wish to comment on the subject any more than necessary.

[^3]: Going into the details between what a compiler versus an interpreter is,
      would be outside the scope of this blog post. To put it simply, a copmiler takes
      in text and outputs machine code directly executable by a computer's CPU,
      without needing the compiler to be present at runtime. In contrast, an
      interpreter is a program that is run on the code, which then executes the
      instructions of the program in real time and therefore must be present to be
      able to run the program.

[^4]: Technically, it should be noted that a given language specification does
      not imply that a programming language necessarily must be compiled or
      interpreted. Based on the same spec, it would entirely be possible to
      offer both alternatives. However, coding either a reference compiler or
      interpreter is quite difficult, so in practice most languages are limited
      to one or the other. In the case of go, both can be seen somewhat though,
      for example a Go program can directly be executed via `go run` while a
      compiled version can be created with `go build`.

[^5]: It can be an interesting field of research to understand the
      advantages/disadvantages of different compiler options for a given
      language specificaiton. Some outdated comparisons between `go build` and
      `gccgo` can be seen [here](https://github.com/quasilyte/gccgo_vs_gc).
      It should also be noted that different compilers do not always produce
      identical machine code for the same input code, due to the compilers
      applying different optimizations to their input.

[^6]: It also comes with other differences such as: a higher likelyhood of
      supply-chain troubles, varying licenses, different authors, versioning
      separate from changes to the language specification, etc.

[^7]: Program licenses is a topic all for itself. In essence any code written is
      copyrighted by default to the person who originally wrote it. In order for
      others to be allowed to reuse it, they must obtain a license for use. Some
      commonly known licenses are: [MIT](https://spdx.org/licenses/MIT.html),
      [GPLv3](https://spdx.org/licenses/GPL-3.0-or-later.html),
      [BSD-3-clause](https://spdx.org/licenses/BSD-3-Clause.html), but there are
      many others as well. For some more info, see
      [here](https://choosealicense.com).

[^8]: For example, Google publishes
      [style guides used internally](https://google.github.io/styleguide/) for
      many different programming languages, even though these are completely
      separate from the actual language specifications.

[^9]: Technically, writing guideline changes should not have an effect on the
      execution of the program. However, this is not always the case. One
      notable example would be Python code, in which reordering import
      statements can lead to a difference in code output due to the way
      different namespaces can be shadowed by a later import. See the section on
      _5. Shadowing Standard Library Modules_ for an example
      [here](https://readmedium.com/common-issues-and-bugs-with-imports-in-python-57c07bfd9117).

[^10]: Even though the standard library is often times seen as omni-present,
       this is not always the case. For example in the Rust programming
       language, the crate (package) containing the standard library (`std`) is
       not guranteed to be usable on embedded hardware devices.
