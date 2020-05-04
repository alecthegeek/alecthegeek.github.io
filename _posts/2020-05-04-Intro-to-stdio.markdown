# Beginners Introduction to using Standard I/O

## Introduction

If you are a novice programmer sometimes you hear phrases like "read from standard input"
or read things similar "write to stdout",
which is very confusing when you first see it.

This article aims to show you what Standard I/O means and how to do it.

We are going to use Python 3 and the Bash shell -- but standard I/O works identically
in PowerShell and the concepts are the same in other programing languages.

However, because I am using Python, we are going to
see some Python specific features that help make file I/O more pleasant to use. The examples
try to be as simple as possible and easy to understand, but also complete and correct.
You will find it useful to look at the source code at the same time as reading this explanation
and you can find the example code [here](https://github.com/alecthegeek/Intro2/tree/master/stdio)

Note that standard I/O is generally used in text based programs, that we run
from the terminal for instance. Some GUI programs also use standard I/O, but it is less
common.

## What advantages does standard I/O give us?

1. Because standard I/O is set up for us by the calling program (for example the Bash shell),
it makes life simpler for the developer

2. It gives programs a standard way to communicate with each other.

## Reading from the keyboard and writing to the screen

Let's go back to basics...

The program [`reverseTextLines0.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines0.py)
uses the commonly used `input()` and `print()` functions
to: read a line of text from the keyboard; and then write a line of text back to the screen. If you have written
a couple of simple Python programs you should have already seen these functions.

In between the `input()` and `print()` a local function (called `revLine()`) reverses the words in the line.

If we run it we can type in a phrase and see some output:

```
$ ./reverseTextLines0.py 
Please type in line of text : Hello from Earth and Moon
Moon and Earth from Hello
```

However this program has a couple of problems:

1. It only handles a single line of input text

2. It's default behaviour reads from the keyboard and writes to the screen, which can be problem if we have a file of input or
need to save the output

Let's solve those problems

## Reading and writing files

The programs `reverseTextLines1.*.py` use a file based approach instead.

Look at [`reverseTextLines1.beta1.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines1.beta1.py):

1. It opens two files, one for input and one for output (`myFile` and `yourFile`))
2. It uses `while` loop to read each line in turn so that we can process as much input as needed
3. After processing all the input it cleans up by closing the files. Note we should clear up even if the program terminates with an error
4. During the file reading the program has to check to see if it's reached the end of the input (end of file => `eof`)

So that fixed the previous problems, but this solution is still hard work.

1. The file names are hard coded, so we either have to make sure we always use the same file names, or change
the source code each time to use the relevant file names
2. It's hard to make this part of something bigger because it's not a very general solution
3. We have to do a fair bit of housekeeping (opening and closing files, checking for end of file)

Python has some additional sugar to help with problem number 3. If you look at
[`reverseTextLines1.beta2.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines1.beta2.py) we can use the file
iterator to manage the reading of content and handle the eof checking:

```python
for line in infile:
```

Python also has a context feature (using the `with` keyword) so that any resources we use (in this case open files)
are released when we leave the context (no matter if there is an error, or the processing completely normally).
Now we don't need to close the files explicitly (see [`reverseTextLines1.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines1.py):

```python
with open("myFile",   "r") as infile, \
     open("yourFile", "w") as outfile:

    for line in infile:
```

But the file names are still hard coded and it's not generally useful 😒.

