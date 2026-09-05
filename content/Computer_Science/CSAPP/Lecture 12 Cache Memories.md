# General Cache Orgnization

Cache 的结构可以理解为一个大型二维数组，包含三个参数 $(S,E,B)$，$S$ 表示含有 $2^S$ 个 sets 即行数，$E$ 表示含有 $2^E$ 个 lines per set 即列数，$B$ 表示每个 cache block 中包含 $2^B$ 个字节的数据，此外还有 valid bit $v$ 表示这些数据是否有意义，例如电脑刚开机很多随机初始化的 cache 就没有意义，以及 tag 用来辅助搜索缓存。

## Cache Read 读缓存

Address of Word 读取缓存地址包含三个部分：

- tag
- set index
- block offset

在读取缓存的过程中，我们先根据 set index 定位目标在哪一个 set 中，然后搜索 line 匹配 tag 如果存在再检查 valid bit，一切正常表明读取命中。最后通过 block offset 判断从哪里开始读取数据。
如果读取没有命中，那么从内存中读取数据，替换原有的 Line。

## 那末写内存呢？

我们有多份重复的数据存储在以下几个地方：L1, L2, L3, Main Memory, Disk

### 当命中写入的时候该怎么做？

- **Write-through** 直接写入：写入缓存时，直接写到对应的内存。
- **Write-back** 磨洋工写入：直到需要改变当前 line 存储的写入内容时再将数据写到内存里。为此我们需要一个额外的比特来记录当前块是否已经被写入过。

### 未命中写入如何呢？

- **Write-allocate** 只写入到缓存，修改相应的 line。如果后续还要修改该位置的数据，那么久没什么问题。
- **No-write-allocate** 直接写入到内存，不存到缓存中。

# 程序优化

我们可以利用缓存的特性来优化程序效率。

- 利用空间局部性，步长为 1 的迭代
- 利用时间局部性，多次访问同一个位置的数据
- 分块 Blocking
