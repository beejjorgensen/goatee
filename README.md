Goatee
======

A simple file includer and macroish file processor

        )_)
     ___|oo)   File inclusion and macroish processing--
    '|  |\_|                                           --for goats!
     |||| #
     ````

Description
-----------

Goatee (Goat Expression Evaluator) processes an input file, which might
include other files, or execute Python code within special delimiters,
and produces an output file.

The original intended purpose was to be used in HTML/JS/CSS build
pipelines, where the CSS or HTML might need to be processed with file
inclusion or variable substitution.

Goatee is small, simple, and standalone, to be easily included with
other build-related sources, e.g. Makefiles.


Features
--------

* Includes files from within other files
* Stores and prints variables, macro style
* Executes arbitrary Python code
* Endorsed by goats on spools


Usage
-----

Anything between the goatee code delimiter marks `[[` and `]]` (which
currently must be on the same line) will be executed in the Python
interpreter (spaces around the expression next to the delimiter marks
are stripped):

    [[x = 10]]
    Hey, I think x is [[prints(x)]]!

would output:

    Hey, I think x is 10!

An example with file inclusion:

*globals.src.css*:
     
    [[x = 10]]
    [[y = 20]]

*main.src.css*:

    [[include('globals.src.css')]]

    #goatlogo {
        top: [[prints(y)]]px;
        left: [[prints(x)]]px;
    }

At this point, running
  
    goatee -o main.css main.src.css

will produce a main.css file with this content:

    #goatlogo {
        top: 20px;
        left: 10px;
    }

Additionally, an optional "flags" field is present in the expression
match:

    [[expression]flags]

The supported flag is currently:

**d** - discard everything after the closing goatee ']' until the end of
the line, e.g.:

    [[prints('bonjour!')]d]  prints 'hello' in French

would emit:

    bonjour!

and discard the remainder of the line (including the newline).

You can also import modules:

    [[import math]]

    The cosine of almost-pi is [[ps(math.cos(3.14))]].

Or do whatever else you can cram into one line of Python:

    the numbers 1-10 are [[for i in range(10): ps(" %s" % (i+1))]].


Built-in Functions
------------------

In addition to arbitrary code execution, Goatee defines the following
functions:

-------------------------------------------------------------------------
    include(filename, evaluate=True, typeoverride=None)

This is the workhorse function that includes and processes subfiles.
       
If evaluate is set to False, the file is included, but no code between
the delimiters is processed.

If typeoverride is set to a mime type, that mime type is used while
processing the file, otherwise it is autodetected.


-------------------------------------------------------------------------
    prints(s)
    ps(s)

Prints the string representation (as returned by str()) of parameter s
to the current output file (ps is shorthand for prints).  


-------------------------------------------------------------------------
    output(b)

If true, processed input data should be printed to output. False if the
output should be quiet. Also silences calls to `prints()`. Subsequent
Python commands are processed normally in any case.


-------------------------------------------------------------------------
    source(filename)

Executes the Python source in filename in the global namespace. This can
be useful for defining more Python functions to have at your disposal.


-------------------------------------------------------------------------
    env(var)

Returns the value of the given environment variable, or an empty string
if not defined.


-------------------------------------------------------------------------
    envdef(var)

Returns true if the environment variable is defined.


-------------------------------------------------------------------------
    ifdef(var)
    ifdef(var, val)
    ifndef(var)
    ifndef(var, val)
    endif()

Make subsequent output conditional on the existence of (or value of, if
val is also specified) of an environment variable. `ifndef` negates the
sense of the test. `endif()` makes subsequent output unconditional.

Example:

    [[ ifdef('PROFILE', 'DEBUG') ]]
    <script src="site.js"></script>
    [[ endif() ]]

    [[ ifdef('PROFILE', 'PRODUCTION') ]]
    <script src="site-min.js"></script>
    [[ endif() ]]

CHEESINESS WARNING: all Python commands inside the ifdef/endif block are
unconditionally executed!  (`prints()` is muted, however.)  I realize
this isn't as neat as it could be, but I feel it still beats defiling
the code to support it.

CHEESINESS WARNING: ifdefs don't nest.

Command Line Usage
------------------

    usage: goatee [options] [file]

       -h      --help           this help
       -o file --output=file    send output to file (default stdout)
       -t type --type=type      input file type override
       -p      --preserve-nl    preserve newlines on goatee-only lines
       -w      --warn-comment   add a computer-generated warning comment


**-t** is most useful when piping into goatee (and it cannot determine the
MIME type).

**-p** preserves newlines on lines that consist solely of goatee
substitution.  In normal usage, if a line has no other data except a
trailing \n, it implies the *]d]* flag (see above), so:

    [[x=10]]\n

would produce no output (the \n would be eaten). *-p* causes the
trailing \n to be emitted.


Modification of Code Delimiters
-------------------------------

At the top of the source, the variable `_goatee_patterns` is defined,
which holds replacement regexes organized by MIME type.  The `[[` and
`]]` code delimiters are visible in the 'expr' field.

New file types can be added here, or the delimiters can be modified.

I considered putting this in a conf file, but goatee is only 200 lines,
so screw it.  Make a copy, put it in your build tree, and modify it
there. Just imagine that the entire source is one big conf file. ;)


   
Related
-------

<beej@beej.us>

http://beej.us/


License (MIT)
-------------
Copyright (c) 2013  Brian "Beej Jorgensen" Hall <beej@beej.us>

Permission is hereby granted, free of charge, to any person obtaining a
copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

