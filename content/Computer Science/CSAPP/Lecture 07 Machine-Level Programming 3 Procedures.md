# x86-64 Stack

Stack is just a **region of memory** managed with stack discipline
First, we let `%rsp` be at the bottom of the stack and then subtract a piece of address to it, so that the stack grows toward **lower** addresses. The **top** of stack has **lowest** address, and `%rsp` points to the top, while the **bottom** of stack has **highest** address.

## Push

`pushq Src`

- Fetch operand at `Src`
- Decrement `%rsp` by 8
- Write operand at address given by `%rsp`

## Pop

`popq Dest`

- Read value at address given by `rsp%`
- Increment `rsp%` by 8
- Store value at `Dest` (must be register)

## Procedure Control Flow

Use stack to support procedure call and return
**Procedure call**: `call label`

- Push return address to stack
- Jump to label
  Return address:
- Address of the next instruction right after the `call`
  **Procedure return**:
- Pop address from stack
- Jump to address

## Procedure Data Flow

The arguments of a function call are stored in registers: `%rdi, %rsi, %rcx, %rdx, %r8, %r9`
The return value stores in `%rax`
If there are more than 6 arguments, the others are stored in stack.

## Stack-Based Language

**Only one** function can be running at the same time, so we can allocate as many resources as you want using stack during execution.

## Stack Frames 帧栈

当 `call` 一个函数时，将 `%rbp` 设置为该函数调用的起点 base pointer 然后移动 `%rsp` 分配栈空间。当函数调用结束时，将 `%rsp` 移回 `%rbp` 释放栈空间。

## Register Saving Conventions

When procedure `yoo` calls `who`

- `yoo` is the **caller**
- `who` is the **callee**
  The callee might modify the values in caller, how to avoid this?

### Conventions

- **"Caller Saved"**
  - Caller saves temporary values in its frame before the call
- **"Callee Saved"**
  - Callee saves temporary values in its frame before using
  - Restores them before returning to caller

## Recursion 递归

C 编译器处理递归和处理常规函数没有任何区别，得益于它的**栈**结构
