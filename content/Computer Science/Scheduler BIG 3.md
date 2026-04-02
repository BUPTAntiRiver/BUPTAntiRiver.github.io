# O(1) Scheduler

It can schedule within a constant amount of time.

There are 2 arrays used in O(1) scheduler, an **active** one and an **expired** one. Each process is given **a fixed time quantum**, and when it is preempted, it will be moved to the expired array. Once all the tasks in the active array have exhausted their time quantum and have been moved to expired array, an array switch happens, expired array becomes the new active array and previous empty active array is taken as a expired array.

Each CPU has such 2 arrays.

# CFS

CFS has multiple queues, one for each CPU, whose nodes are time-ordered schedulable entities that are kept sorted by [red–black trees](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree "Red–black tree"). Each queue uses red-black tree as its data structure. CFS give away the old notion of per-priorities fixed time slices, and instead it aims at giving a fair share of CPU time to tasks.

Each process has a calculated _maximum execution time_, if the entity finished within given time, then it is simply removed from the queue, if it is not finished then it will be reinsert into the queue added with running time.

The insertion of new nodes takes $O(\log N)$, and choosing the next entity to run takes constant time.

# BFS

Brain Fuck Scheduler, I have to spell out the name!!!

## Motivation

The creator thought O(1) and CFS were designed mainly for greater throughput on larger servers, while this may cause latency problems, so he wants to focus on simplicity and reponsiveness.

The objective of BFS is to provide a simpler scheduler that does not require adjustment or tuning parameters to tailor performance to a specific type of computational workload.
Each CPU has only one single global runqueue.
