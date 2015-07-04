log2mem
=======

log2mem is a high-performance diagnostic C/C++ logger designed to serve as a debugging
tool when dealing with bugs and race conditions in multi-threaded or low latency
code.

[describe the nautre of the application more?]

ie, microsecond level timings multi threaded code introduces a new class of
subtles bugs, as threads race to both use some data source.  Low latency code is
general term of tasks which have to operate on the order of tens of
microseconds.


Most developers are familer with the two basic approachs to debugging: execute
the code in a debugger; or litter the source with increasingly targeted and
detailed printf/std::cout statements.  However for multithreaded or low latency
code, both approaches often fail to work.


The problem is time. Whether or not a bug gets triggered can be dependent on not
just a program's inputs but also on its run-time speed.  Stepping through the
code with a debugger (such as gdb) will significantly reduce execution speed.
So if a bug only arises when its threads interact at normal speed, then slowing
down the program typically supresses the bug.  Similarly printf/std::cout
statements have a cost; strings are constructed and written to an output stream,
and these too impede program speed. The developer is left with the frustrating
and not uncommon condumdrun that the act of debuging causes the problem to
temporarily disappear. [better word for condundrum?]

The excellent blog Preshing on Programming [] also identifies this problem of
debugging multi-threaded code, and infact log2mem is based on a solution
described there.

[TODO: need the reference to the blog post

[Lightweight In-Memory Logging, 22 May 2012]

Additionally, for some class of programs typically characterised as low latency
or real-time, proper program execution is also dependent on it running at normal
speed (eg software for robot contro or processing live finanical market data).
So any runtime slow introduced by debugging techniques might not even allow for
correct functionality, let alone tickling the bug into appearence.

Use of application specific asynchronous loggers is sometimes an option. However
they are often not designed or implemented with low latency in mind (e.g. they
typically build strings), and so again their usage can adveresly impact program
speed.  Additionally such loggers might not be readily accesible at the code
site being examined.

So what does log2mem do?

log2mem is a tool to use for these situations when a debugger and printf can not
be used.  It offers the rough equivalent of very high speed printf; this can
then be used to instrument the code without comprimsing runtime performance. The
program can be run at full speed, the bug can arise, and its footprints can be
logged.

log2mem acheives high performance by writing log statements to a circular buffer
in shared memory.  So whereas printf involves string construction and writing to
some kind of stream at the time of the call, log2mem instead writes the raw data
values direct to memory. A separate tool, log2mem-util, is later used to
inspect the shared memory and convert the raw logging data to human readable
strings. log2mem additionally makes use of lock-free and lock-less techniques to
ensure optimal performance.  Additionally, log2mem is easy to integrate in
existing C/C++ code.

[maybe illustrate this?  ie, compare printf and xlog_write, shwoing what each does]

[make a statement about typical performance ... but where to place?]

On a 2.5 GHz Intel Core i5, logging a simple message including timestamp, thread
id and a couple of values, takes under 150 nanoseconds.

[say is currently limited to Linux based systems]

[would be nice to have a diagram]

Hello World
===========

Below is a basic "hello world" example of using log2mem.  Integration into your
code requires that the header file log2mem.h is included and the library
liblog2mem added to the link line.

    #include <iostream>
    #include <log2mem/log2mem.h>

    int main(int, char**)
    {
        std::cout << "hello world" << std::endl;

        // obtain a logging row, and write a simple message & value
        void * rowptr = log2mem_get_row(MEMLOGFL);
        log2mem_append_str2(rowptr, "hello world");
        log2mem_append_int(rowptr, 100);

        // print location of the memmap
        std::cout << "log2mem memmap: " << log2mem_filename() << std::endl;
        return 0;
    }

This could be complied with the following kind of command, which needs to
specify the location where log2mem has been installed.

    g++ -I/tmp/log2mem/include  hello.cc  -o hello  -L/tmp/log2mem/lib -llog2mem

This assumes that, when you built log2mem, you installed in into
/tmp/log2mem.  If you choose a different install location, then the compile
command should be changed appropriately.


[ullustrate the program output, and operation of the utilo ?]

[other uses]
* timing
* counters
* interaction


Extra Features
==============

In addition to diagnositc logging, log2mem offers some extra feature to help the
process of debugging.  Well, in
additiojs to the logger, log2meme has some other feature to help debugging.
Theyse bcan be of use for all lopgging tasks, not just ultra high speed rint .

Variables & external interaction
--------------------------------

One feature is variables.  Variable can be defined in the memmap whic can be
read nadn written, using log2mem functions.  This nice this is that these can
also be read and wirtten to using the util tool; this give a quite methods to
easily integrate with a program while it is running.

Counters
--------


[say about extra things log2mem can do]
