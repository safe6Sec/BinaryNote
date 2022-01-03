## 说明

- 个人建议是搞完汇编和c之后再来了解pe
- 工具：die、peid、exeinfo、c32asm



## pe文件解析

### 基础知识

###### PE（Portable Execute）

PE文件是Windows下可执行文件的总称，常见的有DLL，EXE，OCX，SYS等，事实上，一个文件是否是PE文件与其扩展名无关，PE文件可以是任何扩展名。



PE文件的结构一般来说如下图所示：从起始位置开始依次是DOS头，NT头，节表以及具体的节。（**由下而上**）



![img](https://gitee.com/safe6/img/raw/master/906508_BF2RKFKB64ZKDG5.png)



![img](https://gitee.com/safe6/img/raw/master/906508_WCKUYZ2EAWCJ27Y.jpg)



- ###### 基地址(ImageBase)

  当PE文件通过Windows载入内存后，内存中的版本称为模块(Module)。映射文件的起始地址称为基地址(ImageBase)。也就是我们常说的基址。

  IDA加载的时候，会默认以00400000作为基地址，而调试器加载的时候，基地址由操作系统决定。也就是会出现动态分析的地址和静态分析的不一样。

  我们在IDA中选择 Edit->Segments->Rebase Program进行修改。

  ![image-20220102135923154](https://gitee.com/safe6/img/raw/master/image-20220102135923154.png)

- ###### 虚拟地址(Vitual Address,VA)

  PE文件被系统加载器映射到内存中，每个程序都有自己的虚拟空间，这个虚拟空间的内存地址称为虚拟地址(Vitual Address,VA)

- ###### 相对虚拟地址(Relative Vitual Address,RVA)

  RVA就是相对虚拟偏移，就是偏移地址。假设一个EXE文件从400000h处载入，它的代码区块开始于401000h处，代码区块的RVA计算方式如如下：

  `目标地址401000h - 载入地址400000h = RVA 1000h`

  将一个RVA转换成真实的地址，如下：

  `虚拟地址（VA）= 基地址（ImageBase）+相对虚拟地址（RVA）`

搞过外挂的老哥一定对基址+偏移不陌生。

- ###### 文件偏移地址

  FOA: 文件偏移，就是文件中所在的地址。当PE文件在磁盘中时，某个数据的位置相对于文件头的偏移量称为文件偏移地址（File Offset）或物理地址（RAW Offset）





### MS-DOS头

 DOS头主要就是为了兼容之前的DOS操作系统，DOS头后面便是DOS Stub，这两部分构成了MS-DOS可执行文件的基本要素，如果PE文件运行在DOS系统中便会执行Stub中的代码，一般上都是“Thisprogram cannot run in DOS mode ”。所以DOS没有什么可以讲的，主要就是开头两个字节MZ表明他是一个PE文件的DOS头开始。还有就是距离MZ偏移量为0x3C的内容便是PE头的位置。

```cpp
struct _IMAGE_DOS_HEADER {
*0x00 WORD e_magic;            //DOS可执行文件标记"MZ"(MS-DOS的创建者之一),5A4Dh
 0x02 WORD e_cblp;           
 0x04 WORD e_cp;               
 0x06 WORD e_crlc;               
 0x08 WORD e_cparhdr;           
 0x0a WORD e_minalloc;       
 0x0c WORD e_maxalloc;       
 0x0e WORD e_ss;
 0x10 WORD e_sp;               
 0x12 WORD e_csum;           
 0x14 WORD e_ip;                //DOS代码入口IP
 0x16 WORD e_cs;                //DOS代码入口IP
 0x18 WORD e_lfarlc;           
 0x1a WORD e_ovno;           
 0x1c WORD e_res[4];           
 0x24 WORD e_oemid;               
 0x26 WORD e_oeminfo;           
 0x28 WORD e_res2[10];           
*0x3c DWORD e_lfanew;        //指向PE文件头"PE",0,0。文件头的相对偏移（RVA）
};
```



![image-20220102143849001](https://gitee.com/safe6/img/raw/master/image-20220102143849001.png)



### PE文件头

紧跟着DOS头的是PE文件头（PE Header）。PE Header是PE相关结构NT映像头（_IMAGE_NT_HEADERS）的简称。PE装载器从_IMAGE_DOS_HEADERS的 e_lfanew字段里找到PEHeader的起始偏移量，用其加上基址，得到PE文件头的指针。

```
PNTHeader = ImageBase + dosHeader->e_lfanew
```

 _IMAGE_NT_HEADERS

```cpp
struct _IMAGE_NT_HEADERS {
0x00 DWORD Signature;                        //PE文件标识
0x04 _IMAGE_FILE_HEADER FileHeader;
0x18 _IMAGE_OPTIONAL_HEADER OptionalHeader;
};
```

![image-20220102144719850](https://gitee.com/safe6/img/raw/master/image-20220102144719850.png)

- _IMAGE_FILE_HEADER

  _IMAGE_FILE_HEADER指出了 _IMAGE_OPTIONAL_HEADER的大小

  ```cpp
  struct _IMAGE_FILE_HEADER {         //映像文件头
  0x00 WORD Machine;                    //运行平台
  0x02 WORD NumberOfSections;            //文件的区块数
  0x04 DWORD TimeDateStamp;            //文件创建日期和时间
  0x08 DWORD PointerToSymbolTable;    //指向符号表（用于调试）
  0x0c DWORD NumberOfSymbols;            //符号表中符号的个数（用于调试）
  0x10 WORD SizeOfOptionalHeader;        //_IMAGE_OPTIONAL_HEADER结构的大小
  0x12 WORD Characteristics;            //文件属性
  };
  ```

  ![image-20220102144739889](https://gitee.com/safe6/img/raw/master/image-20220102144739889.png)

此处打住，暂时不想研究的太细。

**数据目录表**

 数据目录表是PE文件中各种数据结构的索引目录，由多个IMAGE_DATA_DIRECTORY组成的数组。pe头的一部分。

![image-20220102150755977](https://gitee.com/safe6/img/raw/master/image-20220102150755977.png)



### 区段表

 PE文件头后面是区段表，用于描述各个区段的属性，文件至少拥有一个区段才能执行。区段表是多个相连的IMAGE_SECTION_HEADER结构组成。

![image-20220102151012512](https://gitee.com/safe6/img/raw/master/image-20220102151012512.png)



**区段名功能约定**

| 区段名 | 描述                                   |
| ------ | -------------------------------------- |
| .text  | 代码段，里面的数据全都是代码           |
| .data  | 可读写的数据段，存放全局变量或静态变量 |
| .rdata | 只读数据区                             |
| .idata | 导入数据区，存放导入表信息                       |
| .edata | 导出数据区，导出表信息                           |
| .rsrc  | 资源区段，存放程序用到的所有资源，如图表，菜单等 |
| .bss   | 未初始化数据区                                   |
| .crt    | 用于支持C++运行时库所添加的数据        |
| .tls    | 存储线程局部变量                       |
| .reloc  | 包含重定位信息                         |
| .sdata  | 包含相对于可被全局指针定位的可读写数据 |
| .srdata | 包含相对于可被全局指针定位的只读数据   |
| .pdata  | 包含异常表                             |
| .debug$S | 包含OBJ文件中的Codeview格式符号       |
| .debug$T | 包含OBJ文件中的Codeview格式类型的符号 |
| .debug$P | 包含使用预编译头时的一些信息          |
| .drectve | 包含编译时的一些链接命令              |
| .didat   | 包含延迟装入的数据                    |

#### 重点区段

![](https://gitee.com/safe6/img/raw/master/image-20220102153225821.png)

**导入表**

 导入表机制是指PE文件从第三方程序中导入API以提供本函数调用的机制。事实上Windows平台下所有系统提供的API函数都是通过导入导出表完成的。所以想要看程序调用了哪些函数就要看导入表。但是其实也可以手工来调用而不是直接调用，比如用LoadLibrary()等函数加载dll文件然后获得里面函数的指针，最终直接通过地址调用函数。

**两种方式：第一是直接导入api，进行调用。第二种是加载对应的库进内存，通过地址调用(基址+偏移)。后面免杀会用到。**

**导出表**

导出表是PE文件为其他应用程序提供API的一种函数导出方式。

导出表由名称表、函数表和序号表，后两者是必须要的，名称表则是可选的。名称表和序号表起到索引找到函数表中的函数的作用，而函数表则是记录函数地址的。然而导出表的序号顺序和函数顺序并不是对应的，在内存中的数据是被完全打乱的，函数的顺序是按照名称表来确定的。

**基址重定位**

 数据目录表中的IMAGE_DIRECTORY_ENTRY_BASERELOC结构指向重定位目录。由于dll文件并不一定可以装载到默认的基址，所以需要进行重定位。例如在dll文件中有一条确定地址的指令，这时就需要用到重定位，通常是加载多个**相同基地址的dll**会用到。这里不再细述，等以后用到再详细来看。



待补充













## References

- https://bbs.pediy.com/thread-262784.htm
- https://blog.csdn.net/wuyangbotianshi/article/details/17380835
- https://blog.csdn.net/wuyangbotianshi/article/details/17381113
- https://www.anquanke.com/post/id/262543
