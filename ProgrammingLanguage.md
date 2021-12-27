# JIT 编译器原理简述/原理简述

http://accu.cc/content/jit_tour/principle/

编写一个JIT编译器只需要四步：
1. 申请一段可写和可执行的内存
2. 将源码翻译为机器码(通常经过汇编)
3. 将机器码写入第一步申请的内存
4. 执行这部分内存

于是，一个体现了JIT编译器核心的最简单代码就出现了(太他妈神了)：
```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>

int main(int argc, char *argv[]) {
  // Machine code for:
  //   mov eax, 0
  //   ret
  unsigned char code[] = {0xb8, 0x00, 0x00, 0x00, 0x00, 0xc3};

  if (argc < 2) {
    fprintf(stderr, "Usage: jit1 <integer>\n");
    return 1;
  }

  // Overwrite immediate value "0" in the instruction
  // with the user's value.  This will make our code:
  //   mov eax, <user's value>
  //   ret
  int num = atoi(argv[1]);
  memcpy(&code[1], &num, 4);

  // Allocate writable/executable memory.
  // Note: real programs should not map memory both writable
  // and executable because it is a security risk.
  void *mem = mmap(NULL, sizeof(code), PROT_WRITE | PROT_EXEC,
                   MAP_ANON | MAP_PRIVATE, -1, 0);
  memcpy(mem, code, sizeof(code));

  // The function will return the user's value.
  int (*func)() = mem;
  return func();
}
```