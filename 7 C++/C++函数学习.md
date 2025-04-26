# size_t

size_t 是一个无符号整数类型，用于表示特定实现中任何对象（包括数组）的大小。

运算符 sizeof 生成 size_t 类型的值。

size_t 的最大大小通过 一个宏常量在 header （ header in C++） 中定义。

size_t保证至少为 16 位宽。此外，POSIX 还包括 ssize_t，这是一种与 size_t 宽度相同的有符号整数类型。

# sizeof()

sizeof() 函数是一个运算符而不是函数，用于计算一个类型或变量所占用的内存字节数。可以用它来获取任何类型的数据的字节数，包括基本数据类型、数组、结构体、共用体等等。

```c++
#include <iostream>

using namespace std;

struct dir_demo {
    int age;
    string name;
} DIR_DEMO[]{
        {1, "zzfan"},
};
int main() {
    int i = 18;
    string s = "zzfan";
    cout << sizeof(i) << endl;
    cout << s.size() << endl;
    cout << sizeof(s) << endl;
    cout << sizeof(DIR_DEMO) << endl;

};
//output:
//4
//5
//32
//40
```

长度和类型有关



# calloc()

`calloc` 会将分配的内存**初始化为全零**，适用于需要清零初始状态的场景。



# strnact()

**strncat** 是 C 语言中的一个预定义函数，用于将一个字符串的指定长度追加到另一个字符串的末尾。它在 *string.h* 头文件中定义

```cpp
#include <stdio.h>
#include <string.h>

int main() {
    char src[50] = "world";
    char dest[50] = "Hello ";
    strncat(dest, src, 3);
    printf("最终的目标字符串：%s\n", dest); // 输出: Hello wor
    return 0;
}
```



# fgets()

C 库函数 **char \*fgets(char \*str, int n, FILE \*stream)** 从指定的流 stream 读取一行，并把它存储在 **str** 所指向的字符串内。当读取 **(n-1)** 个字符时，或者读取到换行符时，或者到达文件末尾时，它会停止，具体视情况而定。

下面是 fgets() 函数的声明。

```c
char *fgets(char *str, int n, FILE *stream)
```

参数

- **str** -- 这是指向一个字符数组的指针，该数组存储了要读取的字符串。
- **n** -- 这是要读取的最大字符数（包括最后的空字符）。通常是使用以 str 传递的数组长度。
- **stream** -- 这是指向 FILE 对象的指针，该 FILE 对象标识了要从中读取字符的流。

# memmove()

## 描述

C 库函数 **void \*memmove(void \*str1, const void \*str2, size_t n)** 从 **str2** 复制 **n** 个字符到 **str1**，但是在重叠内存块这方面，memmove() 是比 memcpy() 更安全的方法。如果目标区域和源区域有重叠的话，memmove() 能够保证源串在被覆盖之前将重叠区域的字节拷贝到目标区域中，复制后源区域的内容会被更改。如果目标区域与源区域没有重叠，则和 memcpy() 函数功能相同。

## 声明

下面是 memmove() 函数的声明。

```
void *memmove(void *str1, const void *str2, size_t n)
```

## 参数

- **str1** -- 指向用于存储复制内容的目标数组，类型强制转换为 void* 指针。
- **str2** -- 指向要复制的数据源，类型强制转换为 void* 指针。
- **n** -- 要被复制的字节数。

## 返回值

该函数返回一个指向目标存储区 str1 的指针。

## 实例

下面的实例演示了 memmove() 函数的用法。

```c++
#include <stdio.h>
#include <string.h>

int main ()
{
   const char dest[] = "oldstring";
   const char src[]  = "newstring";

   printf("Before memmove dest = %s, src = %s\n", dest, src);
   memmove(dest, src, 9);
   printf("After memmove dest = %s, src = %s\n", dest, src);

   return(0);
}
```

让我们编译并运行上面的程序，这将产生以下结果：

```bash
Before memmove dest = oldstring, src = newstring
After memmove dest = newstring, src = newstring
```
