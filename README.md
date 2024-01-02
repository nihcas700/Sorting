This project contains the implementations of in-memory and diskbased sorting algorithms and compares their runtimes across various optimizations. The code was run on a machine with 8-core CPU with 32 GB RAM.

# In-memory Merge sort

[This package](https://github.com/nihcas700/ExternalSort/tree/master/src/main/java/inmemory) contains the following implementations

1. Sequential Merge sort - using standard divide and conquer
2. Parallel Merge sort - using ForkJoinPool framework

Both the algorithms are run on a list of 10^8 integers, and their runtimes are as follows :-

| Number of integers      | Sequential Merge Sort      | Parallel Merge Sort             |
| ------------------      | -------------------------- | ------------------------------- |
| 10^8                    | 100188 millis (100 seconds)| 46251 millis (46 seconds)       |


As can be seen, `Parallel Merge Sort` takes around half the time of `Sequential Merge Sort`.

# External Sort

[This package](https://github.com/nihcas700/ExternalSort/tree/master/src/main/java/diskbased) contains the following implementations

1. External Sort with blocking IO - K-way merge (different values of K)
2. [Upcoming] External Sort with Async IO.

The algorithms are run on an input file containing 10^9 integers, and their runtimes are as follows :-

|  Number of integers   | External Sort with blocking IO (K = 2) |
|-----------------------|----------------------------------------|
|  10^9                 | 1225.730 seconds (20.42 mins)          |

The following sections document the optimizations that were made to both the implementations 
and their performance implications along the way.

### Optimization 1 (for K = 2) - Changed the way two sorted lists were merged.
Initially, the signature of the merge method looked like `merge(list, start, mid, end)`. It expected
a single list containing two sorted sub-lists - first ranging from start to mid and the next sublist ranging from
mid+1 to end. The implementation was copying the second sublist (mid+1 to end) over to a new list
before performing a merge, eventually having the final sorted list in the input list itself.

The optimization here was to avoid the copying by providing two sorted lists in the input itself. Now, the
method signature looks like `merge(list1, list2)`. Their updated runtimes are as follows :- 

| Number of integers | External Sort with blocking IO (K = 2) | 
| ------------------ |----------------------------------------|
| 10^9               | 929.743 seconds (15.49 mins)           |

### Optimization 2 - Changed the way sorted lists were merged, to pave way for K-way merge
We changed the implementation of `merge(list1, list2)` to `merge(List<Iterator<Integer>> iterators)` to avoid the step of 
copying the data to the intermediate lists, which would not only save some runtime, but also reduce the memory footprint,
which can be used else where. Their updated runtimes for different values of `K` are as follows :-

| Value of K | Runtimes                         |
|------------|----------------------------------|
| K=2        | 929.743 seconds (15.49 mins)     |
| K=10       | 420.322 seconds (7 minutes)      |
| K=33       | 375.648 seconds (6.26 minutes)   |

### Optimization 3 - Removed the need of a buffer to store the output of merge.
We changed the `merge(List<Iterator<Integer>> iterators)` method to return an iterator instead of a list, to save some runtime
and the memory footprint.

| Optimizations  | Runtimes                       |
|----------------|--------------------------------|
| Optimization 2 | 375.648 seconds (6.26 minutes) |
| Optimization 3 | 345.422 seconds (5.75 minutes) |

At this point, no intermediate buffer is used for the K-way merge process.

### Optimization 4 - Increased the buffer size of the BufferedReaders used in K-way merge 
| Optimizations  | Runtimes                       |
|----------------|--------------------------------|
| Optimization 3 | 345.422 seconds (5.75 minutes) |
| Optimization 4 | 311.465 seconds (5.19 minutes) |

### Optimization 5 - Removed the redundant buffer copy in divide and scatter step.
Similar to Optimization 3.

| Optimizations  | Runtimes                       |
|----------------|--------------------------------|
| Optimization 4 | 311.465 seconds (5.19 minutes) |
| Optimization 5 | 302.571 seconds (5.04 minutes) |

### Optimizations that did not work

###### Compression of intermediate files' outputs
This approach performed as good as the corresponding optimizations. One probable reason why it didn't add any value was 
because probably the I/O bandwidth was never the bottleneck.

# Learnings
1. Prefer iterators over the temporary buffers (if performance is the goal). Be careful, this would make code pretty 
hard to read.
