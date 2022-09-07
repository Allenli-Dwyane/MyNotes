# Notes on Persistent Memory Related Security Issues

Hopefully this will be my PhD research project.

## Reading List

1. [Witcher: Systematic Crash Consistency Testing for Non-Volatile Memory Key-Value Stores](https://www3.cs.stonybrook.edu/~dongyoon/papers/SOSP-21-Witcher.pdf) SOSP'21
2. [Hippocrates: Healing Persistent Memory Bugs Without Doing Any Harm Extended Abstract](https://dl.acm.org/doi/abs/10.1145/3445814.3446694) ASPLOS'21
3. [RECIPE: Converting Concurrent DRAM Indexes to Persistent-Memory Indexes](https://dl.acm.org/doi/10.1145/3341301.3359635) SOSP'19
4. [Write-Optimized and High-Performance Hashing Index Scheme for Persistent Memory](https://www.usenix.org/conference/osdi18/presentation/zuo) OSDI'18
5. [PMTest: A Fast and Flexible Testing Framework for Persistent Memory Programs](https://dl.acm.org/doi/10.1145/3297858.3304015) ASPLOS'19
6. [AGAMOTTO: How Persistent is your Persistent Memory Application?](https://www.usenix.org/conference/osdi20/presentation/neal) OSDI'20
7.  [MOD: Minimally Ordered Durable Datastructures for Persistent Memory](https://dl.acm.org/doi/10.1145/3373376.3378472) ASPLOS'20
8. [Cross-Failure Bug Detection in Persistent Memory Programs](https://dl.acm.org/doi/10.1145/3373376.3378452) ASPLOS'20
9. [DURINN: Adversarial Memory and Thread Interleaving for Detecting Durable Linearizability Bugs](https://www.usenix.org/conference/osdi22/presentation/fu) OSDI'22
10. [Mnemosyne: Lightweight Persistent Memory](https://pages.cs.wisc.edu/~swift/papers/asplos11_mnemosyne.pdf) ASPLOS'11
11. [NV-Heaps: Making Persistent Objects Fast and Safe with Next-Generation, Non-Volatile Memories](http://mesl.ucsd.edu/pubs/Coburn_ASPLOS11.pdf) ASPLOS'11
12. [Yat: A Validation Framework for Persistent Memory Software](https://www.usenix.org/system/files/conference/atc14/atc14-paper-lantz.pdf) ATC'14
13. [Crash Consistency in Encrypted Non-Volatile Main Memory Systems](https://ieeexplore.ieee.org/document/8327018) HPCA'18
14. [PMFuzz: Test Case Generation for Persistent Memory Programs](https://www.cs.virginia.edu/~smk9u/Liu_PMFuzz_ASPLOS21.pdf) ASPLOS'21
15. [Atlas: Leveraging Locks for Non-volatile Memory Consistency](https://dl.acm.org/doi/pdf/10.1145/2714064.2660224) OOPSLA'14

## Persistent Memory Background Knowledge

Some articles/talks worthing reading are:

1. Andy Rudoff's articles on USENIX: [Programming Models for Emerging NonVolatile Memory Technologies](https://www.usenix.org/system/files/login/articles/08_rudoff_040-045_final.pdf), [Persistent Memory Programming](https://www.usenix.org/system/files/login/articles/login_summer17_07_rudoff.pdf)