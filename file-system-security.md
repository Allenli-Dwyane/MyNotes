# Notes on File System Security

This note is dedicated to papers related to file system security (verification not included).
## Paper List

1. [Finding Crash-Consistency Bugs with Bounded Black-Box Crash Testing](https://www.usenix.org/conference/osdi18/presentation/mohan) OSDI'18
2. [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662) SOSP'19
3. [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422) SOSP'15
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

Given the two key insights as thickened above, the authours proposed an approach named B3, which stands for bounded black-box. The authors described B3 as follows:

```
B3 is a black-box testing approach built upon the insight that most reported crash-consistency bugs can be found by systematically testing small sequences of file-system operations on a new file system. 
B3 exercises the file system through its system-call API, and observes the file-system behavior via read and write IO.
```

One of the most important advantages of B3 is that **B3 does not require annotating or modifying file-system source codes**. Well, this is great! As for what I have learned from reading all these bug-finding papers, which exploited techniques as fuzzing or so, these techniques, especially fuzzing, need to modify the file-system source code to find bugs. The novelty of B3 makes it easier to check file system bugs since it just generates file system operations to test the crash-consistency of file systems. Awesome!

Next, let's what B3 mainly does. How does B3 actually works? The above definition does not describe clearly its routine. Here's the answer: B3 just generates sequences of file system operations, such as read() and write(), within a bounded space (Aha? What is this?). Then it tests all these generated sequences (i.e., executes the sequences) and crashes the system each time a persistence point is hit (Hmm..?). At last, B3 recovers from the crash and checks whether the file system recovers back to the right state.

We have successfully gotten the main idea of this paper! But there are still some ambiguity in the B3 process. We need to figure them out.

* crash points: **B3 only simulates crashes after each persistence point in the workload**. That is to say, only after the user intentionally performs persistence operations (e.g., sync() or other similar operations), B3 treats it as point that deserves to be crashed and tested. This is based on our strict definition on crash-consistency, rememeber? If you don't, go back and read the definition again carefully!

* bounds: B3 bounds the file system operations generated in a workload in three ways. 
  
    1. number of operations
    2. files and directories in workload
    3. data operations
    4. initial file-system state

### Implementation (CrashMonkey and Ace)

To achieve the B3 approach, the authors built two tools. One is Ace (full name is Automatic Crash Explorer), which is used to **exhaustively** generates workloads within the given bounds. The other is CrashMonkey, which is used for **record-and-replay techniques to simulate a crash in the middle of the workload and test if the file system recovers to a correct state after the crash**. Ok, we will examine these two tools in detail next.

* CrashMonkey employs the key technique called **record-and-replay**, which is to record all the information from the given workload and replay this workload until crash. CrashMonkey achieves so by using three phases:

    1. **Phase One**:profiles the workload by collecting information about all file-system operations and I/O requests made during the workload.
    2. **Phase Two**:replays I/O requests until a persistence point to create a **crash state**. This **crash state** is important!! It represents the state of storage if the system had crashed after a persistence operation completed. Then CrashMonkey mounts the file system again and recovers it. Another thing CrashMonkey does at this phase is that it captures a reference file system image named oracle by safely unmounting it so the file system completes any pending operations or checkpointing.
    3. **Still Phase Two!!** Well, the phase two of CrashMonkey might seem a little confusing, but it actually is a pretty simple one: the **crash state** is like the answer written by the students in an exam, and the **oracle** is the right answer given by the teacher. What we need to do is compare the **crash state** to the right answer. That's what Phase Three is for.
    4. **Phase Three**: CRASHMONKEY’s AutoChecker tests for correctness by comparing the persisted files and directories in the oracle with the crash state after recovery.

* Ace uses four phases to exhaustively generate workloads within the given bounds:
  
  1. First, Ace is given the bound of X operations in a workload. For instance, X equals 2 means only two file-system operations are allowed in this worload. In this phase, what Ace does is select X file-system operations to make a *skeleton*.
  2. Next, Ace needs to select parameters for the file-system operations.
  3. Then, Ace adds persistence points to the skeleton (i.e., sync() or other similar operations).
  4. Last, Ace adds dependencies. Since every file/directory needs to exist before being taken in file-system operations, Ace must create these files/directories beforehand (e.g., mkdir() before calling a directory).

### Results and My Thoughts (Most Important Part!)

Well, I won't go into details of the experimental results of B3. The results show that this approach is pretty efficient in finding bugs. But it generally takes pretty much to find all these bugs: the authors needed to deploy CrashMonkey and Ace in cloud to compute the results (which costs money and time).

And here comes the important part! My thoughts! 

The authors have already demonstrated the limitations of B3, which I will not repeat in this note. My thoughts will go beyond these. The strategy exploited in B3 is rather straight forward, but such straightforward strategy comes from the careful study into the reported crash-consistency bugs. I think this is pretty important to my future researches: you need to first observe some commonalities, and based on which propose a method that solves the problem. And one more thing: the simpler yet more efficient method, the better!

Also, B3's techniques are similar to fuzzing because both need to generate lots of inputs to try out the bugs. Perhaps here's where I can dive into?

## [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422)

Woo..Finished reading this paper. Pretty interesting but there are some ambiguous points in this paper. Worth some improvement.

### Semantic Bugs

This paper and the following paper (i.e., HYDRA) both stress on finding semantic bugs. So I guess I should first conclude what semantic bugs refer to and why are they attracting so much attention.

**Semantic bugs** mostly refer to those bugs which violate high-level rules of invariants. Hmmm..this seems like a pretty ambiguous definition. It is due to that semantic bugs come in various forms including violations of agreed property (e.g., crash-consistency), non-conformance to specifications and etc. As we can tell from the types and definition of semantic bugs, they are different from the memory bugs as use-after-free in that they do not cause obvious consequences! For instance, violations of POSIX standard may be due to the ambiguous definition of POSIX, thus different file system developers may interpret these definitions in different ways.

With 


## [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662)

Read this paper a while ago. Need to re-read it! Fuzzing is kind of like the technique used in B3.

