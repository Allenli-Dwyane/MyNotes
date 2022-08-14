# Notes on File System Security

This note is dedicated to papers related to file system security (verification not included).
## Paper List

1. [Finding Crash-Consistency Bugs with Bounded Black-Box Crash Testing](https://www.usenix.org/conference/osdi18/presentation/mohan) OSDI'18
2. [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662) SOSP'19
3. [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422)
4. [RAZZER: Finding Kernel Race Bugs through Fuzzing](https://ieeexplore.ieee.org/abstract/document/8835326/) IEEE S&P'19
5. [Unleashing Use-Before-Initialization Vulnerabilities in the Linux Kernel Using Targeted Stack Spraying.](https://www.ndss-symposium.org/ndss2017/ndss-2017-programme/unleashing-use-initialization-vulnerabilities-linux-kernel-using-targeted-stack-spraying/) NDSS'17
6. [A Study of Linux File System Evolution](https://www.usenix.org/conference/fast13/technical-sessions/presentation/lu) FAST'13
7. [Witcher: Systematic Crash Consistency Testing for Non-Volatile Memory Key-Value Stores](https://www3.cs.stonybrook.edu/~dongyoon/papers/SOSP-21-Witcher.pdf) SOSP'21
8. [Hippocrates: Healing Persistent Memory Bugs Without Doing Any Harm Extended Abstract](https://dl.acm.org/doi/abs/10.1145/3445814.3446694) ASPLOS'21

## [Finding Crash-Consistency Bugs with Bounded Black-Box Crash Testing](https://www.usenix.org/conference/osdi18/presentation/mohan)

Aha! Finally finished reading this paper! It seems pretty fun and I shall read more papers to get some relevant knowledge.

As is clearly demonstrated in the title, this paper focuses on finding crash-consistency bugs. Next...Let's see how this paper works.

### Some Background and Case Study

The first question I came up with when reading this paper (considering I was new in this area, or should I say I am still a newbie after reading this paper..) was "what on earth are *crash-consistency bugs*??". And the second question was "why do crash-consistency bugs matter so much?". Carrying these two questions with me, I started my journey of reading this paper as a total newbie (maybe *dumbass* is more appropriate lol..).

According to the definition in this paper, "A file system is crash-consistent if a number of invariants about the file-system state hold after a crash due to power loss or a kernel panic." Hmmm...good definition! In my own words, which are much more vulgar, crash-consistency means that when an unexpected event that completely shuts off the computer occurs, such as sudden power loss or kernel panic, we do not lose or mess up data (file data or metadata) or files after recovery.

In this paper, authors put a more strict yet pretty reasonable limit on defining crash-consistency: only recovering data or files that **have already been persisted to storage before** counts for crash-consistency. This strict definition rules out possible situations where users just write some data or files to memory but do not persist them to storage, such as when I write this note on VsCode but do not explicitly save the file. If a sudden power cut makes all these un-persisted files and data disappear, then it is considered as the fault of the careless user, not the file system (file system: thank god I do not need to cover these idiots' ass!). It is a pretty reasonable definition, right?

Wait...This is the definition of crash-consistency, not the bugs! Well, crash-consistency bugs refer to those bugs that make the file system violate the crash-consistency property, you stupid! All right, now we've got the definition of crash-consistency bugs, the next thing to figure out is: what causes the crash-consistency bugs to occur? The author gives a clear answer: The root of crash consistency bugs is the fact that most file-system operations only *modify in-memory state*. Here are some examples:

* Modifications to metadata structures are usually accumulated in memory and written to storage via fsync() or a background thread. Developers might forget to update some fields of the data structure or they might improperly order the data and metadata when persisting it.

The authors did a great study on the crash-consistency bugs reported over the last 5 years on Linux file systems. They found a few important points:

1. crash-consistency bugs are hard too find (of course they are! Few people would write a file system, mount it and keep crashing the system to see if something ever went wrong. If you do so, I might think you are out of your mind...there are so many other bugs out there! Not to mention that even regular bugs are pretty hard to find, even with many debugging tools.), thus systematic testing is necessary.

2. **small workloads can reveal bugs on an empty file system.** This insight is super super important! It provides a basis for the authors' work (i.e., B3).

3. **All reported bugs involved a crash right after a persistence point: a call to fsync(), fdatasync(), or the global sync command.** This is also super important! This insight points out a way for testing crash-consistency bugs: perform file-system operations, **sync() or similar calls**, crash(!!) and check states after recovery.

### B3: An Approach to Crash Testing in Bounded Black-Box (Not Actual Implementation!!)

## [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662)

Read this paper a while ago. Need to re-read it! Fuzzing is kind of like the technique used in B3.

## [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422)

Woo..Finished reading this paper. Pretty interesting but there are some ambiguous points in this paper. Worth some improvement.