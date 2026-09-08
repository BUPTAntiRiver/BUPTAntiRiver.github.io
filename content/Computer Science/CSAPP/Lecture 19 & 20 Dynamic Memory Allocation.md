# Basic Concepts
## Knowing How Much to Free
- Standard Method
	- Keep the length of a block preceding needed length, add a extra word as *header field* or *header* to track the length of allocated block.
	- This forms a *Implicit list*, we can also build a *Explicit list* only among the free blocks using pointers, explicit one is more efficient but requires more space.
	- There are also more advanced methods like *Segregated free list* or *Block sorted by size* which uses a balanced tree.
### Method 1: Implicit List
For each block we need both *size* and *allocation status*: could store these information in 2 words but this is wasteful!
**Standard trick**: use the lowest bit of size to indicate allocation status.
We need to extend the size of free block when there are continuous free blocks, called coalescing. We can add *Boundary tags* at the end of free blocks to replicate size/allocated word. It allows us to traverse the "list" backwards, but requires extra space.
# Advanced Concepts
## Freeing With Explicit Free Lists
**Insertion Policy**: where in the free list do you want to put the newly freed block?
- *LIFO (last in first out)*
	- Insert at the beginning of the free list
	- **Pro**: simple and constant time
	- **Con**: studies suggest fragmentation is worse than address ordered
- *Address-ordered*
	- Insert free blocks so that list blocks are always in address order
	- **Pro**: lower fragmentation
	- **Con**: requires search
