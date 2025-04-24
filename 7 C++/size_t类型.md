size_t 是一个无符号整数类型，用于表示特定实现中任何对象（包括数组）的大小。

运算符 sizeof 生成 size_t 类型的值。

size_t 的最大大小通过 一个宏常量在 header （ header in C++） 中定义。

size_t保证至少为 16 位宽。此外，POSIX 还包括 ssize_t，这是一种与 size_t 宽度相同的有符号整数类型。
