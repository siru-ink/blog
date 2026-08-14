+++
title = "Understanding the Executable Stack"
date = "2026-03-29"
+++
Thanks to modern programming languages, most entry level programmers do not have
to deal with the peculiarities of how to get a computer to actually do anything.
Instead, one can simply write a program such as:

```python
print("Hello, world!")
```

and then merily type `python3 <filename>` into a shell, and somewhat
miraculousely the program will run. In the following few paragraphs I will try
to provide a bottom-up view of how the computer actually manages to run
programs.

## Lowest Level: The [CPU](https://en.wikipedia.org/wiki/Central_processing_unit) (Central Processing Unit)

As the name suggests, this is the actual hardware component that does
computational/processing logic.[^1] The CPU is a piece of electrical hardware,
that due to it's inate structure can process information in certain ways. These
CPU _built-in functions_ is what
[machine code](https://en.wikipedia.org/wiki/Machine_code) operates on. Machine
code is the lowest level of software code that exists. It is part of the
contents of most executable files that you can commonly find on operating
systems, such as the `ls` program.[^2] Machine code is completely CPU
architecture dependent, such as AMD64.[^3] It is by default a binary format,
which makes it immensely difficult for humans to read. Instead, there is the
next level up, called Assembly, which makes this code somewhat accessible to
humans.

## Somewhere in the middle: [Assembly](https://en.wikipedia.org/wiki/Assembly_language)

This used to also commonly be referred to as *symbolic machine code*, which
shows what it was meant to be. In a certain CPU architecture, assembly code is a
one-to-one translation of binary machine cope
[operation codes](https://en.wikipedia.org/wiki/Opcode) into a human-readable,
i.e. ASCII, abbreviation. It is a basic syntax that simply lists one CPU
instruction per line, with a shorthand for the
[CPU registers](https://en.wikipedia.org/wiki/Processor_register) being affected
by the instruction. In a sense, this can be considered the original programming
language, as it is the first abstraction level on this list to have a proper
syntax in text form. However, due to the one-to-one nature, it is still
dependent on the specific processor architecture, and is also an immensely
verbose way to define program logic, as it is limited by the operations
available to a given processor.[^4]

## The Lowest Level Commonly in Use: The [C Lang](https://en.wikipedia.org/wiki/C_(programming_language))

C is one of the oldest still commonly used programming languages. The main
advantage it provides above assembly code is it's ability to abstract individual
processor's capabilities away from the actual program logic. The programer is
given tools such as `functions`, `if/else`-constructs, and `loops` to define
program _business logic_ instead of having to write code the depends on a
particular architecture. And these function of these tools are defined in the C
language specification, see ISO 9899. This specification has evolved over time,
albeit quite a bit slower than commonly known programming languages, which has
given us C language specifications such as C99, C11, C17, and C23. However, this
also comes with the downside, that a CPU itself is no longer able to execute the
program. Instead, the C code first has to be translated into machine code
applicable to a particular CPU architecture. This gave rise to the concept of a
compiler, the most notable of which are gcc and clang.[^5] At a high level of
abstractions, these are programs, themselves in machine code, that process text
files and turn them into executable machine code for a particular CPU
architecture.

## The High Level: Interpreted Languages such as [Python](https://en.wikipedia.org/wiki/Python_(programming_language))

At the top end of the abstraction stack, there are interpreted languages such as
Python. Concisely put, these run a program in machine code that then itself
executes the text files comprising the user created program in real time. In the
case of Python, there is [CPython](https://github.com/python/cpython), which is
an interpreter for the Python language specification, which itself is written in
C and then compiled into machine code for various different architectures. The
various supported architectures by CPython can i.e. be seen at the bottom of
[this Python release page](https://www.python.org/downloads/release/python-3143/).

Theoretically speaking, it would be possible to write a compiler similar to gcc
which can process text files written according to the Python language
specification and turn them directly into machine code, however due to some
design choices in the language specification itself, this is
[rather difficult](https://stackoverflow.com/questions/138521/is-it-feasible-to-compile-python-to-machine-code).[^6] There are some project in existence today that attempt to compile parts of the interpreted code into machine code, such as [PyPy](https://pypy.org) and [Nuitka](https://nuitka.net).

This is the reason why one usually cannot simply type the name of a Python
executable directly into a shell for executaion, but rather passes it into the
_Ptyhon program_ as an argument when running the program, such as:

```bash
python3 <my-python-program>.py
```

## Addendum and Other Interesting Tidbits

### An Installed Python Program Can Run Directly. Why?

Sometimes programs written in Python are included in operating system package
managers, such as apt, and can then be run directly from the shell without the
need to first call the Python interpreter program. At least this is what is
shown to the user on firt glance. However this normally is simply a layer of
shell magic that is abstracting away the actual command being run. As an
example, there is the tool `img2pdf`, which is written completely in Python.
Installing it makes it accessible in my bash shell via `img2pdf`, however, on
closer inspection, the file being run by this command is actaully a shell script
located in my user folder at `~/.local/bin/img2pdf`.

```python
#!/Users/siru/.local/share/uv/tools/img2pdf/bin/python
# -*- coding: utf-8 -*-
import sys
from img2pdf import main
if __name__ == "__main__":
    if sys.argv[0].endswith("-script.pyw"):
        sys.argv[0] = sys.argv[0][:-11]
    elif sys.argv[0].endswith(".exe"):
        sys.argv[0] = sys.argv[0][:-4]
    sys.exit(main())
```

This script actually uses a
[shebang](https://en.wikipedia.org/wiki/Shebang_%28Unix%29) line to call a
python interpreter at `~/.local/share/uv/tools/img2pdf/python` which is then fed
the actual Python code which is to be executed. In other word, while it may seem
that one is executing a Python program directly, one is actually executing a
shell script which is being interpreted by my shell to run the Python
interpreter automatically.

### What about the Java Virtual Machine (JVM)?

In other words, why does Java code have to be compiled if it still need the
[JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) as an interpreter? The
easy answer would be that the java compiler does not create an output of machine
code specific for the host computer's architecture, but rather for a synthetic
CPU architecture known as the Java Virtual Machine.

*But why was this done?* Java created a revolutionary way of solving the cross
compilation problem. Specifically, most software is not scoped exclusively by
the code freshly written, but also pulls in code from the language's standard
library and third-party libraries. On a host system, these are all already
present in a way that is accessible to the compiler to be able to link against.
However, when trying to compile for a CPU architecture different than the host
computer's, all this information needs to be made available to the compiler. In
theory, this is possible, but to most common users, this is not exactly
feasible.[^7] So instead, Java opted to create synthetic CPU arhictecture as the
JVM, which then could be used as the only CPU architecture that the java
compiler, `javac`, can compile to as a target. This way all the information
necessary to comilation can be provided as part of the JDK directly. In turn,
this allowed for a compiled Java program to automatically run on any computer
and CPU architecture that could run the JVM, which made distributing programs
between computers much easier for end users.

___

[^1]: One can definitely go to a lower level even than the CPU, but this can
      conveniently be considered the boundary between coding (computer science)
      and hardware design (physics). Personally, I am more interested in the
      coding side of things, and that is largely also the limit as to what one
      can change at home. It is
      [prohibitively difficult to build your own CPU](https://dev.to/adityabhuyan/how-to-design-a-cpu-from-scratch-50pl).

[^2]: Technically, these executable files are not direct machine code, but
      rather
      [ELF files](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format).
      They are archives with machine code and some other stuff that the
      operating system kernel can use to load the executable and necessary state
      into memory before the CPU starts processing the machine code.

[^3]: Do not quote me on the accuracy of saying that AMD64 specifies a
      particular machine code syntax. There is also the issue of RISC vs CISC,
      that should have an effect on how the machine code needs to be written,
      but that is outside the scope of this blog post, and perhaps even this
      entire blog.

[^4]: For example, most processors do not have built-in functions for queries
      such as `if/else` or `for`. They mostly rely on addition, register
      substitution, and jumps to other instruction set locations.

[^5]: Technically, a compiler such as gcc is a bit more complex. It actually is
      at least a two part process in wich text is first turned into an internal
      representation, and in a second step converted from the internal
      representation to machine code. This allows gcc to take in code written in
      different language specifications, i.e. C, C++, Fortran, etc., and also
      output to different machine code platforms, such as AMD64, PowerPC, etc. I
      might do a post with more detail on this in the future, if so I will link
      it here.

[^6]: Very generally speaking, compilers can perform better the more they know
      about what the code will be doing. In languages such as Python, that have
      both dynamic typing and the possibility ov `eval` on user-input strings,
      the compiler would have to be extremely complex to deal with all the
      potential outcomes and paths that the input code could take.

[^7]: Even programmers don't always know everything about every detail in how
      the system works. What a shock.
