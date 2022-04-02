# Title
Optimized event sending, input file I/O and checksum calculations

# Summary

The goal of the changes to the Hackathon simulation were to speed up the program.
The performance bottlenecks were determined using profiling tools like 'perf report' and
through the generation of flamegraphs. As a consequence, three major bottlenecks were identified and addressed: (a) the processing time required to read input files; (b) the time wasted bouncing unprocessed ideas through the student threads; (c) the frequency of global checksum updates alongside the decoding costs. These bottlenecks were addressed and the subsequent changes where made.

# Technical Details

## A: Input File I/0
The **Package::run(...)** and **IdeaGenerator::run(...)** method were processing their respective input files every time they were sending an event. Therefore changes where made to read the input file once per thread into a struct property. Then queries can be made quicker to this data struct than rereading input files. 

## B: Bouncing Unprocessed Ideas
The **Student::run(...)** would waste time recieving and sending **idea::Idea** events when the thread was in the process of accumulating packages to build an idea. Therefore a global atomic **idea::Idea** vector was created as a worklist for the threads. Therefore, a thread would not recieve an idea until it was ready to process one. This reduced the overhead of receiving and sending ideas.

## C: Checksum Updating
The acquisition of mutexes to update the global idea and package checksums (i.e. **checksum::Checksum**) was a problem due to its frequency. Since the calculation of the checksums adhered to the mathematical associative and commutative principles, each thread could update its own local checksum and do a global update at the end. This was updated for **Student::run(...)**, **Package::run(...)** and **IdeaGenerator::run(...)**.

Lastly, the **checksum::Checksum** data structure was originally designed to be an extension of the **String** class. This meant that checksums were stored as strings and needed to be decoded into byte arrays every **Checksum::update(...)**. Therefore the structure was modified to represented the internal checksum as a byte array to circumnavigate the decoding.

# Testing for correctness
Correctness was ensured by asserting that the produced packages and idea suggession checksums matched with the ones consumed by the student threads. Furthermore, **valgrind** was used to ensure that all heap blocks were freed.
![](images/checksums.png)

# Testing for performance.
Flame graphs were generated to visually observer the execution time distribution at varius stages. Alongside the
**perf** report's these observations informed the optimization directoins. Hyperfine was used to benchmark the models execution speed via the command `hyperfine -i "target/release/lab4 -w 5 -m 100` to run the benchmark a minimum of 100 times with 5 warmup executions on the **ecetesla1** server. The original version of the program had a hyperfine performance of:
![](images/dev_branch.png)
The fully optimized program had a hyperfine performance of:
![](images/kk_v3.png)
That is an improvment of +20x. Here is the original flame graph:
