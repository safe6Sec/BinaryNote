## 说明

win32真tm难顶啊。2022.1.2

- 我们学习win32不是学如何画图的，重点是win32api也就是各种dll里面的函数。
- 当发现看不懂如下的笔记时，就去找个win32教学视频看看。
- 文档https://docs.microsoft.com/zh-cn/windows/win32/



## Win32 API

Win32 API就是Windows操作系统提供给我们的函数（应用程序接口），其主要存放在C:\Windows\System32（存储的DLL是64位）、C:\Windows\SysWOW64（存储的DLL是32位）下面的所有DLL文件（几千个）。

重要的DLL文件：

1. Kernel32.dll：最核心的功能模块，例如内存管理、进程线程相关的函数等；
2. User32.dll：Windows用户界面相关的应用程序接口，例如创建窗口、发送信息等；
3. GDI32.dll：全称是Graphical Device Interface（图形设备接口），包含用于画图和显示文本的函数。

在C语言中我们想要使用Win32 API的话直接在代码中包含**windows.h**这个头文件即可。



## 编码基础

了解各种编码如UTF-8、Unicode、UTF-16。

win32里面新增了一堆类型其实都是之前学的基础类型的别名。也就是那一堆大写字母,如下。



字符类型：

```cpp
CHAR strBuff[] = "中国"; // char

WCHAR strBuff[] = L"中国"; // wchar_t

TCHAR strBuff[] = TEXT("中国"); // TCHAR 根据当前项目的编码自动选择char还是wchar_t，在Win32中推荐使用这种方式
```

字符串指针

```cpp
PSTR strPoint = "中国"; // char*

PWSTR strPoint = L"中国"; // wchar_t*

PTSTR strPoint = TEXT("中国"); // PTSTR 根据当前项目的编码自动选择如char*还是wchar_t*，在Win32中推荐使用这种方式
```



不知道怎么记、自己查api就可搞定了







