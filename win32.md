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
2. ntdll.dll：内核dll，更为底层属于ring0。ring3到ring0的入口。位于Kernel32.dll和user32.dll中的所有win32 API 最终都是调用ntdll.dll中的函数实现的
3. GDI32.dll：全称是Graphical Device Interface（图形设备接口），包含用于画图和显示文本的函数。



在C语言中我们想要使用Win32 API的话直接在代码中包含**windows.h**这个头文件即可。



## win32基础-编码和字符串

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



![](https://gitee.com/safe6/img/raw/master/20220107201645.png)



## 进程



创建进程，每个进程至少需要一个线程

进程创建的过程也就是**CreateProcess函数**

```c
BOOL CreateProcess(
    LPCTSTR lpApplicationName,                 // name of executable module 进程名（完整文件路径）
    LPTSTR lpCommandLine,                      // command line string 命令行传参
    LPSECURITY_ATTRIBUTES lpProcessAttributes, // SD 进程句柄
    LPSECURITY_ATTRIBUTES lpThreadAttributes,  // SD 线程句柄
    BOOL bInheritHandles,                      // handle inheritance option 句柄
    DWORD dwCreationFlags,                     // creation flags 标志
    LPVOID lpEnvironment,                      // new environment block 父进程环境变量
    LPCTSTR lpCurrentDirectory,                // current directory name 父进程目录作为当前目录，设置目录
    LPSTARTUPINFO lpStartupInfo,               // startup information 结构体详细信息（启动进程相关信息）
    LPPROCESS_INFORMATION lpProcessInformation // process information 结构体详细信息（进程ID、线程ID、进程句柄、线程句柄）
);
```



带参数创建一个cmd进程

```c
#include <windows.h>
#include <stdAfx.h>
 
int main(int argc, char* argv[])
{
    TCHAR childProcessName[] = TEXT("C:/WINDOWS/system32/cmd.exe");
    TCHAR childProcessCommandLine[] = TEXT(" /c ping 127.0.0.1");
 
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
 
    //ZeroMemory函数用0填充结构体数据
    ZeroMemory(&si, sizeof(si));
    ZeroMemory(&pi, sizeof(pi));
 
    //给si.cb成员赋值当前结构体大小（为什么需要？这是因为Windows会有很多个版本，便于未来更新换代
    si.cb = sizeof(si);
 
    if(CreateProcess(childProcessName, childProcessCommandLine, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi)) {
        printf("CreateProcess Successfully! \n");
    } else {
        printf("CreateProcess Error: %d \n", GetLastError());
    }
 
    //每个进程至少有一个线程，所以我们也要关闭线程
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);
    
    //system("pause");
    return 0;
}
```



**GetLastError函数来获取问题编号**，最近一次命令执行有错误会写到里面。

**CloseHandle这个函数**，它是用来关闭句柄的，暂时先不用管其原理，我们只要知道它所支持关闭就都是内核对象：



借助**dwCreationFlags**这个参数，将其修改为**CREATE_NEW_CONSOLE**即可在新窗口打开进程

改为**CREATE_SUSPENDED**，也就是以挂起的形式创建进程，挂起本质上**挂起的是线程**。可用ResumeThread函数恢复进程。

CreateProcess函数的第八个参数**LPCTSTR lpCurrentDirectory**。可以指定工作目录



只有进程才会有句柄表，并且每一个进程都会有一个句柄表。，我们得到所谓句柄的值实际上就是句柄表里的一个索引。



TerminateProcess函数来终止A进程,win32里面的H开头的结构体都是句柄

```c
// TerminateProcess函数
BOOL TerminateProcess(
  HANDLE hProcess, // handle to the process 句柄
  UINT uExitCode   // exit code for the process 退出代码
);
```



想要操作一个进程需要先打开进程，拿到句柄才能操作。

OpenProcess函数

，**这个函数是用来打开进程对象的**：

```c
HANDLE OpenProcess(
  DWORD dwDesiredAccess,  // access flag 你希望的访问权限
  BOOL bInheritHandle,    // handle inheritance option 是否可以被继承
  DWORD dwProcessId       // process identifier 进程ID
);
```



如下拿到句柄，终止一个进程

```c
HANDLE hProcess;
hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, 0x524);
 
if(!TerminateProcess(hProcess, 0)) {
    printf("终止进程失败：%d \n", GetLastError());
}
```



通过**GetModuleFileName**和**GetCurrentDirectory**函数可以分别获得当前模块目录和当前工作目录：

```c
char strModule[256];
GetModuleFileName(NULL,strModule, 256); // 得到当前模块目录，当前exe所在的路径，包含exe文件名
 
char strWork[1000];
GetCurrentDirectory(1000, strWork); // 获取当前工作目录
 
printf("模块目录：%s \n工作目录：%s \n", strModule, strWork);
```

![image-20220107213201778](https://gitee.com/safe6/img/raw/master/image-20220107213201778.png)



进程api

获取当前进程ID（PID）：GetCurrentProcessId

获取当前进程句柄：GetCurrentProcess

获取命令行：GetCommandLine

获取启动信息：GetStartupInfo

遍历进程ID：EnumProcesses

快照：CreateToolhelp32Snapshot



## 线程

使用CreateThread函数，其语法格式如下：

```c
HANDLE CreateThread( // 返回值是线程句柄
  LPSECURITY_ATTRIBUTES lpThreadAttributes, // SD 安全属性，包含安全描述符
  SIZE_T dwStackSize,                       // initial stack size 初始堆栈
  LPTHREAD_START_ROUTINE lpStartAddress,    // thread function 线程执行的函数代码
  LPVOID lpParameter,                       // thread argument 线程需要的参数
  DWORD dwCreationFlags,                    // creation option 标识，也可以以挂起形式创建线程
  LPDWORD lpThreadId                        // thread identifier 返回当前线程ID
);
```





