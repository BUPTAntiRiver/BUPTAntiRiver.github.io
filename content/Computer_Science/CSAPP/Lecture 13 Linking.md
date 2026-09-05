# Linking

## 为什么需要链接？

1. 模块化，让我们能够分开编写代码，而不是处理一大坨臃肿的代码。
2. 能够将分散的代码编译后组合起来，提高程序运行效率，例如多个模块都导入某个库我们只需要一次一起导入就可以了。

### Linker 都做些啥？

1. Symbol resolution 符号处理
   将程序内对于符号的定义和引用关联起来，它们是一一对应的。
2. Relocation
   把所有代码和数据整合到一起，同时调整每个符号的地址，使得最终它们在内存中的可执行区域。当然也要更新对于这些符号的引用。

#### Object files

我们有三种 Object 文件，也就是所谓的模块。

- Relocatable object file (`.o` file)
  包含代码和数据，不过是一种全新的形式，可以和其他 Relocatable object file 结合形成可执行对象文件。每个 `.o` 文件都由一个对应的 `.c` 文件得到。
- Executable object file (`.out` file)
  包含代码和数据，可以直接复制到内存中然后运行。
- Shared object file (`.so` file)
  一种特殊的 Relocatable object file 可以动态地被加载入内存或者链接，在加载时或者运行时。

#### Linker Symbols

- Global symbols
- External symbols
- Local symbols

# Case Study: Library Interpositioning
