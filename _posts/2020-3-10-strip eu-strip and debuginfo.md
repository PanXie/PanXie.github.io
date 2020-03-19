---
layout:     post
title:      strip eu-strip debuginfo
subtitle:   
date:       2020-3-10
author:     XP
header-img: img/problems.jpg
catalog: true
tags:
    - rpm
    - strip
    - eu-strip
    - debuginfo
    - spec
    - readelf
    - nm
---

## strip, eu-strip, objcopy, debuginfo ##

elf(可执行和可链接格式--Executable and Linkable Format)文件由ELF Header、Sections、Section Header Table
和Program Header Table组成，其中section,又包括代码段(.text), 数据段(.data)，只读数据段(.rodata), .bss, 
注释信息段(.comment), 堆栈提示段(.note.GNU-stack)等等. 

查看efl文件，可以用到file，readelf，objdump，nm等工具。 
写个文件测试下：

```c
#include <stdio.h>

int g_value = 1;
int g_value_uninit;
const int g_const_value = 10;

void fun(int age)
{
    printf("age = %d\n",age);
}

int main()
{
    int value = 1;
    static int s_value = 1;
    char *name = "Win!xp!";
    fun(value);

    return 0;
}
```
编译生成可执行文件：
>$ gcc -g hello.c -o hello

使用file，查看hello信息:  
>$ file hello  
```
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 2.6.32, 
BuildID[sha1]=61469bbc6f06707b5c145ac3ba6d79892423e58f, not stripped
```

readelf可以查看更多elf文件的信息，比如文件头，段表等  
> $ readelf -h hello  
```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x400430
  Start of program headers:          64 (bytes into file)
  Start of section headers:          7824 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         36
  Section header string table index: 33
```

段表（Section Header Table）保存了efl中段的基本属性信息，比如每个段的段名，长度，在文件中的偏移，读写权限等。
编译器、链接器和装载器都是依据段表来定位和访问各个段的属性。可以使用`readelf -S` 查看段表信息：  
> $ readelf -S -W hello 

```
Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        0000000000400238 000238 00001c 00   A  0   0  1
  [ 2] .note.ABI-tag     NOTE            0000000000400254 000254 000020 00   A  0   0  4
  [ 3] .note.gnu.build-id NOTE            0000000000400274 000274 000024 00   A  0   0  4
  [ 4] .gnu.hash         GNU_HASH        0000000000400298 000298 00001c 00   A  5   0  8
  [ 5] .dynsym           DYNSYM          00000000004002b8 0002b8 000060 18   A  6   1  8
  [ 6] .dynstr           STRTAB          0000000000400318 000318 00003f 00   A  0   0  1
  [ 7] .gnu.version      VERSYM          0000000000400358 000358 000008 02   A  5   0  2
  [ 8] .gnu.version_r    VERNEED         0000000000400360 000360 000020 00   A  6   1  8
  [ 9] .rela.dyn         RELA            0000000000400380 000380 000018 18   A  5   0  8
  [10] .rela.plt         RELA            0000000000400398 000398 000030 18  AI  5  24  8
  [11] .init             PROGBITS        00000000004003c8 0003c8 00001a 00  AX  0   0  4
  [12] .plt              PROGBITS        00000000004003f0 0003f0 000030 10  AX  0   0 16
  [13] .plt.got          PROGBITS        0000000000400420 000420 000008 00  AX  0   0  8
  [14] .text             PROGBITS        0000000000400430 000430 0001b2 00  AX  0   0 16
  [15] .fini             PROGBITS        00000000004005e4 0005e4 000009 00  AX  0   0  4
  [16] .rodata           PROGBITS        00000000004005f0 0005f0 00001a 00   A  0   0  4
  [17] .eh_frame_hdr     PROGBITS        000000000040060c 00060c 00003c 00   A  0   0  4
  [18] .eh_frame         PROGBITS        0000000000400648 000648 000114 00   A  0   0  8
  [19] .init_array       INIT_ARRAY      0000000000600e10 000e10 000008 00  WA  0   0  8
  [20] .fini_array       FINI_ARRAY      0000000000600e18 000e18 000008 00  WA  0   0  8
  [21] .jcr              PROGBITS        0000000000600e20 000e20 000008 00  WA  0   0  8
  [22] .dynamic          DYNAMIC         0000000000600e28 000e28 0001d0 10  WA  6   0  8
  [23] .got              PROGBITS        0000000000600ff8 000ff8 000008 08  WA  0   0  8
  [24] .got.plt          PROGBITS        0000000000601000 001000 000028 08  WA  0   0  8
  [25] .data             PROGBITS        0000000000601028 001028 000018 00  WA  0   0  8
  [26] .bss              NOBITS          0000000000601040 001040 000008 00  WA  0   0  4
  [27] .comment          PROGBITS        0000000000000000 001040 000035 01  MS  0   0  1
  [28] .debug_aranges    PROGBITS        0000000000000000 001075 000030 00      0   0  1
  [29] .debug_info       PROGBITS        0000000000000000 0010a5 00013d 00      0   0  1
  [30] .debug_abbrev     PROGBITS        0000000000000000 0011e2 00009d 00      0   0  1
  [31] .debug_line       PROGBITS        0000000000000000 00127f 000044 00      0   0  1
  [32] .debug_str        PROGBITS        0000000000000000 0012c3 0000fd 01  MS  0   0  1
  [33] .shstrtab         STRTAB          0000000000000000 001d44 00014c 00      0   0  1
  [34] .symtab           SYMTAB          0000000000000000 0013c0 000738 18     35  53  8
  [35] .strtab           STRTAB          0000000000000000 001af8 00024c 00      0   0  1

```


