# Notes on File System Security

This note is dedicated to papers related to file system security (verification not included).
## Paper List

1. [Finding Crash-Consistency Bugs with Bounded Black-Box Crash Testing](https://www.usenix.org/conference/osdi18/presentation/mohan) OSDI'18
2. [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662) SOSP'19
3. [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422) SOSP'15
4. [A Study of Linux File System Evolution](https://www.usenix.org/conference/fast13/technical-sessions/presentation/lu) FAST'13
5. [Witcher: Systematic Crash Consistency Testing for Non-Volatile Memory Key-Value Stores](https://www3.cs.stonybrook.edu/~dongyoon/papers/SOSP-21-Witcher.pdf) SOSP'21
6. [Hippocrates: Healing Persistent Memory Bugs Without Doing Any Harm Extended Abstract](https://dl.acm.org/doi/abs/10.1145/3445814.3446694) ASPLOS'21
7. [RECIPE: Converting Concurrent DRAM Indexes to Persistent-Memory Indexes](https://dl.acm.org/doi/10.1145/3341301.3359635) SOSP'19
8. [Write-Optimized and High-Performance Hashing Index Scheme for Persistent Memory](https://www.usenix.org/conference/osdi18/presentation/zuo) OSDI'18
9. [PMTest: A Fast and Flexible Testing Framework for Persistent Memory Programs](https://dl.acm.org/doi/10.1145/3297858.3304015) ASPLOS'19
10. [AGAMOTTO: How Persistent is your Persistent Memory Application?](https://www.usenix.org/conference/osdi20/presentation/neal) OSDI'20
11. [Evaluating File System Reliability on Solid State Drives](https://www.usenix.org/conference/atc19/presentation/jaffer) USENIX ATC'19
12. [MOD: Minimally Ordered Durable Datastructures for Persistent Memory](https://dl.acm.org/doi/10.1145/3373376.3378472) ASPLOS'20
13. [Cross-Failure Bug Detection in Persistent Memory Programs](https://dl.acm.org/doi/10.1145/3373376.3378452) ASPLOS'20
14. [DURINN: Adversarial Memory and Thread Interleaving for Detecting Durable Linearizability Bugs](https://www.usenix.org/conference/osdi22/presentation/fu) OSDI'22


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

## [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422) and [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662)

Woo..Finished reading this paper ([Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422)). Pretty interesting but there are some ambiguous points in this paper. Worth some improvement.

Read this paper ([Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662)) a while ago. Need to re-read it! Fuzzing is kind of like the technique used in B3.
### Semantic Bugs

This paper and the following paper (i.e., HYDRA) both stress on finding semantic bugs. So I guess I should first conclude what semantic bugs refer to and why are they attracting so much attention.

**Semantic bugs** mostly refer to those bugs which violate high-level rules of invariants. Hmmm..this seems like a pretty ambiguous definition. It is due to that semantic bugs come in various forms including violations of agreed property (e.g., crash-consistency), non-conformance to specifications and etc. As we can tell from the types and definition of semantic bugs, they are different from the memory bugs as use-after-free in that they do not cause obvious consequences! For instance, violations of POSIX standard may be due to the ambiguous definition of POSIX, thus different file system developers may interpret the definition in different ways, but these bugs are invisible from time to time since they may not cause severe damage.

With all these ambiguity in POSIX, different implementations from one file system to another and invisible consequences, finding semantic bugs are difficult yet important, since such bugs can cause damage that is not obvious to developers. To address such issues, JUXTA is proposed in [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422) and HYDRA is proposed in [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662). I will examine the two tools from the high level in the rest of this note.

### JUXTA in [Cross-checking Semantic Correctness: The Case of Finding File System Bugs](https://dl.acm.org/doi/10.1145/2815400.2815422)

The structure of JUXTA comprises of a sequence of stages, and five checkers based on the stages. Remerber, what JUXTA tries to achieve is a general method that infers latent semantic bugs across different file system implementations without knowing domain-specific knowledge.

* Stage One. In this stage, JUXTA merges each file system's files into one large file. This stage is called *the source code merge stage*. The reason for this combination is that current static anaylyzer can perform inter-procedural anaylysis only within a single file.
  
* Stage Two. In this stage, JUXTA constructs a control-flow graph (CFG) for a function and symbolically explores a CFG from the entry to the end. Also, to prevent the path explosion, JUXTA sets a limit on the maximum inline basic blocks and functions in this stage.

* Stage Three. Since different file systems exhibit different symbols, JUXTA needs to represent symbolic expressions using universally comparable symbols. In this stage, JUXTA uses symbol canonicalization to achieve so.

* Stage Four. JUXTA creates a per-file system path database, which can be easily used to access the extracted organized information.

* Stage Five. This stage is **important** as it provides a basis for all the application checkers implemented on the four stages. Basically, JUXTA uses two statistical models. One is histogram based  comparison, the other is based on entropy. 
    
    1. Histogram-based model. What this model mainly does is that JUXTA first encodes integer ranges into multidimensional histograms for comparison and uses distances among histograms to find deviant behaviors in different file systems. That is to say JUXTA uses histogram as a measurement tool of each file system, and use the average of VFS histogram as the standard one, if any file system deviates from this average too far, it is considered as having semantic bugs.
    2. Entropy-based comparison. This comparsion is used to find deviation in an event. Basically JUXTA calculates the entropy of each VFS interface for an event, and a VFS interface whose corresponding entropy is small (except for zero) can be  onsidered as buggy.

All the applications, such as checkers or file-system specification extracters, are mainly based on the two statistical models above. So that I won't go into details about the applications. The efficacy of the two models are key to the performance of JUXTA!!

### HYDRA in [Finding Semantic Bugs in File Systems with an Extensible Fuzzing Framework](https://dl.acm.org/doi/10.1145/3341301.3359662)

Finally, HYDRA! I love this name lol (Hail Hydra! Just kidding, watched a lot of Marvel movies before). All right, back to track. HYDRA is proposed to be a general framework for finding semantic bugs, including crash inconsistency, POSIX violations and file system-specific logic bugs. The key technique employed in HYDRA is an old friend, fuzzing (hola! Nice to see you again). As described in the article, HYDRA is "a comprehensive framework that complements existing and future bug checkers by providing a set of commonly required components, all tailored to file system fuzzing". Thus, we can see some existing tools used in HYDRA as well as some new faces.  

The process of HYDRA framework can be described below:

1. feeds the seed to to the fuzzer, which generates mutated file system image and syscalls accordingly. Wait a minute, what is the seed? The seed consists of a file system image and a sequence of syscalls.
2. then the generated file system image and syscalls (we call them the test cases hereafter) are sent to a clean library OS-based executor, which mounts the image and executes the syscalls.
3. after execution, a bitmap is generated and the bug-checking dispatcher invokes the necessary run-time checks. And the dispatcher collects the feedback from the checker and merges it with the bitmap into a fuzzing feedback report.
4. the test case is saved if a new coverage is reported or marked as interesting. Otherwise, it is discarded.
5. if a new bugs is found, the test case will be sent for replay and confirmation.

As we can tell from the process, the most important part is in 3, which refers to generating a bitmap and **bug-checking**. Therefore, HYDRA integrates already-existing bug checkers as well as a novel crash-consistency checker (SYMC3) into this part. 

The novel SYSMC3 checker, which is proposed by the authors, "emulates the syscalls to derive a symbolic representation of all allowed post-crash states according to the file system-specific notion of crash consistency and checks whether the recovered image falls into one of the states." Here I just copy the text from the paper to this note because of the laziness of human beings (not my fault, it is the nature)! This text concisely describes the algorithm for crash-consistency bugs. In short, SYMC3 tracks the history of changes of each c3_inode (which records the historage until persisted to storage) also until they are persisted. SYMC3 also take snapshots of memory state and disk state, generates allowed states and checks whether recovery file system state falls into the allowed ones. If not, a crash-consistency bug is reported.

Other checkers are employed from existing tools. I won't talk too much about it since I have not read these papers...(I will one day! A flag has been created!)

## My Thoughts on the Aforementioned Three Papers

All the three papers concentrate on finding semantic bugs in file systems. B3 especially focuses on crash-consistency bugs, JUXTA on semantic violation bugs and HYDRA on all semantic bugs, but also proposes a novel crash-consistency checker.

These papers are really inspiring. Some things I have learned from the three papers are:

1. perhaps starting a new idea from some observations is a good way. As in B3, the authors proposed this approach and according implementation based on their case study on the reported crash-consistency bugs. Finding the commonalities are important.
2. sometimes straight-forward ideas may prove efficient. HYDRA uses history-tracking in SYMC3, JUXTA uses two simple statistical models as the basis.

Some questions about these three papers:

1. In B3, is the workload generation too simple? As to my understanding, the generated workload feels like random inputs generated within a boundary, which is without feedback! It looks like fuzzing but different in many ways. Isn't this method too brutal? It's like enumerating all possible inputs, and no feedback is provided? Not to mention that this "boundary" is also simple, the authors just assigns a number, like 32, to be its boundary. And once the boundary is set, it is no longer modified during the whole process.
2. In JUXTA, the statistical models are simple, **but** the results are not ideal...the false positive rate is over 80%!! That means every time I get some results from JUXTA, 4 out of 5 may not be bugs and I still need to look into the 4 false bugs to determine whether they are bugs. I think it is the statistical models that make the false positvie rate so high. For example, in the entropy-based model, I do not understand fully how the probability of each event is determined. I mean, if the probability is assigned based on the observed facts, that would mean a great deal of time. Or if the probability is determined by the authors, then how can the entropy be appropriate? Moreover, is entropy really a good fit for this?
3. In HYDRA, fuzzing a file system costs time since each time we need to mount the file system...

Some thoughts on improvement:

* combining B3 and fuzzing, B3 uses black-box but without feedback, fuzzing is with feedback but needs to modify file system each time. Combining both may be a solution.

* construct a better model for the bug-detection scenario. I have not come up with any, but I think maybe my inter-discipline knowledge might help.

These thoughts are naive..I welcome any comments and discussion!

## [Witcher: Systematic Crash Consistency Testing for Non-Volatile Memory Key-Value Stores](https://www3.cs.stonybrook.edu/~dongyoon/papers/SOSP-21-Witcher.pdf)

Finished reading Witcher! retty easy to read, salute to the authors who have demonstrated their ideas in a fluent way! Continue to read Hippocrates. Will make the note later.

## [Hippocrates: Healing Persistent Memory Bugs Without Doing Any Harm Extended Abstract](https://dl.acm.org/doi/abs/10.1145/3445814.3446694)