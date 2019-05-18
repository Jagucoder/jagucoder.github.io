---
layout: post
title: "ELF file basic info"
categories: [linux]
tags: [elf, linux]
redirect_from:
  - /2019/05/17/
---
* Kramdown table of contents
{:toc .toc}
ELF 分为头部和文件数据两个大块

# ELF头部

用`readelf ***.elf -h` 查看elf头部

```
 readelf ***.elf -h
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32                         //32-bit架构 
  Data:                              2's complement, little endian //看大小端
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)  //看该elf的用途
  Machine:                           ARM                       //期望芯片类型(ARM) 
  Version:                           0x1
  Entry point address:               0x13c4d
  Start of program headers:          52 (bytes into file)
  Start of section headers:          4327716 (bytes into file)
  Flags:                             0x5000002, has entry point, Version5 EABI
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         10
  Size of section headers:           40 (bytes)
  Number of section headers:         27
  Section header string table index: 24

```

# ELF文件数据

ELF文件数据分两类一个用于链接重定向一个用于存数据，Segments 用于链接器链接后执行(used for the linker to allow execution)，分类指示(categorizing instructions)和data(sections)

## ELF Program Headers and Segments

ELF 可能含有0个或多个segment， 它们描述了在执行时如何创建进程和内存空间，当内核看到这些segment时，会将它们映射到虚拟地址空间，比如`mmap`系统调用。

```
readelf ***.elf --segments

Elf file type is EXEC (Executable file)
Entry point 0x13c4d
There are 10 program headers, starting at offset 52

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  LOAD           0x000174 0x00000000 0x00000000 0x00200 0x00200 R   0x4
  LOAD           0x000374 0x00000400 0x00000400 0x00010 0x00010 R   0x4
  LOAD           0x000384 0x0000a000 0x0000a000 0x00200 0x00200 R   0x4
  LOAD           0x000588 0x0000a400 0x0000a400 0xa4054 0xa4054 RWE 0x8
  LOAD           0x0a45e0 0x20000000 0x000ae454 0x061c0 0x061c0 RW  0x8
  LOAD           0x0aa7a0 0x20013b00 0x000b4614 0x00ac0 0x00ac0 R E 0x8
  LOAD           0x0ab260 0x000b50d4 0x000b50d4 0x00024 0x00024 RW  0x1
  LOAD           0x0ab284 0x200145c0 0x000b50d4 0x00000 0x00400 RW  0x1
  LOAD           0x0ab284 0x1fff0000 0x1fff0000 0x00000 0x10000 RW  0x1
  LOAD           0x0ab288 0x200061c0 0x200061c0 0x00000 0x29e30 RW  0x8

 Section to Segment mapping:
  Segment Sections...
   00     .bootvectors
   01     .cfmprotect
   02     .vectors
   03     .text .ARM .init_array .fini_array
   04     .data
   05     .ramfunc
   06     .romp
   07     .rom_func_copy
   08     .shared
   09     .bss .rom_func_copy .kernel_data

```

## ELF Section Header or Sections

Section的头部定义了这个文件的所有section，这些头部信息主要用于链接和重定向。在编译器将C源码转换成汇编后，就会将Section 信息塞入ELF二进制。

对于可执行的ELF，有四种主要的Sections：.text , .data, .rodata , .bss  

每一种用不同的访问权限装载，可通过readelf -S 查看

### .text

包含可执行代码。它会以可读可执行的权限被打包进segment，且只会被加载一次，可以被objdump 读取。

**存放：常量**

### .data

包含初始化的数据，读写权限

**存放：初始化为非0的全局变量**

### .rodata

包含初始化的数据，只有读权限（=A）

**存放：初始化的数组**

### .bss

未初始化的数据，读写权限（=WA）

**存放：未初始化的静态变量、未初始化的全局变量**

### 另外：不在ELF中的

堆(heap)：动态分配

栈(stack)：局部变量，未初始化数组，函数调用时传递参数和返回值



```
高地址>>>>>>>>
    stack(grow down)
    ||||
    ||||
    heap(grow up)
    |||
uninnitialized data(bss)//initialized to 0
    |||
initialized data//read from program file
    |||
    text
低地址>>>>>>>>
```

## 静态和动态的二进制

### 区分动态或静态二进制

```
$ file /bin/ps
/bin/ps: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=2053194ca4ee8754c695f5a7a7cff2fb8fdd297e, stripped
```

### 查看动态二进制依赖

```
$ ldd /bin/ps
linux-vdso.so.1 => (0x00007ffe5ef0d000)
libprocps.so.3 => /lib/x86_64-linux-gnu/libprocps.so.3 (0x00007f8959711000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f895934c000)
/lib64/ld-linux-x86-64.so.2 (0x00007f8959935000)
```

查看潜在依赖可以用`lddtree`

**参考：<https://linux-audit.com/elf-binaries-on-linux-understanding-and-analysis/>**