### section

| 段名 | 说明 |
| ---- | --- |
| .data | 保存已经初始化的全局变量和静态变量|
| .rodata | Read only Data, 保存只读数据，比如字符串常量，全局const变量|
| .comment | 存放编译器版本信息，比如：GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609 |
| .debug | 调试信息 |
| .strtab | 字符串表，存储ELF文件中用到的各种字符串|
| .symtab | 符号表，存储ELF文件用到的所有符号|
| .shstrtab | Section String Table,段表字符串表，保存所有段名 |
| .bss | 存放所有未初始化的全局变量和静态变量 |
| .text | 代码段 |
| .init | 程序初始化代码段 |
| .fini | 程序终结代码段 |

更多信息可以参考[Executable and Linkable Format (ELF)](http://www.skyfree.org/linux/references/ELF_Format.pdf)

查看段信息：
> readelf -p .comment hello
```
String dump of section '.comment':
  [     0]  GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609
```

> readelf -p .strtab hello
```
String dump of section '.strtab':
  [     1]  crtstuff.c
  [     c]  __JCR_LIST__
  [    19]  deregister_tm_clones
  [    2e]  __do_global_dtors_aux
  [    44]  completed.7594
  [    53]  __do_global_dtors_aux_fini_array_entry
  [    7a]  frame_dummy
  [    86]  __frame_dummy_init_array_entry
  [    a5]  hello.c
  [    ad]  s_value.2292
  [    ba]  __FRAME_END__
  [    c8]  __JCR_END__
  [    d4]  __init_array_end
  [    e5]  _DYNAMIC
  [    ee]  __init_array_start
  [   101]  __GNU_EH_FRAME_HDR
  [   114]  _GLOBAL_OFFSET_TABLE_
  [   12a]  __libc_csu_fini
  [   13a]  g_value
  [   142]  _ITM_deregisterTMCloneTable
  [   15e]  _edata
  [   165]  fun
  [   169]  printf@@GLIBC_2.2.5
  [   17d]  __libc_start_main@@GLIBC_2.2.5
  [   19c]  __data_start
  [   1a9]  __gmon_start__
  [   1b8]  __dso_handle
  [   1c5]  _IO_stdin_used
  [   1d4]  __libc_csu_init
  [   1e4]  g_const_value
  [   1f2]  __bss_start
  [   1fe]  main
  [   203]  _Jv_RegisterClasses
  [   217]  __TMC_END__
  [   223]  _ITM_registerTMCloneTable
  [   23d]  g_value_uninit
```
symtab信息可以使用nm查看：  
> nm hello
```
0000000000601040 B __bss_start
0000000000601040 b completed.7594
0000000000601028 D __data_start
0000000000601028 W data_start
0000000000400460 t deregister_tm_clones
00000000004004e0 t __do_global_dtors_aux
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
0000000000601030 D __dso_handle
0000000000600e28 d _DYNAMIC
0000000000601040 D _edata
0000000000601048 B _end
00000000004005e4 T _fini
0000000000400500 t frame_dummy
0000000000600e10 t __frame_dummy_init_array_entry
0000000000400758 r __FRAME_END__
0000000000400526 T fun
00000000004005f4 R g_const_value
0000000000601000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
000000000040060c r __GNU_EH_FRAME_HDR
0000000000601038 D g_value
0000000000601044 B g_value_uninit
00000000004003c8 T _init
0000000000600e18 t __init_array_end
0000000000600e10 t __init_array_start
00000000004005f0 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000600e20 d __JCR_END__
0000000000600e20 d __JCR_LIST__
                 w _Jv_RegisterClasses
00000000004005e0 T __libc_csu_fini
0000000000400570 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
0000000000400548 T main
                 U printf@@GLIBC_2.2.5
00000000004004a0 t register_tm_clones
0000000000400430 T _start
000000000060103c d s_value.2292
0000000000601040 D __TMC_END__
```


### strip
实际发布的软件中，不会包含调试信息，否则安装包会很大，并且实际运行时，不需要这些调试信息。比如打包发布的rpm, 会将调试信息，符号表信息等放入到debuginfo中，如果发成了core dump再安装对应的debuginfo rpm，进行调试。对ELF进行瘦身，有很多工具可以用，strip，eu-strip，objcopy. 上面的hello文件，使用file查看时，可以看到`not stripped`.


### eu-strip

eu-strip在[elfutils](https://sourceware.org/elfutils/)工具集中，elfutils包含了一系列读取，修改，创建elf文件的工具，包括：
  
- eu-ld (a linker)
- eu-nm (for listing symbols from object files)
- eu-size (for listing the section sizes of an object or archive file)
- eu-strip (for discarding symbols)
- eu-readelf (to see the raw ELF file structures)
- eu-elflint (to check for well-formed ELF files)

安装eu-strip 
```
sudo apt-get install elfutils
```

eu-strip说明 
```
 Output selection:
  -f FILE                    Extract the removed sections into FILE
  -F FILE                    Embed name FILE instead of -f argument
  -o, --output=FILE          Place stripped output into FILE

 Output options:
  -g, -d, -S, --strip-debug  Remove all debugging symbols
  -p, --preserve-dates       Copy modified/access timestamps to the output
      --permissive           Relax a few rules to handle slightly broken ELF
                             files
      --reloc-debug-sections Resolve all trivial relocations between debug
                             sections if the removed sections are placed in a
                             debug file (only relevant for ET_REL files,
                             operation is not reversable, needs -f)
      --remove-comment       Remove .comment section
      --strip-sections       Remove section headers (not recommended)
```

使用时，发现--remove-comment选项，并非只是去除.comment段，而是会去除.debug, .comment, .strtab, .symtab.

>$ eu-strip --remove-comment hello
>$ readelf -S -W  hello
```
Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        0000000000400238 000238 00001c 00   A  0   0  1
  [ 2] .note.ABI-tag     NOTE            0000000000400254 000254 000020 00   A  0   0  4
  [ 3] .note.gnu.build-id NOTE            0000000000400274 000274 000024 00   A  0   0  4
  [ 4] .gnu.hash         GNU_HASH        0000000000400298 000298 00001c 00   A  5   0  8
  [ 5] .dynsym           DYNSYM          00000000004002b8 0002b8 000060 18   A  6   1  8
  [ 6] .dynstr           STRTAB          0000000000400318 000318 00003f 00   A  0   0  1
  [ 7] .gnu.version      VERSYM          0000000000400358 000358 000008 02   A  5   0  2
  [ 8] .gnu.version_r    VERNEED         0000000000400360 000360 000020 00   A  6   1  8
  [ 9] .rela.dyn         RELA            0000000000400380 000380 000018 18   A  5   0  8
  [10] .rela.plt         RELA            0000000000400398 000398 000030 18  AI  5  24  8
  [11] .init             PROGBITS        00000000004003c8 0003c8 00001a 00  AX  0   0  4
  [12] .plt              PROGBITS        00000000004003f0 0003f0 000030 10  AX  0   0 16
  [13] .plt.got          PROGBITS        0000000000400420 000420 000008 00  AX  0   0  8
  [14] .text             PROGBITS        0000000000400430 000430 0001b2 00  AX  0   0 16
  [15] .fini             PROGBITS        00000000004005e4 0005e4 000009 00  AX  0   0  4
  [16] .rodata           PROGBITS        00000000004005f0 0005f0 00001a 00   A  0   0  4
  [17] .eh_frame_hdr     PROGBITS        000000000040060c 00060c 00003c 00   A  0   0  4
  [18] .eh_frame         PROGBITS        0000000000400648 000648 000114 00   A  0   0  8
  [19] .init_array       INIT_ARRAY      0000000000600e10 000e10 000008 00  WA  0   0  8
  [20] .fini_array       FINI_ARRAY      0000000000600e18 000e18 000008 00  WA  0   0  8
  [21] .jcr              PROGBITS        0000000000600e20 000e20 000008 00  WA  0   0  8
  [22] .dynamic          DYNAMIC         0000000000600e28 000e28 0001d0 10  WA  6   0  8
  [23] .got              PROGBITS        0000000000600ff8 000ff8 000008 08  WA  0   0  8
  [24] .got.plt          PROGBITS        0000000000601000 001000 000028 08  WA  0   0  8
  [25] .data             PROGBITS        0000000000601028 001028 000018 00  WA  0   0  8
  [26] .bss              NOBITS          0000000000601040 001040 000008 00  WA  0   0  4
  [27] .shstrtab         STRTAB          0000000000000000 001040 0000f3 00      0   0
```

### objcopy

[Linux Objcopy Command Examples to Copy and Translate Object Files](https://www.sanfoundry.com/objcopy-command-usage-examples-in-linux/)


### 参考文献
- [Split debugging info -- symbols](https://www.technovelty.org/category/code.html)
- [深入理解debuginfo](https://blog.csdn.net/chinainvent/article/details/24129311)
- [程序减肥，strip，eu-strip 及其符号表](https://blog.csdn.net/doniexun/article/details/45043297)
