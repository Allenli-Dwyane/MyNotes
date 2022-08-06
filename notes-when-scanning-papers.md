# Notes On Papers

This note is for me briefly scanning papers when trying to find my research topic, i.e., only scanning the abstract or instroduction to get the main idea of this paper, rather than diving into the details of design or implementation.

## OSDI 22

### [Metastable Failures in the Wild](https://www.usenix.org/conference/osdi22/presentation/huang-lexiang)

Motivation: [Metastable Failures in Distributed
Systems.](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-bronson.pdf) introduced a new class of failures in distributed systems called *metastable failures*.

Definition of *metastable failure state*: the
state of a permanent overload with an ultra-low goodput
(throughput of useful work). 

* stable state: the state when a system experiences a low enough load than it can successfully recover from temporary overloads

* vulnerable state: the state when a system experiences a high load, but it can successfully handle that load in the absence of temporary overloads.

* metastable failure occurs when a system is in a vulnerable state, and a trigger causes a temporary overload that sets off a sustaining effect (a work amplification due to a common-case optimization), which tips off the system into the metastable failure state.

Characteristics of *metastable failure state*:

* the sustaining effect keeps the system in the metastable
failure state even after the trigger is removed.

* Not caused by some specific software/hardware bugs, but arises from optimizations that lead to sustained work amplification. Thus, it is hard to predict.

Basically, *metastable failures* are transformed from the vulnerable state, which is commonly seen in most large-scale systems for that running in a vulnerable state is much more efficient than in a stable state. Therefore, *metastable failures* are pretty common and worth being inspected upon.

Contributions of this paper: 

1. A study of metastable failures in the wild that confirms
metastable failures are universally observed and comprise a
substantial fraction of the most severe outages

2. categorizes two types of triggers and two types of amplification mechanisms, and better explains how metastable failures happen.

3. An insider view at Twitter of a new type of metastable
failure where garbage collection acts as an amplification
mechanism

4. Proposes three example applications to experimentally reproduce metastable failures.

### [Demystifying and Checking Silent Semantic Violations in Large Distributed Systems](https://www.usenix.org/conference/osdi22/presentation/lou-demystifying)

Motivation: Bugs that silently violate a system's semantics without apparent anomalies (silent semantic failures) can cause prolonged damage and are hard o detect/address.

Major contribution: *Oathkeeper*, a tool that automatically infers semantic rules from past failures and enforces the rules at runtime to detect new failures.

### [RESIN: A Holistic Service for Dealing with Memory Leaks in Production Cloud Infrastructure](https://www.usenix.org/conference/osdi22/presentation/lou-resin)

Motivation: memory leaks in large-scale systems, such as Microsoft Azure, are the potentials of disastrous system failures. And they are hard to deal with, especially in a production cloud infrastructure setting. Moreover, detecting and fixing such problems is time-consuming.

Previous works: 

1. static analysis: not efficient enough in detecting such bugs
   
2. dynamic tracking: effective in doing so, but too costly.

Contributions of this paper: RESIN

1. highly scalable to address the memory leak issues in large cloud infrastructure. It analyzes all the components on millions of nodes on Azure.
2. low overheads introduced and non-intrusive.
3. good accuracy.

### [Cancellation in Systems: An Empirical Study of Task Cancellation Patterns and Failures](https://www.usenix.org/conference/osdi22/presentation/sethi)

Motivation: task cancellation (which stops the execution of a software component) is crucial for the current execution of today's systems. And efficient and correct task cancellation is non-trivial, e.g., a cancelled task may be depended on by other tasks. But few prior works have focused on the problems of task cancellation.

Contributions of this paper: "attempts to provide an in-depth analysis of cancellation usage and problems in popular software applications across multiple languages, which we hope will help guide cancellation-related systems research and design."

### [Automatic Reliability Testing For Cluster Management Controllers](https://www.usenix.org/conference/osdi22/presentation/sun)

    