First implement a explicit list, then write functions like add block and remove block to track the free list.

# The data structure
**Requirements**:
- tell us *where* the blocks are, how *big* they are, and whether they are *free*
- be able to *CHANGE* the data structure during calls to `malloc` and `free`
- be able to find the **next free block** that is a good fit for a given payload
- be able to quickly *mark* a block as free/allocated
- be able to *detect* when we are out of blocks (what do we do when out of blocks?)

Read the book to get some starting help.

Use Macro to control heap checker.