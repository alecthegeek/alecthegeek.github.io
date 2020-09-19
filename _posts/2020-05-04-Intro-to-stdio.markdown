---
layout: post
title: Beginners Introduction to using Standard I/O
tags: [python, learntocode]
status: publish
type: post
published: true
meta: {}
---

## Introduction

If you are a novice programmer sometimes you sometimes hear phrases like "read from standard input",
or read things similar to "write to stdout",
which is very confusing when you first come across it.

This article aims to show you what Standard Input/Output (`stdio`) is, and how to use it.

We are going to use Python 3 and the Bash shell -- but stdio works identically
in PowerShell and Windows, and the file I/O concepts are the same in other programing languages.
However, because I am using Python, we are also going to
see some Python specific features that help make file I/O more pleasant to use.

These examples
try to be as simple as possible and easy to understand, but also complete and correct.
You will find it useful to look at the source code at the same time as reading this explanation
and you can find the example code [here](https://github.com/alecthegeek/Intro2/tree/trunk/stdio)

Note that standard I/O is frequently used in text based programs, that we run
from the terminal for instance. Some GUI programs also use standard I/O, but it is less
common.

## What advantages does standard I/O give us?

1. The biggest benefit is that programs have a standard way to communicate with each other,
so that they can be used as part of something bigger.
They can become components in larger, custom, solutions

2. Because standard I/O is set up for us by the calling program at run time (for example the Bash shell),
then there is less work for the developer to do to handle I/O in a standard way.

Let's build up our understanding through a series of examples.

## Reading from the keyboard and writing to the screen

Let's go back to basics...

The program [`reverseTextLines0.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines0.py)
uses the well known functions `input()` and `print()` for I/O:

* Read a line of text from the keyboard
* Write a line of text back to the screen.

In between the `input()` and `print()` a local function (called `revLine()`) reverses the word order in the line.

If we run the program then  we can type in a phrase and see some output:

```
$ ./reverseTextLines0.py 
Please type in line of text : Hello from Earth and Moon
Moon and Earth from Hello
```

However this program has a couple of problems:

1. It only handles a single line of input text

2. It reads from the keyboard and writes to the screen, which can be problem if we have a file of input or
need to save the output for later use.

Let's solve those problems

## Reading and writing files

The next set of examples use a file based approach instead.

Look at [`reverseTextLines1.beta1.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines1.beta1.py):

1. Two files are opened, one for input and one for output (`myFile` and `yourFile`)

    ```python
    infile  = open("myFile", "r")
    outfile = open("yourFile", "w")
    ```

2. It uses a `while` loop to read each line in turn so that we can process each line of input text as needed

3. During the file reading operation, the program has to check to see if it's reached the end of the input (end of file => `eof`)

    ```python

    line = infile.readline()

    while line != '':  # '' is end of file

        ... #Process Content in line variable

        line = infile.readline()
    ```

4. After processing all the input it releases resources by closing the files.
Note that developers should careful to clear up (close files in this case), even if the program terminates with an error

    ```python
    infile.close()
    outfile.close()
    ```

So that fixed the previous problems, but it could be improved:

1. The file names are hard coded,
so we either have to make sure we always use the same file names,
change the source code each time to use the relevant file names or add additional code to
get the file names at run time using command line arguments<sup>[1](#parameters)</sup>.

2. It's hard to make this part of something bigger because it's not a general solution

3. As developers we have to do a fair bit of housekeeping, i.e. opening and closing files, and checking for end of file (eof).

Python has some additional sugar to help reduce the developer workload and the amount of housekeeping we need to do:

### Checking for end of file

If you look at
[`reverseTextLines1.beta2.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines1.beta2.py) we use the file
iterator to manage the reading of content **and** handle eof checking:

```python
for line in infile:
    ... #Process Content in line variable
```

This much simpler than the while loop and eof checking in previous examples.

### Closing files automatically

Python also has a [context manager](https://docs.python.org/3/reference/datamodel.html#context-managers) feature (the `with` keyword) so that any resources we use (in this case open files)
are released when we leave the context (no matter if there is an error, or the processing completes normally).

Now we don't need to close the files explicitly (see [`reverseTextLines1.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines1.py)):

```python
with open("myFile",   "r") as infile, \
     open("yourFile", "w") as outfile:

    for line in infile:
        ... #Process Content in line variable
```


As an aside, in the above examples the `revLine()` function has been improved by adding a call to [`strip()`](https://github.com/alecthegeek/Intro2/blob/8a23321a4ea535a4231c548f672ead418cd87ae8/stdio/reverseTextLines1.beta1.py#L5)
which cleans up the string before further processing -- more information about
`str.strip()` [here](https://docs.python.org/3.7/library/stdtypes.html#str.strip).
Removing the whitespace like this makes the text easier to process in a generic
fashion (no unexpected end of line characters for instance). However we need
to add the end of line character back in the correct location, before we write
the content (e.g. `... + "\n"`).

## Using Standard I/O

Standard I/O solves our final problem, namely file names are hard coded.

The important thing to know is that when a program starts it get access to certain pre defined I/O streams. At a minimum these are:

1. Standard Input (`stdin`, also refereed to as file handle 0)
2. Standard Output (`stdout` or file handle 1)
3. Standard Error (`stderr` file handle 2)

![A program connected to standard I/O](https://raw.githubusercontent.com/alecthegeek/Intro2/trunk/stdio/stdioDiagram.png "A program connected to stdio")

Programs can read from `stdin`, or write to `stdout` or `stderr`, without any extra work. They exist at startup and don't need to be closed.
We just need to take care not to read beyond the end of the input.

In program [`reverseTextLines2.beta1.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines2.beta1.py)
you can see this in action. We are still reading and writing to files, but we
are just accepting that there is input for us to read with `for line in stdin:` and we can write to a file with `stdout.write()`.

So how do these standard file streams get attached the correct files?

By default stdin is attached the keyboard, and both stdout and stderr are attached to the screen. For example:

```shell
$ ./reverseTextLines2.beta1.py 
Hello from Mars and Earth
Earth and Mars from Hello
Goodbye from Venus
Venus from Goodbye
```

Each second line is the output -- `Earth and Mars from Hello` and `Venus from Goodbye` in this example.

**BUT** when you start a program from the terminal you can tell your shell to attach the programs to specific files or devices, rather than
the default keyboard and screen. For example

```shell
./reverseTextLines2.beta1.py < myFile > anotherFile
```

The `<` and `>` characters are [redirection](https://en.wikipedia.org/wiki/Redirection_(computing)) symbols and attach `stdin` to `myFile` and `stdout` to `anotherFile`.
All the previous work of opening and closing the correct files is handled for us and we need no extra code in our
Python script to manage files.

We are not going to discuss the details of using shell redirection here, but you need to be aware of how the `<`, `>` and `|` are used.
You may find this [article](https://www.tldp.org/LDP/abs/html/io-redirection.html) useful if you are using the Bash shell, and this [article](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_redirection) if you use PowerShell.

The stderr file stream is obviouly useful for reporting errors, but it's also useful to report information
that should not be part of normal program output -- such as final summary information and status messages.

[`reverseTextLines2.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines2.py)
shows this in practice -- it keeps track of the number of text records it processes but
does not pollute the output stream with that information. Instead the final record count is printed to stderr.

Note that I have also moved the `revLine()` function out of the main program file and created a new module
called [`textutils`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/textutils.py). This makes the main
program file simpler and allows `revLine()` to be used in multiple programs without repeating the code each time.

## Where do print() and input() fit in the world of standard I/O?

In [example 0](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines0.py) `input()` and `print()` were used for the keyboard and screen? Can, and should, we still use them?

The answer is yes, but use with some care.

1. Using `print()` works well, but does behave slightly differently to `stdout.write()`, so read the fine [manual](https://docs.python.org/3/library/functions.html#print)

2. Using `input()` can be awkward:

    a. `input()`  provides an optional prompt, which does not make sense unless you know your program is interactive
    (you can detect this by checking the value of `sys.stdout.isatty()`)

    b. It only reads a single line of input and does not have an iterator

    c. Detecting eof is not as elegant (zero length input is the eof marker)

See example [`reverseTextLines3.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines3.py)
for the details of using `input()` in this fashion.

Generally it only makes sense to use `input()` when you are sure your program will only ever read input typed by a user.

So the final example [`reverseTextLines3.1.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/reverseTextLines3.1.py)
shows all these concepts in one place.

## So what use cases does standard I/O support?

Using standard I/O we can now do a couple of things

1. Create a general tool that can be used with other tools in a pipeline
(see the example script [`aPipeline`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/aPipeline))

    ![Using a pipeline](https://raw.githubusercontent.com/alecthegeek/Intro2/trunk/stdio/pipesDiagram.png "Using a pipeline")

2. Use standard I/O as a simple "API" that allows other programs to make use of our code as a module
(see this example web application [`APIviaSTDIO.py`](https://github.com/alecthegeek/Intro2/blob/trunk/stdio/APIviaSTDIO.py))

    ![Using Standard I/O as an API](https://raw.githubusercontent.com/alecthegeek/Intro2/trunk/stdio/APIcalls.png "Using stdio as an API mechanism")

In addition standard I/O makes it easier for the developer with simple needs -- no need to manage file resources because it's all done for you
before your code starts

## What's next?

There are other handy ways to get information into a program using command line switches and environnement values. We can also return error codes when
our program finishes. But these are stories for another day...

<a name="parameters">1</a>: More about using command line arguments in a future post.
