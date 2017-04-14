# *CpuId* — Ask your CPU what it can do for you.

_Status: Experimental._

[![Build Status](https://travis-ci.org/m-j-w/CpuId.jl.svg?branch=master)](https://travis-ci.org/m-j-w/CpuId.jl)
[![Build Status](https://ci.appveyor.com/api/projects/status/q34wl2a441dy87gy?svg=true)](https://ci.appveyor.com/project/m-j-w/cpuid-jl)
[![codecov](https://codecov.io/gh/m-j-w/CpuId.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/m-j-w/CpuId.jl)

Expected to work in general on Julia 0.5 and 0.6, on Linux, Mac and Windows
with Intel compatible CPUs (which include e.g. AMD).


## Motivation

Besides the obvious reason to gather information for diagnostics, the CPU
provides valuable information when aiming at increasing the efficiency of code.
Such usecases could be to tailor the size of working sets of data according to
the available cache sizes, to detect when the code is executed in a virtual
machine (hypervisor), or to determine the size of the largest SIMD registers
available.  This information is obtained by directly querying the CPU through
the `cpuid` assembly instruction.  A comprehensive overview of the `cpuid`
instruction is found at [sandpile.org](http://www.sandpile.org/x86/cpuid.htm).

Same information may of course be collected from various sources from Julia
itself or from the operating system, e.g. on Linux from `/proc/cpuinfo`.
However, the `cpuid` instruction should be perfectly portable and efficient.


## Installation and Usage

*CpuId* is a registered Julia package; Clone the repository from the REPL:

    Julia> Pkg.add("CpuId")

Or, if you're keen to get some intermediate updates, clone from GitHub master
branch:

    Julia> Pkg.clone("https://github.com/m-j-w/CpuId.jl")


## Features

See the diagnostic summary by typing

    julia> using CpuId
    julia> cpuinfo()

       Cpuid Property   Value
      ╾───────────────╌─────────────────────────────────────────────────────╼
       Brand            Intel(R) Xeon(R) CPU E3-1225 v5 @ 3.30GHz
       Vendor           Intel
       Model            Dict(:Family=>6,:Stepping=>3,:CpuType=>0,:Model=>94)
       Clock Freq.      3300 / 3700 MHz (base/max)
                        100 MHz bus frequency
       Address Size     48 bits virtual, 39 bits physical
       SIMD             max. vector size: 32 bytes = 256 bits
       Data cache       level 1:3 : (32, 256, 8192) kbytes
                        64 byte cache line size
       Hypervised       No


This initial release covers a selection of basic functionality:

 - `cpuinfo()` generates the summary shown above (markdown string).
 - `cpubrand()`, `cpumodel()`, `cpuvendor()` allow the identification of the
     CPU.
 - `cpucycle()` and `cpucycle_id()` let you directly get the CPU's time stamp
     counter, which is increased for every CPU clock cycle. Lowest overhead for
     benchmarking, though, technically, this uses the `rdtsc` and `rdtscp`
     instructions rather than `cpuid`.
 - `address_size()` and `physical_address_size()` return the number of bits used
     in pointers.  Useful when stealing a few bits from a pointer.
 - `cachelinesize()` gives the size in bytes of one cache line, which is
     typically 64 bytes.
 - `cachesize()` returns a tuple with the sizes of the data caches in bytes.
 - `cpu_base_frequency()`, `cpu_max_frequency()`, `cpu_bus_frequency()` give -
     if supported by the CPU, the base, maximum and bus clock frequencies.
     Use `has_cpu_frequencies()` to check whether this property is supported.
 - `hypervised()` returns true when the CPU indicates that a hypervisor is
     running the operating system, aka a virtual machine.  In that case,
     `hvvendor()` may be invoked to get the, well, hypervisor vendor.
 - `simdbits()` and `simdbytes()` return the size of the largest SIMD register
     available on the executing CPU.


## Limitations

The behaviour on non-Intel CPUs is currently unknown; though technically a crash
of Julia should be expected, theoretically, a rather large list of CPUs support
the `cpuid` instruction. Tip: Just try it and report back.

There are plenty of different CPUs, and in particular the `cpuid` instruction
has numerous corner cases, which this package does not address, yet.  In systems
having multiple processor packets (independent sockets holding a processor), the
`cpuid` instruction may give only information with respect to the current
physical and logical core that is executing the program code.

#### Specific limitations

- Why aren't all infos available that are seen e.g. in `/proc/cpuinfo`?
    Many of those features, flags and properties reside in the so called machine
    specific registers (MSR), which are only accessible to priviledged programs
    running in the so called *Ring0*, such as the Linux kernel itself. Thus,
    short answer: We don't get it...

- My hypervisor is not detected!
    Yeah, well, hypervisor vendors are free to provide the `cpuid` information
    by intercepting calls to that instruction.  Not all vendors comply, and some
    even permit the user to change what is reported.  A non-reporting example
    is said to be VirtualBox.

- But `rdtsc`/`rdtscp` are not `cpuid`!
    True, but who cares. Both are valuable when diagnosing performance issues
    and fit the *absolutely minimal overhead* pattern by directly talking to the
    CPU.


## Terms of usage

This Julia package *CpuId* is published as open source and licensed under the
[MIT "Expat" License](./LICENSE.md).


**Contributions welcome!**

You're welcome to report successful usage or issues on GitHub, and to open pull
requests to extend the current functionality.

