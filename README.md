# Title
Optimized event sending, input file I/O and checksum calculations

# Summary

The goal of the changes to the Hackathon simulation were to speed up the program.
The performance bottlenecks were determined using profiling tools like 'perf report' and
through the generation of flamegraphs. As a consequence, three major bottlenecks were identified and addressed:\

1. The **Package::run(...)** and **IdeaGenerator::run(...)** method were processing their respective input files every time they were sending and event. Therefore changes where made to read the input file once per thread into a struct property. Then queries can be made quicker to this data struct than rereading input files
2. The **Student::run(...)** would waste time recieving and sending 

# Technical details

Anything interesting or noteworthy for the reviewer about how the work is done (3-5 paragraphs)

# Testing for correctness

Something about how you tested this code for correctness and know it works (about 1 paragraph).

# Testing for performance.

Something about how you tested this code for performance and know it is faster (about 3 paragraphs, referring to the flamegraph as appropriate.)