As an aside, in the above examples the `revLine()` function has been improved by adding a call to
[`strip()`](https://github.com/alecthegeek/Intro2/blob/8a23321a4ea535a4231c548f672ead418cd87ae8/stdio/reverseTextLines1.beta1.py#L5)
which cleans up the string before further processing -- more information about
`str.strip()` [here](https://docs.python.org/3.7/library/stdtypes.html#str.strip)

## Using Standard I/O

Standard I/O solves problem number 3.

When a program starts it get access to certain pre defined I/O streams. At a minimum these are:

1. Standard Input (`stdin`)
2. Standard Output (`stdout`)
3. Standard Error (`stderr`)

![A program connected to standard I/O](https://raw.githubusercontent.com/alecthegeek/Intro2/master/stdio/stdioDiagram.png "A program connected to stdio")

They might be called something different on your system, and you might get a few more for free,
but these are the ones we are going to work with here and Python follows this convention on every system.

Programs can read from `stdin`, or write to `stdout` or `stderr`, without any extra work. They exist at startup and don't need to be closed.
We just need to take care not to read of the end of the input on `stdin`.

In program [`reverseTextLines2.beta1.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines1.beta1.py)
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

Each second line is the output -- `Earth and Mars from Hello` and `Venus from Goodbye`

**BUT** when you start a program from the terminal you can tell your shell to attach the programs to specific files or devices, rather than
the default keyboard and screen. For example

```shell
./reverseTextLines2.beta1.py < myFile > anotherFile
```

The `<` and `>` characters are redirection symbols and attach `stdin` to `myFile` and `stdout` to `anotherFile`.
All the previous work of opening and closing the correct files is handled for us and we need no extra code in our
Python script to manage files.


We are not going to discuss shell redirection here, but you need to be aware of how the `<`, `>` and `|` are used.
You may find this [article](https://www.tldp.org/LDP/abs/html/io-redirection.html) useful.

The stderr file stream is obviouly useful for reporting errors, but it's also useful to report information
that should not be part of normal program output -- such as final summary information and status messages.

[`reverseTextLines2.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines2.py)
shows this in practice -- it keeps track of the number of text records it processes but
does not pollute the output stream with that information. Instead the final record count is printed to stdout.

Again as an aside, I have taken the `revLine()` function out of the main program file and created a new module
called [`textutils`](https://github.com/alecthegeek/Intro2/blob/master/stdio/textutils.py). This makes the main
program file simpler and means I can use the `revLine()` in multiple programs without repeating the code.

## Where do `print()` and `input()` fit in the world of standard I/O?

In example 0 `input()` and `print()` were used for the keyboard and screen? Can and should we still use them?

The answer is yes, but use with some care.

1. Using `print()` works well, but does behave slightly differently to `stdout.write()`, so consult the fine [manual](https://docs.python.org/3/library/functions.html#print)

2. Using `input()` can be awkward:

    a. `input()`  provides an optional prompt, which does not make sense unless you know you program is interactive
    (you can detect this by checking the value of `sys.stdout.isatty()`)

    b. It only reads a single line of input and does not have an iterator

    c. Detecting eof is a little not as elegant (zero length input is the eof marker)

See example [`reverseTextLines3.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines3.py)
for the details of using `input()` in this fashion.

It makes sense to use input when the program will only read an exact number of lines of input and ideally of if it detects is it reading
from a terminal. Example [`reverseTextLines3.1.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/reverseTextLines3.1.py)
is a reasonable compromise.

## So what use cases does standard I/O?

Using standard I/O we can now do a couple of things

1. Create a general tool that can be used with other tools in a pipeline (see the example script `aPipeline`)

![Using a pipeline](https://raw.githubusercontent.com/alecthegeek/Intro2/master/stdio/pipesDiagram.png "Using a pipeline")

2. Use standard I/O as a simple "API" that allows other programs to make use of our code as a module
(see the web application [`APIviaSTDIO.py`](https://github.com/alecthegeek/Intro2/blob/master/stdio/APIviaSTDIO.py) )

![Using Standard I/O as an API](https://raw.githubusercontent.com/alecthegeek/Intro2/master/stdio/APIcalls.png "Using stdio as an API mechanism")

In addition standard I/O makes it easier for the developer with simple needs -- no need to manage file resources because it's all done for you
before your code starts

## What's next?

There are other handy ways to get information into a program using command line switches and environnement values. We can also return error codes when
our program finishes. But these are stories for another day...