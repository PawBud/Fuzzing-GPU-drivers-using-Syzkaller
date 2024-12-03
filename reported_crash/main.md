This directory contains other interesting crashes that were completely reproduced by syzkaller while fuzzing the nouveau drivers.

The contents of each of the crash directories are:-

* report* - This is the complete report that is returned by syzkaller.
* repro.c - the c code that causes the warning/error. Note that Syzkaller __attempts__ to convert the Syz reproducer (which uses it's own DSL) into C code. Hence the C code might not produce the warning/error <br> that is highlighted in the description.
