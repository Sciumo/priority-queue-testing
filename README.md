# A Back-to-Basics Empirical Study of Priority Queues

* Daniel H. Larkin
  * Research at Princeton University partially supported by NSF grant CCF-0832797 and a Google PhD Fellowship.†Princeton University, Department of Computer Science. Email:dhlarkin@cs.princeton.edu.
* Siddhartha Sen
  * Microsoft Research SVC. Email:sisen@microsoft.com
* Robert E. Tarjan
  * Princeton University, Department of Computer Science and Microsoft Research SVC Email:ret@cs.princeton.edu.
* [arXiv:1403.0252v1](https://arxiv.org/abs/1403.0252v1)
* see [docs/1403.0252v1.pdf](docs/1403.0252v1.pdf)

  **March 4, 2014**

## Abstract

The theory community has proposed several new heap variants in the recent past which have remainedlargely  untested  experimentally.   We  take  the  field  back  to  the  drawing  board,  with  straightforward implementations of both classic and novel structures using only standard, well-known optimizations.  Westudy  the  behavior  of  each  structure  on  a  variety  of  inputs,  including  artificial  workloads,  workloads generated by running algorithms on real map data, and workloads from a discrete event simulator usedin recent systems networking research.  We provide observations about which characteristics are mostcorrelated  to  performance.   For  example,  we  find  that  the  **L1  cache  miss  rate  appears  to  be  strongly correlated with wallclock time**.  We also provide observations about how the input sequence affects therelative  performance  of  the  different  heap  variants.   For  example,  we  show  (both  theoretically  and  inpractice)  that  certain  random  insertion-deletion  sequences  are  degenerate  and  can  lead  to  misleading results.  Overall, our findings suggest that while the conventional wisdom holds in some cases, it is **sorely mistaken** in others.

## Introduction

The priority queue is a widely used abstract data structure.  Many theoretical variants and implementations exist,  supporting  a  varied  set  of  operations  with  differing  guarantees.

We  restrict  our  attention  to  the following base set of commonly used operations:

* `Insert(Q, x, k)` — insert item `x` with key `k` into heap `Q` and return a handle ` ̄x`.
* `DeleteMin(Q)` — remove the item of minimum key from heap `Q` and return its corresponding key `k`
* `DecreaseKey(Q, ̄x, k′)`  —  given  a  handle  ` ̄x`,  change  the  key  of  item `x` belonging  to  heap `Q` to  be `k′`, where `k′` is guaranteed to be less than the original key `k`
  
It has long been known that either `Insert` or `DeleteMin` must take `Ω (logn)` time due to the classic lower  bound  for  sorting,  but  that  the  other  operations  can  be  done  in `O(1)`  time.   In  practice,  the worst-case  of  logn is  often  not  encountered  or  can  be  treated  as  a  constant,  and  for  this  reason  simpler structures with logarithmic bounds have traditionally been favored over more complicated,  constant-time alternatives.  In light of recent developments in the theory community and the **outdated nature of the most widely cited experimental studies on priority queues**, we aim to revisit this area  and  reevaluate  the  state  of  the  art.   More  recent  studies have  been  narrow  in  focus  with respect to the implementations considered (e.g., comparing a single new heap to a few classical ones), the workloads tested (e.g., using a few synthetic tests), or the metrics collected (e.g., measuring wallclock time and element comparisons).  In addition to the normal metric of wallclock time, we have collected additional metrics such as branching and caching statistics.  Our goal is to identify experimentally verified trends which can provide guidance to future experimentalists and theorists alike.  We stress that this is not the final wordon the subject, but merely another line in the continuing dialogue.

## Implementation

In implementing the various heap structures, we take a different approach from the existing algorithm engineering  literature,  in  that  **we  do  not  perform  any  algorithm  engineering**.   That  is,  our  implementations are **intentionally straightforward** from their respective descriptions in the original papers.  The lackof considerable tweaking and algorithm engineering in this study is, we believe, an example of naıvete as a virtue.  We expect that this would accurately reflect the strategy of a practitioner seeking to make initial comparisons between different heap variants. As a sanity check, we also compare our implementations witha state-of-the-art, well-engineered implementation often cited in the literature.Our high-level findings can be summarized as follows.  We find that wallclock time is highly correlatedwith the cache miss rate, especially in the L1 cache.  High-level theoretical design decisions—such as whetherto use an array-based structure or a pointer-based one—have a significant impact on caching,  and whichdesign fares best is dependent on the specific workload.  For example, Fibonacci heaps sometimes outperform implicit d-ary heaps, in contradiction to conventional wisdom.  Even a well-engineered implementation like Sanders’ sequence heap can be bested by our untuned implementations if the workload favors a different method. Beyond caching behavior, those heaps with the simplest implementations tend to perform very well.  It is not always the case that a theoretically superior or simpler structure lends itself to simpler code in practice. Pairing heaps dominate Fibonacci heaps across the board, but interestingly, recent theoretical simplifications to Fibonacci heaps tend to do worse than the original structure. Furthermore we found that a widely-used benchmarking workload is degenerate in a certain sense.  As the  sequence  of  operations  progresses the  distribution  of  keys  in  the  heap  becomes  very  skewed  towards large keys,  contradicting the premise that the heap contains a uniform distribution of keys.  This can be shown both theoretically and in practice. Our complete results are detailed in Sections 4 and 5.  We first describe the heap variants we implemented in Section 2, and then discuss our experimental methodology and the various workloads we tested in Section 3. We conclude in Section 6 with some remarks.
