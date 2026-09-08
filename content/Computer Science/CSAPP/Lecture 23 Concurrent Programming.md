## Process-based
## Event-based Servers
Server maintains *set* of active connections.
**Pros**:
- one logical control flow and address space
- can single-step with a debugger
- no process or thread control overhead

**Cons**:
- more complex
- hard to provide fine-grained concurrency
- cannot take advantage of multiple cores
## Thread-based
### Critical points for Threads
Threads have their **own**:
- Thread ID
- Stack
- Stack pointer
- Instruction pointer
- General Purpose Registers
- Status codes
Threads **share**:
- Code sections
- Open files
- Heap