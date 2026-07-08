Memory Cache Simulator

A trace-driven cache simulator that lets you feed in real memory access traces (captured with Intel Pin) and see how different cache replacement policies actually perform on them, instead of just reading about them in a textbook.

Built for CS204 (Computer Architecture) at IIT Ropar, under Prof. Neeraj Goel.

Team: Abhijit Kamble (2024CSB1401), Tanish Kumar (2024CSB1161)

How it works

The project is split into two independent stages:


Trace generation - a Pintool (MyPinTool.cpp) hooks into a running program and logs every memory read/write it makes: the instruction pointer, whether it's a load or store, the address, and the access size. Stack accesses are filtered out since they're mostly noise for cache analysis.
Simulation - the simulator (main.cpp) reads the trace and replays it against a configurable set-associative cache, trying out different replacement policies and reporting hits, misses, evictions, and hit rate for each.


Because the two stages are decoupled, you can generate a trace once and run it through every policy without re-instrumenting anything.

Replacement policies


LRU - evicts the block that hasn't been touched the longest
FIFO - evicts whichever block was inserted first, hits don't save you
Random - picks a victim at random, no metadata needed, mostly here as a baseline
Belady's Optimal - looks into the future of the trace and evicts the block that won't be needed for the longest time. Not realizable in real hardware, used as a theoretical upper bound
SRRIP - static re-reference interval prediction, uses a 2-bit counter per block instead of exact timestamps
Hawkeye - tries to predict per-PC whether a block deserves to be cached long-term, based on whether blocks inserted by that PC tend to get hit again before eviction
DRP - a custom policy we wrote. It keeps a small buffer of recently evicted addresses, and if one of them comes back into the cache shortly after being evicted, that was a "regretted" eviction, so the block is marked stable and protected from eviction unless every other block in the set is also stable


Repo layout

MyPinTool.cpp          Pintool that generates the memory trace
main.cpp                Cache simulator, runs all policies on a trace
matrix_mul.cpp          Sample workload (matrix multiplication) used to generate traces
Tests/                  A few smaller test programs used while debugging the pintool
Tracefiles/             Sample trace outputs
Memory cache Simulator Doc.pdf   Write-up covering the architecture and early results
Project Proposal.pdf    Original project proposal

Trace format

Each line in a trace file looks like this:

0x400000 L 0x0000 8
0x400000 L 0x0020 8

That's [instruction pointer] [L/S] [address] [size in bytes]. L is a load, S is a store. The simulator only actually cares about the address field, everything else is there for completeness / future use.

Building and running

1. Generate a trace

You'll need Intel Pin set up first. Then:

bashcd ~/pin_kit/source/tools/MyPinTool
make

Compile a test workload:

bashg++ -O0 matrix_mul.cpp -o matrix_test

Run it under Pin to get a trace:

bash$PIN_ROOT/pin -t obj-intel64/MyPinTool.so -- ./matrix_test

This produces trace.out in the current directory.

2. Run the simulator

main.cpp currently reads from a hardcoded filename (trace6.out), so either rename your trace or edit that line before building.

bashg++ -O0 main.cpp -o simulator
./simulator

Results get printed to the terminal and also written to statistics.out.

Changing cache parameters

Cache size, block size and associativity are set as constants near the top of main.cpp:

cppconst int CACHE_SIZE = 64;
const int BLOCK_SIZE = 8;
const int ASSOCIATIVITY = 2;

Edit and rebuild to try different configurations. Number of sets, offset bits and index bits are all derived from these automatically.
