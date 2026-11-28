# Block

```cpp
class BlockNode : public StmtNode {
 public:
  Array<IterVar> iter_vars;
  Array<BufferRegion> reads;
  Array<BufferRegion> writes;
  String name_hint;
  Optional<Stmt> init;
  Stmt body;
  Array<Buffer> alloc_buffers;   // ← THIS is where alloc_buffer lives
};
```

When we try to modify things like `alloc_buffers`, we can not just append it like a normal statement, it lives inside the `BlockNode`. And it is immutable, so we have to create a new one.
