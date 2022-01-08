## 说明

win32真tm难顶啊。2022.1.2

- 我们学习win32不是学如何画图的，学习win32api其实也就是学使用windows系统的各种dll里面的函数。
- 学习win32api和c需要思想上的转变，从面向对象转到面向过程。这样才能更好的理解。
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

1. 线程是附属在进程上的执行实体，是代码的执行流程；
2. 一个进程可以包含多个线程（**一个进程至少要包含一个线程，进程是空间上的概念，线程是时间上的概念**）。

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



尝试创建一个线程执行for循环

固定语法，需要一个proc。如同java语法需要继承thread一样。

```c
#include <windows.h>
 
// 线程执行的函数有语法要求，参考MSDN Library
DWORD WINAPI ThreadProc(LPVOID lpParameter) {
    // 要执行的代码
    for(int i = 0; i < 100; i++) {
        Sleep(500);
        printf("++++++thread is runing %d \n", i);
    }
 
    return 0;
}
 
int main(int argc, char* argv[])
{
    // 创建线程
    CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
 
    // 要执行的代码
    for(int i = 0; i < 100; i++) {
        Sleep(500);
        printf("------main is runing %d \n", i);
    }
    return 0;
    
}
```

传递参数

```c
#include <windows.h>
 
// 线程执行的函数有语法要求，参考MSDN Library
DWORD WINAPI ThreadProc(LPVOID lpParameter) {
    
    int* aa= (*int)lpParameter;
    // 要执行的代码
    for(int i = 0; i < 100; i++) {
        Sleep(500);
        printf("++++++thread is runing %d \n", i);
        printf("++++++thread is runing %d \n", *a);
    }
 
    return 0;
}
 
int main(int argc, char* argv[])
{
    int a=10;
    // 创建线程
    CreateThread(NULL, NULL, ThreadProc, (LPVOID)&a, 0, NULL);
 
    // 要执行的代码
    for(int i = 0; i < 100; i++) {
        Sleep(500);
        printf("------main is runing %d \n", i);
    }
    return 0;
    
}
```



SuspendThread函数

SuspendThread函数用于暂停（挂起）某个线程，当暂停后该线程不会占用CPU，其语法格式很简单，只需要传入一个线程句柄即可：



```
DWORD SuspendThread(
  HANDLE hThread   // handle to thread
);
```



ResumeThread函数

ResumeThread函数用于恢复被暂停（挂起）的线程，其语法格式也很简单，只需要传入一个线程句柄即可：

```
DWORD ResumeThread(
  HANDLE hThread   // handle to thread
);
```

需要注意的是，挂起几次就要恢复几次。

```
SuspendThread(hThread);
SuspendThread(hThread);
```

```
ResumeThread(hThread);
ResumeThread(hThread);
```





等待线程结束

**WaitForSingleObject函数**重点函数

WaitForSingleObject函数用于等待**一个内核对象**状态发生变更，那也就是执行结束之后，才会继续向下执行，其语法格式如下：



```
DWORD WaitForSingleObject(
  HANDLE hHandle,        // handle to object 句柄
  DWORD dwMilliseconds   // time-out interval 等待超时时间（毫秒）
);
```

如果你想一直等待的话，可以将第二参数的值设置为**INFINITE**。

```x
HANDLE hThread;
hThread = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
WaitForSingleObject(hThread, INFINITE);
printf("OK...");
```



WaitForMultipleObjects函数

WaitForMultipleObjects函数与WaitForSingleObject函数**作用是一样**的，只不过它可以等待**多个内核对象的状态**发生变更，其语法格式如下：

```c
DWORD WaitForMultipleObjects(
  DWORD nCount,             // number of handles in array 内核对象的数量
  CONST HANDLE *lpHandles,  // object-handle array 内核对象的句柄数组
  BOOL bWaitAll,            // wait option 等待模式
  DWORD dwMilliseconds      // time-out interval 等待超时时间（毫秒）
);
```

第三个参数等待模式的值是布尔类型，一个是TRUE，一个是FALSE，TRUE就是等待所有对象的所有状态发生变更，FALSE则是等待任意一个对象的状态发生变更。

```c
HANDLE hThread[2];
hThread[0] = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
hThread[1] = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
WaitForMultipleObjects(2, hThread, TRUE, INFINITE);
```



GetExitCodeThread函数

线程函数会有一个返回值（DWORD）这时候就可以使用**GetExitCodeThread**函数

```c
HANDLE hThread;
hThread = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
 
WaitForSingleObject(hThread, INFINITE);
 
DWORD exitCode;
GetExitCodeThread(hThread, &exitCode);
 
printf("Exit Code: %d \n", exitCode);
```

线程上下文

**线程上下文**是指某一时间点CPU寄存器和程序计数器的内容，如果想要设置、获取线程上下文就需要先将线程**挂起**。



GetThreadContext函数

GetThreadContext函数用于获取线程上下文，其语法格式如下：

```c
BOOL GetThreadContext(
  HANDLE hThread,       // handle to thread with context 句柄
  LPCONTEXT lpContext   // context structure
);
```



当我们将CONTEXT结构体的ContextFlags成员的值设置为CONTEXT_INTEGER时则可以获取edi、esi、ebx、edx、ecx、eax这些寄存器的值：

```c
HANDLE hThread;
hThread = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
 
SuspendThread(hThread);
 
CONTEXT c;
c.ContextFlags = CONTEXT_INTEGER;
GetThreadContext(hThread, &c);
 
printf("%x %x \n", c.Eax, c.Ecx);
```



SetThreadContext函数

GetThreadContext函数是个设置修改线程上下文，其语法格式如下：

```c
BOOL SetThreadContext(
  HANDLE hThread,            // handle to thread
  CONST CONTEXT *lpContext   // context structure
);
```

我们可以尝试修改Eax，然后再获取：

```c
HANDLE hThread;
hThread = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
 
SuspendThread(hThread);
 
CONTEXT c;
c.ContextFlags = CONTEXT_INTEGER;
c.Eax = 0x123;
SetThreadContext(hThread, &c);
 
CONTEXT c1;
c1.ContextFlags = CONTEXT_INTEGER;
GetThreadContext(hThread, &c1);
 
printf("%x \n", c1.Eax);
```



临界区（加锁）

### 线程锁

线程锁就是临界区的实现方式，通过线程锁我们可以完美解决如上所述的问题，其步骤如下所示：

1. 创建全局变量：CRITICAL_SECTION cs;
2. 初始化全局变量：InitializeCriticalSection(&cs);
3. 实现临界区：进入 → EnterCriticalSection(&cs); 离开 → LeaveCriticalSection(&cs);

```c
#include <windows.h>
 
CRITICAL_SECTION cs; // 创建全局变量
int countNumber = 10;
 
DWORD WINAPI ThreadProc(LPVOID lpParameter) {
    while (1) {
        EnterCriticalSection(&cs); // 构建临界区，获取令牌
        if (countNumber > 0) {
            printf("Thread: %d\n", *((int*)lpParameter));
            printf("Sell num: %d\n", countNumber);
            // 售出-1
            countNumber--;
            printf("Count: %d\n", countNumber);
        } else {
            LeaveCriticalSection(&cs); // 离开临临界区，归还令牌
            break;  
        }
        LeaveCriticalSection(&cs); // 离开临临界区，归还令牌
    }
    
    return 0;
}
 
int main(int argc, char* argv[])
{
 
    InitializeCriticalSection(&cs); // 使用之前进行初始化
    
    int a = 1;
    HANDLE hThread;
    hThread = CreateThread(NULL, NULL, ThreadProc, (LPVOID)&a, 0, NULL);
    
    int b = 2;
    HANDLE hThread1;
    hThread1 = CreateThread(NULL, NULL, ThreadProc, (LPVOID)&b, 0, NULL);
 
    CloseHandle(hThread);
 
    getchar();
    return 0;
}
```



### 互斥体

针对内核对象的锁

内核级的临界资源（**内核对象：线程、文件、进程...**）



**CreateMutex**

创建互斥体的函数为**CreateMutex**，

```c
HANDLE CreateMutex(
  LPSECURITY_ATTRIBUTES lpMutexAttributes,  // SD 安全属性，包含安全描述符
  BOOL bInitialOwner,                       // initial owner 是否希望互斥体创建出来就有信号，或者说就可以使用，如果希望的话就为FALSE；官方解释为如果该值为TRUE则表示当前进程拥有该互斥体所有权
  LPCTSTR lpName                            // object name 互斥体的名字
);
```



互斥体相当于一个开关


```c
#include <windows.h>
 
int main(int argc, char* argv[])
{
    // 创建互斥体
    HANDLE cm = CreateMutex(NULL, FALSE, "XYZ");
    // 等待互斥体状态发生变化，也就是有信号或为互斥体拥有者，获取令牌
    WaitForSingleObject(cm, INFINITE);
 
    // 操作资源
    for (int i = 0; i < 5; i++) {
        printf("Process: A Thread: B -- %d \n", i);
        Sleep(1000);
    }
    // 释放令牌
    ReleaseMutex(cm);
    return 0;
}
```

互斥体和线程锁的区别

1. 线程锁只能用于单个进程间的线程控制
2. 互斥体可以设定等待超时，但线程锁不能
3. 线程意外结束时，互斥体可以避免无限等待
4. 互斥体效率没有线程锁高



利用互斥体，防止多开

**如果函数CreateMutex成功，返回值是一个指向mutex对象的句柄；如果命名的mutex对象在函数调用前已经存在，函数返回现有对象的句柄，GetLastError返回ERROR_ALREADY_EXISTS（表示互斥体以及存在）；否则，调用者创建该mutex对象；如果函数失败，返回值为NULL，要获得扩展的错误信息，请调用GetLastError获取**。

```c
#include <windows.h>
 
int main(int argc, char* argv[])
{
    // 创建互斥体
    HANDLE cm = CreateMutex(NULL, TRUE, "XYZ");
    // 判断互斥体是否创建失败
    if (cm != NULL) {
        // 判断互斥体是否已经存在，如果存在则表示程序被多次打开
        if (GetLastError() == ERROR_ALREADY_EXISTS) {
            printf("该程序已经开启了，请勿再次开启！");
            getchar();
        } else {
            // 等待互斥体状态发生变化，也就是有信号或为互斥体拥有者，获取令牌
            WaitForSingleObject(cm, INFINITE);
            // 操作资源
            for (int i = 0; i < 5; i++) {
                printf("Process: A Thread: B -- %d \n", i);
                Sleep(1000);
            }
            // 释放令牌
            ReleaseMutex(cm);
        }
    } else {
        printf("CreateMutex 创建失败! 错误代码: %d\n", GetLastError());
    }
    
    return 0;
}
```



## 事件

事件本身也是一种内核对象，其也是是用来控制线程的。



### 通知类型

事件本身可以做为通知类型来使用，创建事件使用函数**CreateEvent**，其语法格式如下：

```c
HANDLE CreateEvent(
  LPSECURITY_ATTRIBUTES lpEventAttributes, // SD 安全属性，包含安全描述符
  BOOL bManualReset,                       // reset type 如果你希望当前事件类型是通知类型则写TRUE，反之FALSE
  BOOL bInitialState,                      // initial state 初始状态，决定创建出来时候是否有信号，有为TRUE，没有为FALSE
  LPCTSTR lpName                           // object name 事件名字
);
```



如下展示通知事件使用。

有点线程锁的味道，通知。

```c
#include <windows.h>
 
HANDLE e_event;
 
DWORD WINAPI ThreadProc(LPVOID lpParameter) {
    // 等待事件
    WaitForSingleObject(e_event, INFINITE);
    printf("ThreadProc - running ...\n");
    getchar();
    return 0;
}
 
DWORD WINAPI ThreadProcB(LPVOID lpParameter) {
    // 等待事件
    WaitForSingleObject(e_event, INFINITE);
    printf("ThreadProcB - running ...\n");
    getchar();
    return 0;
}
 
int main(int argc, char* argv[])
{
 
    // 创建事件
    // 第二个参数，FALSE表示非通知类型通知，也就是互斥；TRUE则表示为通知类型
    // 第三个参数表示初始状态没有信号
    e_event = CreateEvent(NULL, TRUE, FALSE, NULL);
 
    // 创建2个线程
    HANDLE hThread[2];
    hThread[0] = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
    hThread[1] = CreateThread(NULL, NULL, ThreadProcB, NULL, 0, NULL);
    
    // 设置事件为已通知，也就是设置为有信号
    SetEvent(e_event);
 
    // 等待线程执行结束，销毁内核对象
    WaitForMultipleObjects(2, hThread, TRUE, INFINITE);
    CloseHandle(hThread[0]);
    CloseHandle(hThread[1]);
    // 事件类型也是内核对象，所以也需要关闭句柄
    CloseHandle(e_event);
 
    return 0;
}
```



### 事件互斥



每个线程里面都需要等待有事件信号，才开始工作。生产者先生成才能消费。所以一开始就把生产者通知设为有信号。

```c
#include <windows.h>
 
// 容器
int container = 0;
 
// 次数
int count = 10;
 
// 事件
HANDLE eventA;
HANDLE eventB;
 
// 生产者
DWORD WINAPI ThreadProc(LPVOID lpParameter) {
    for (int i = 0; i < count; i++) {
        // 等待事件，修改事件A状态
        WaitForSingleObject(eventA, INFINITE);
        // 获取当前进程ID
        int threadId = GetCurrentThreadId();
        // 生产存放进容器
        container = 1;
        printf("Thread: %d, Build: %d \n", threadId, container);
        // 给eventB设置信号
        SetEvent(eventB);
    }
    return 0;
}
 
// 消费者
DWORD WINAPI ThreadProcB(LPVOID lpParameter) {
    for (int i = 0; i < count; i++) {
        // 等待事件，修改事件B状态
        WaitForSingleObject(eventB, INFINITE);
        // 获取当前进程ID
        int threadId = GetCurrentThreadId();
        printf("Thread: %d, Consume: %d \n", threadId, container);
        // 消费
        container = 0;
        // 给eventA设置信号
        SetEvent(eventA);
    }
    return 0;
}
 
int main(int argc, char* argv[])
{
    // 创建事件
    // 线程同步的前提是互斥
    // 顺序按照先生产后消费，所以事件A设置信号，事件B需要通过生产者线程来设置信号
    // 
    eventA = CreateEvent(NULL, FALSE, TRUE, NULL);
    eventB = CreateEvent(NULL, FALSE, FALSE, NULL);
 
    // 创建2个线程
    HANDLE hThread[2];
    hThread[0] = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
    hThread[1] = CreateThread(NULL, NULL, ThreadProcB, NULL, 0, NULL);
 
    WaitForMultipleObjects(2, hThread, TRUE, INFINITE);
    CloseHandle(hThread[0]);
    CloseHandle(hThread[1]);
    // 事件类型也是内核对象，所以也需要关闭句柄
    CloseHandle(eventA);
    CloseHandle(eventB);
 
    return 0;
}
```





## 窗口

之前我们所学习的进程、线程之类的函数，其接口来源于kernel32.dll → ntoskrnl.exe；而我们要学习的图形化界面的接口，它就来源于user32.dll、gdi32.dll → win32k.sys。

user32.dll和gdi32.dll的区别在哪呢？**前者是你想使用Windows已经画好的界面就用它，我们称之为GUI编程；后者是你想自己画一个界面，例如你要画一朵花，那么就使用后者，因为这涉及到绘图相关的内容，我们称之为GDI编程**。



在图形界面中有一个**新的句柄**，其叫HWND，win32k.sys提供在内核层创建图形化界面，我们想要在应用层调用就需要对应的句柄HWND，**而这个句柄表是全局的，并且只有一个**。



### 消息

开始写窗口之前，需要先学习一下消息。

当我们点击鼠标的时候，或者当我们按下键盘的时候，操作系统都要把这些动作记录下来，存储到一个结构体中，这个**结构体**就是消息。

**有点像是其它语言里面的事件，如点击事件。**



#### 消息队列

每个线程只有一个消息队列。







#### WinMain函数

控制台程序是从Main函数为入口开始执行的，而Win32窗口程序是从WinMain函数开始执行的。

Win32窗口程序的入口函数

```
int WINAPI WinMain(
  HINSTANCE hInstance,      // handle to current instance
  HINSTANCE hPrevInstance,  // handle to previous instance 
  LPSTR lpCmdLine,          // command line
  int nCmdShow              // show state
);
```

1. HINSTANCE hInstance，这是一个句柄，在Win32中H开头的通常都是句柄，这里的HINSTANCE是指向模块的句柄，实际上这个值就是模块在进程空间内的内存地址；
2. HINSTANCE hPrevInstance，该参数永远为空NULL，无需理解；
3. 第三、第四个参数（LPSTR lpCmdLine、int nCmdShow）是由CreateProcess的LPTSTR lpCommandLine、LPSTARTUPINFO lpStartupInfo参数传递的。

这就是为什么od调试的时候，main函数之前为什么会有这些调用了。



由于不是控制台程序了，所以打印也得换了。我们在窗口程序中想要输出信息就不可以使用printf了，我们可以使用另外一个函数OutputDebugString



```c
void OutputDebugString(
  LPCTSTR lpOutputString
);
```



，但是需要注意的是这个函数只能打印固定字符串，不能打印格式化的字符串，所以如果需要格式化输出，需要在这之前使用sprintf函数进行格式化，这里我们可以尝试输出当前模块的句柄：

```c
#include "stdafx.h"
 
int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
    // TODO: Place code here.
    DWORD dwAddr = (DWORD)hInstance;
    
    char szOutBuff[0x80];
    sprintf(szOutBuff, "hInstance address: %x \n", dwAddr); // 该函数需要包含stdio.h头文件
    OutputDebugString(szOutBuff);
 
    return 0;
}
```





### 第一个窗体



创建窗口之前需要先注册到系统里面，默认系统已经给我们注册好很多控件，如按钮。注册完之后的窗口才能通过name进行使用



````c
// Windows.cpp : Defines the entry point for the application.
//
 
#include "stdafx.h"
#include <stdio.h>
 
// 窗口函数定义
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    // 必须要调用一个默认的消息处理函数，关闭、最小化、最大化都是由默认消息处理函数处理的
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}
 
int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
    char szOutBuff[0x80];
 
    // 1. 定义创建的窗口(创建注册窗口类)
    TCHAR className[] = TEXT("My First Window");
    WNDCLASS wndClass = {0};
    // 设置窗口背景色
    wndClass.hbrBackground = (HBRUSH)COLOR_BACKGROUND;
    // 设置类名字
    wndClass.lpszClassName = className;
    // 设置模块地址
    wndClass.hInstance = hInstance;
    // 处理消息的窗口函数
    wndClass.lpfnWndProc = WindowProc; // 不是调用函数，只是告诉操作系统，当前窗口对应的窗口回调函数是什么
    // 注册窗口类
    RegisterClass(&wndClass);
 
    // 2. 创建并显示窗口
    // 创建窗口
    /*
    CreateWindow 语法格式：
    HWND CreateWindow(
        LPCTSTR lpClassName,  // registered class name 类名字
        LPCTSTR lpWindowName, // window name 窗口名字
        DWORD dwStyle,        // window style 窗口外观的样式
        int x,                // horizontal position of window 相对于父窗口x坐标
        int y,                // vertical position of window 相对于父窗口y坐标
        int nWidth,           // window width 窗口宽度：像素
        int nHeight,          // window height 窗口长度：像素
        HWND hWndParent,      // handle to parent or owner window 父窗口句柄
        HMENU hMenu,          // menu handle or child identifier 菜单句柄
        HINSTANCE hInstance,  // handle to application instance 模块
        LPVOID lpParam        // window-creation data  附加数据
    );
    */
    HWND hWnd = CreateWindow(className, TEXT("窗口"), WS_OVERLAPPEDWINDOW, 10, 10, 600, 300, NULL, NULL, hInstance, NULL);
 
    if (hWnd == NULL) {
        // 如果为NULL则窗口创建失败，输出错误信息
        sprintf(szOutBuff, "Error: %d", GetLastError());
        OutputDebugString(szOutBuff);
        return 0;
    }
 
    // 显示窗口
    /*
    ShowWindow 语法格式：
    BOOL ShowWindow(
        HWND hWnd,     // handle to window 窗口句柄
        int nCmdShow   // show state 显示的形式
    );
    */
    ShowWindow(hWnd, SW_SHOW);
 
    // 3. 接收消息并处理

    MSG msg;
    BOOL bRet;
    while( (bRet = GetMessage( &msg, NULL, 0, 0 )) != 0)
    { 
        if (bRet == -1)
        {
            // handle the error and possibly exit
            sprintf(szOutBuff, "Error: %d", GetLastError());
            OutputDebugString(szOutBuff);
            return 0;
        }
        else
        {
            // 转换消息
            TranslateMessage(&msg);
            // 分发消息：就是给系统调用窗口处理函数
            DispatchMessage(&msg);
        }
    }
 
    return 0;
}
````



一个窗口程序必须要包含，消息处理回调函数和接收消息代码。然后定义窗口、注册窗口、创建窗口、显示窗口。



WindowProc就是消息回调，里面用来处理各种消息。



如下处理键盘按下

```c
/ 窗口函数定义
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch(uMsg) {
    // 当键盘按下则处理
    case WM_KEYDOWN:
        {
            char szOutBuff[0x80];
            sprintf(szOutBuff, "keycode: %x \n", wParam);
            OutputDebugString(szOutBuff);
            break;
        }
    }
 
    // 必须要调用一个默认的消息处理函数，关闭、最小化、最大化都是由默认消息处理函数处理的
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}
```





### 子窗口控件（控件）

关于子窗口控件

1. Windows提供了几个预定义的窗口类以方便我们的使用，我们一般叫它们为子窗口控件，简称控件；
2. 控件会自己处理消息，并在自己状态发生改变时通知父窗口；
3. 预定义的控件有：按钮、复选框、编辑框、静态字符串标签和滚动条等。



```c
#include "stdafx.h"
// 定义子窗口标识
#define CWA_EDIT 0x100
#define CWA_BUTTON_0 0x101
#define CWA_BUTTON_1 0x102
 
// 定义全局模块
HINSTANCE gHinstance;
 
 
// 窗口函数定义
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    // 当键盘按下则处理
    case WM_CHAR:
        {
            char szOutBuff[0x80];
            sprintf(szOutBuff, "keycode: %c \n", wParam);
            OutputDebugString(szOutBuff);
            break;
        }
    // 当窗口创建则开始创建子窗口控件
    case WM_CREATE:
        {
            // 创建编辑框
            CreateWindow(
                TEXT("EDIT"),  // registered class name 注册的类名，使用EDIT则为编辑框
                TEXT(""), // window name 窗口名称
                WS_CHILD | WS_VISIBLE | WS_VSCROLL | ES_MULTILINE,        // window style 子窗口控件样式：子窗口、创建后可以看到、滚动条、自动换行
                0,                // horizontal position of window 在父窗口上的x坐标
                0,                // vertical position of window 在父窗口上的y坐标
                400,           // window width 控件宽度
                300,          // window height 控件高度
                hwnd,      // menu handle or child identifier 父窗口句柄
                (HMENU)CWA_EDIT,          // menu handle or child identifier 子窗口标识
                gHinstance,  // handle to application instance 模块
                NULL        // window-creation data 附加数据
            );
 
            // 创建"设置"按钮
            CreateWindow(
                TEXT("BUTTON"),  // registered class name 注册的类名，使用BUTTON则为按钮
                TEXT("设置"), // window name 按钮名称
                WS_CHILD | WS_VISIBLE,        // window style 子窗口控件样式：子窗口、创建后可以看到
                450,                // horizontal position of window 在父窗口上的x坐标
                150,                // vertical position of window 在父窗口上的y坐标
                80,           // window width 控件宽度
                20,          // window height 控件高度
                hwnd,      // menu handle or child identifier 父窗口句柄
                (HMENU)CWA_BUTTON_0,          // menu handle or child identifier 子窗口标识
                gHinstance,  // handle to application instance 模块
                NULL        // window-creation data 附加数据
            );
 
            // 创建"获取"按钮
            CreateWindow(
                TEXT("BUTTON"),  // registered class name 注册的类名，使用BUTTON则为按钮
                TEXT("获取"), // window name 按钮名称
                WS_CHILD | WS_VISIBLE,        // window style 子窗口控件样式：子窗口、创建后可以看到
                450,                // horizontal position of window 在父窗口上的x坐标
                100,                // vertical position of window 在父窗口上的y坐标
                80,           // window width 控件宽度
                20,          // window height 控件高度
                hwnd,      // menu handle or child identifier 父窗口句柄
                (HMENU)CWA_BUTTON_1,          // menu handle or child identifier 子窗口标识
                gHinstance,  // handle to application instance 模块
                NULL        // window-creation data 附加数据
            );
 
            break;
        }
    // 当按钮点击则处理
    case WM_COMMAND:
        {
            // 宏WM_COMMAND中，wParam参数的低16位中有标识，根据标识我们才能判断哪个按钮和编辑框，使用LOWORD()可以获取低16位
            switch (LOWORD(wParam)) {
            // 当按钮为设置
            case CWA_BUTTON_0:
                {
                    // SetDlgItemText函数修改编辑框内容
                    SetDlgItemText(hwnd, (int)CWA_EDIT, TEXT("HACK THE WORLD"));
                    break;
                }
            // 当按钮为获取
            case CWA_BUTTON_1:
                {
                    // MessageBox弹框输出编辑框内容
                    TCHAR szEditBuffer[0x80];
                    GetDlgItemText(hwnd, (int)CWA_EDIT, szEditBuffer, 0x80);
                    MessageBox(NULL, szEditBuffer, NULL, NULL);
                    break;
                }
            }
            break;
        }
    }
 
    // 必须要调用一个默认的消息处理函数，关闭、最小化、最大化都是由默认消息处理函数处理的
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}
 
int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
    char szOutBuff[0x80];
 
    // 1. 定义创建的窗口(创建注册窗口类)
    TCHAR className[] = TEXT("My First Window");
    WNDCLASS wndClass = {0};
    // 设置窗口背景色
    wndClass.hbrBackground = (HBRUSH)COLOR_BACKGROUND;
    // 设置类名字
    wndClass.lpszClassName = className;
    // 设置模块地址
    gHinstance = hInstance;
    wndClass.hInstance = hInstance;
    // 处理消息的窗口函数
    wndClass.lpfnWndProc = WindowProc; // 不是调用函数，只是告诉操作系统，当前窗口对应的窗口回调函数是什么
    // 注册窗口类
    RegisterClass(&wndClass);
 
    // 2. 创建并显示窗口
    // 创建窗口

    HWND hWnd = CreateWindow(className, TEXT("窗口"), WS_OVERLAPPEDWINDOW, 10, 10, 600, 300, NULL, NULL, hInstance, NULL);
 
    if (hWnd == NULL) {
        // 如果为NULL则窗口创建失败，输出错误信息
        sprintf(szOutBuff, "Error: %d", GetLastError());
        OutputDebugString(szOutBuff);
        return 0;
    }
 
    // 显示窗口
    /*
    ShowWindow 语法格式：
    BOOL ShowWindow(
        HWND hWnd,     // handle to window 窗口句柄
        int nCmdShow   // show state 显示的形式
    );
    */
    ShowWindow(hWnd, SW_SHOW);
 
    // 3. 接收消息并处理
    /*
    GetMessage 语法格式：
    BOOL GetMessage(
        LPMSG lpMsg,         // message information OUT类型参数，这是一个指针
        // 后三个参数都是过滤条件
        HWND hWnd,           // handle to window 窗口句柄，如果为NULL则表示该线程中的所有消息都要
        UINT wMsgFilterMin,  // first message 第一条信息
        UINT wMsgFilterMax   // last message 最后一条信息
    );
    */
    MSG msg;
    BOOL bRet;
    while( (bRet = GetMessage( &msg, NULL, 0, 0 )) != 0)
    { 
        if (bRet == -1)
        {
            // handle the error and possibly exit
            sprintf(szOutBuff, "Error: %d", GetLastError());
            OutputDebugString(szOutBuff);
            return 0;
        }
        else
        {
            // 转换消息
            TranslateMessage(&msg);
            // 分发消息：就是给系统调用窗口处理函数
            DispatchMessage(&msg);
        }
    }
 
    return 0;
}
```





## 内存

每一个物理内存的大小是4KB，**按照4KB大小来分页（Page）**，就有物理页这个概念。



虚拟内存地址划分

每个进程都有4GB的虚拟内存，虚拟内存的地址是如何划分的？首先，我们需要知道一个虚拟内存分为**高2G、低2G**。

但是需要注意的是低2G的用户空间使用还有**前64KB的空指针赋值区和后64KB的用户禁入区是我们目前不能使用的**。

![](https://gitee.com/safe6/img/raw/master/20220108183105.png)

申请内存

1. 私有内存通过**VirtualAlloc/VirtualAllocEx函数**申请，这两个函数在底层实现是没有区别的，但是后者是可以在其他进程中申请内存。
2. 共享内存通过**CreateFileMapping函数**映射



VirtualAlloc

申请内存的函数是**VirtualAlloc**，其语法格式如下：

```c
LPVOID VirtualAlloc(
  LPVOID lpAddress,        // region to reserve or commit 要分配的内存区域的地址，没有特殊需求通常不指定
  SIZE_T dwSize,           // size of region 分配的大小，一个物理页大小是0x1000（4KB），看你需要申请多少个物理页就乘以多少
  DWORD flAllocationType,  // type of allocation 分配的类型，常用的是MEM_COMMIT（占用线性地址，也需要物理内存）和MEM_RESERVE（占用线性地址，但不需要物理内存）
  DWORD flProtect          // type of access protection 该内存的初始保护属性
);
```

释放函数为**VirtualFree**

```c
BOOL VirtualFree(
  LPVOID lpAddress,   // address of region 内存区域的地址
  SIZE_T dwSize,      // size of region 内存大小
  DWORD dwFreeType    // operation type 如何释放，释放的类型，一共有两个类型：MEM_DECOMMIT（释放物理内存，但线性地址保留）、MEM_RELEASE（释放物理内存，释放线性地址，使用这个设置的时候内存大小就必须为0）
);
```



demo

```
LPVOID pm = VirtualAlloc(NULL, 0x1000*2, MEM_COMMIT, PAGE_READWRITE);
VirtualFree(pm, 0, MEM_RELEASE);
```



#### 共享内存



**CreateFileMapping函数**映射，该函数语法格式如下：

```c
HANDLE CreateFileMapping( // 内核对象，这个对象可以为我们准备物理内存，还可以将文件映射到物理页
  HANDLE hFile,                       // handle to file 文件句柄，如果不想将文件映射到物理页，则不指定该参数
  LPSECURITY_ATTRIBUTES lpAttributes, // security 安全属性，包含安全描述符
  DWORD flProtect,                    // protection 保护模式，物理页的属性
  DWORD dwMaximumSizeHigh,            // high-order DWORD of size 高32位，在32位计算机里通常设置为空
  DWORD dwMaximumSizeLow,             // low-order DWORD of size 低32位，指定物理内存的大小
  LPCTSTR lpName                      // object name 对象名字，公用时写，自己使用则可以不指定
);
```



该函数的作用就是为我们准备好物理内存（物理页），但是创建好了并不代表就可以使用了，我们还需要通过**MapViewOffile函数**将物理页与线性地址进行映射，**MapViewOffile函数**语法格式如下：

```c
LPVOID MapViewOfFile(
  HANDLE hFileMappingObject,   // handle to file-mapping object file-mapping对象的句柄
  DWORD dwDesiredAccess,       // access mode 访问模式(虚拟内存的限制必须比物理地址更加严格)
  DWORD dwFileOffsetHigh,      // high-order DWORD of offset 高32位，在32位计算机里通常设置为空
  DWORD dwFileOffsetLow,       // low-order DWORD of offset 低32位，指定从哪里开始映射
  SIZE_T dwNumberOfBytesToMap  // number of bytes to map 共享内存的大小，一般与物理页大小一致
);
```



demo

```c
#include <windows.h>
 
#define MapFileName "共享内存"
#define BUF_SIZE 0x1000
HANDLE g_hMapFile;
LPTSTR g_lpBuff;
 
int main(int argc, char* argv[])
{
    // 内核对象：准备好物理页，无效句柄值-1、物理页可读写、申请一个物理页
    g_hMapFile = CreateFileMapping(INVALID_HANDLE_VALUE, NULL, PAGE_READWRITE, 0, BUF_SIZE, MapFileName);
    // 将物理页与线性地址进行映射
    g_lpBuff = (LPTSTR)MapViewOfFile(g_hMapFile, FILE_MAP_ALL_ACCESS, 0, 0, BUF_SIZE);
 
    // 向物理内存中存储
    *(PDWORD)g_lpBuff = 0x12345678;
    
    // 关闭映射，关闭映射则表示释放了线形地址，但是物理页还存在
    UnmapViewOfFile(g_lpBuff);
    // 关闭句柄，这样才能释放物理页，但需要等待物理页使用完毕才会真正的释放，这里只是告诉系统我们当前进程不使用该句柄（物理页）罢了
    CloseHandle(g_hMapFile);
    return 0;
}
```





## 文件操作



这块内容和其它语言差不多，不多记录。



内存映射文件读写

之前我们学习过用CreateFileMapping函数来创建共享内存，这个函数同样也可以将文件映射到物理页，只不过在这之前我们需要传递一个文件句柄。

如下代码我们写了一个读取文件最开始第一个字节的值：

```c
DWORD MappingFile(LPSTR lpcFile) {
    HANDLE hFile;
    HANDLE hMapFile;
    LPVOID lpAddr;
    
    // 1. 创建文件（获取文件句柄）
    hFile = CreateFile(lpcFile, GENERIC_READ|GENERIC_WRITE, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
    
    // 判断CreateFile是否执行成功
    if(hFile == NULL) {
        printf("CreateFile failed: %d \n", GetLastError());
        return 0;
    }
    
    // 2. 创建FileMapping对象
    hMapFile = CreateFileMapping(hFile, NULL, PAGE_READWRITE, 0, 0, NULL);
        
    // 判断CreateFileMapping是否执行成功
    if(hMapFile == NULL) {
        printf("CreateFileMapping failed: %d \n", GetLastError());
        return 0;
    }
 
    // 3. 物理页映射到虚拟内存
    lpAddr = MapViewOfFile(hMapFile, FILE_MAP_COPY, 0, 0, 0);
 
    // 4. 读取文件
    DWORD dwTest1 = *(LPDWORD)lpAddr; // 读取最开始的4字节
    printf("dwTest1: %x \n", dwTest1);
    // 5. 写文件
    // *(LPDWORD)lpAddr = 0x12345678;
    
    // 6. 关闭资源
    UnmapViewOfFile(lpAddr);
    CloseHandle(hFile);
    CloseHandle(hMapFile);
    return 0;
}
```



### dll

DLL文件的入口函数是**DllMain函数**



```c
BOOL WINAPI DllMain(
  HINSTANCE hinstDLL,   // handle to the DLL module DLL模块的句柄，当前DLL被加载到什么位置
  DWORD fdwReason,      // reason for calling function DLL被调用的原因，有4种情况：DLL_PROCESS_ATTACH（当某个进程第一次执行LoadLibrary）、DLL_PROCESS_DETACH（当某个进程释放了DLL）、DLL_THREAD_ATTACH（当某个进程的其他线程再次执行LoadLibrary）、DLL_THREAD_DETACH（当某个进程的其他线程释放了DLL）
  LPVOID lpvReserved    // reserved
);
```



## 远程线程

**CreateThread**函数是在当前进程中创建线程，而**CreateRemoteThread**函数是允许在其他进程中创建线程，所以**远程线程就可以理解为是非本进程中的线程**。

**CreateRemoteThread**

```c
HANDLE CreateRemoteThread(
  HANDLE hProcess,                          // handle to process 输入类型，进程句柄
  LPSECURITY_ATTRIBUTES lpThreadAttributes, // SD 输入类型，安全属性，包含安全描述符
  SIZE_T dwStackSize,                       // initial stack size 输入类型，堆大小
  LPTHREAD_START_ROUTINE lpStartAddress,    // thread function 输入类型，线程函数，线程函数地址应该是在别的进程中存在的
  LPVOID lpParameter,                       // thread argument　输入类型，线程参数
  DWORD dwCreationFlags,                    // creation option 输入类型，创建设置
  LPDWORD lpThreadId                        // thread identifier 输出类型，线程id
);
```



远程线程调用的是目标线程的代码



首先创建A进程，代码如下：



```c
void Fun() {
    for(int i = 0; i <= 5; i++) {
        printf("Fun running... \n");
        Sleep(1000);
    }
}
 
DWORD WINAPI ThreadProc(LPVOID lpParameter) {
    Fun();
    return 0;
}
 
int main(int argc, char* argv[]) {
    
    HANDLE hThread = CreateThread(NULL, NULL, ThreadProc, NULL, 0, NULL);
    
    CloseHandle(hThread);
 
    getchar();
    return 0;
}
```



进程B写了一个远程线程创建的代码：



```c
BOOL MyCreateRemoteThread(DWORD dwProcessId, DWORD dwProcessAddr) {
    DWORD dwThreadId;
    HANDLE hProcess;
    HANDLE hThread;
    // 1. 获取进程句柄
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessId);
    // 判断OpenProcess是否执行成功
    if(hProcess == NULL) {
        OutputDebugString("OpenProcess failed! \n");
        return FALSE;
    }
    // 2. 创建远程线程
    hThread = CreateRemoteThread(
        hProcess,                          // handle to process
        NULL, // SD
        0,                       // initial stack size
        (LPTHREAD_START_ROUTINE)dwProcessAddr,    // thread function
        NULL,                       // thread argument
        0,                    // creation option
        &dwThreadId                        // thread identifier
    );
    // 判断CreateRemoteThread是否执行成功
    if(hThread == NULL) {
        OutputDebugString("CreateRemoteThread failed! \n");
        CloseHandle(hProcess);
        return FALSE;
    }
 
    // 3. 关闭
    CloseHandle(hThread);
    CloseHandle(hProcess);
 
    // 返回
    return TRUE;
}
```



## 远程线程注入

在安全领域，“注入”是非常重要的一种技术手段，注入与反注入也一直处于不断变化的，而且也愈来愈激烈的对抗当中。

**已知的注入方式：**

远程线程注入、APC注入、消息钩子注入、注册表注入、导入表注入、输入法注入等等。



远程线程注入的思路就是在进程A中创建线程，**将线程函数指向LoadLibrary函数**。



**利用dll加载的时候，创建一个线程。dll劫持的味道。**



DLL文件，在DLL文件入口函数判断并创建线程：

```c
#include "stdafx.h"
 
DWORD WINAPI ThreadProc(LPVOID lpParaneter) {
    for (;;) {
        Sleep(1000);
        printf("DLL RUNNING...");
    }
}
 
BOOL APIENTRY DllMain( HANDLE hModule, 
                       DWORD  ul_reason_for_call, 
                       LPVOID lpReserved
                     )
{   // 当进程执行LoadLibrary时创建一个线程，执行ThreadProc线程
    switch (ul_reason_for_call) {
    case DLL_PROCESS_ATTACH:
        CreateThread(NULL, 0, ThreadProc, NULL, 0, NULL);
        break;
    }
    return TRUE;
}
```



演示



````c
#include "StdAfx.h"
 
// LoadDll需要两个参数一个参数是进程ID，一个是DLL文件的路径
BOOL LoadDll(DWORD dwProcessID, char* szDllPathName) {
    
    BOOL bRet;
    HANDLE hProcess;
    HANDLE hThread;
    DWORD dwLength;
    DWORD dwLoadAddr;
    LPVOID lpAllocAddr;
    DWORD dwThreadID;
    HMODULE hModule;
    
    bRet = 0;
    dwLoadAddr = 0;
    hProcess = 0;
    
    // 1. 获取进程句柄
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessID);
    if (hProcess == NULL) {
        OutputDebugString("OpenProcess failed! \n");
        return FALSE;
    }
    
    // 2. 获取DLL文件路径的长度，并在最后+1，因为要加上0结尾的长度
    dwLength = strlen(szDllPathName) + 1;
    
    // 3. 在目标进程分配内存
    lpAllocAddr = VirtualAllocEx(hProcess, NULL, dwLength, MEM_COMMIT, PAGE_READWRITE);
    if (lpAllocAddr == NULL) {
        OutputDebugString("VirtualAllocEx failed! \n");
        CloseHandle(hProcess);
        return FALSE;
    }
    
    // 4. 拷贝DLL路径名字到目标进程的内存
    bRet = WriteProcessMemory(hProcess, lpAllocAddr, szDllPathName, dwLength, NULL);
    if (!bRet) {
        OutputDebugString("WriteProcessMemory failed! \n");
        CloseHandle(hProcess);
        return FALSE;
    }
    
    // 5. 获取模块句柄
    // LoadLibrary这个函数是在kernel32.dll这个模块中的，所以需要现货区kernel32.dll这个模块的句柄
    hModule = GetModuleHandle("kernel32.dll");
    if (!hModule) {
        OutputDebugString("GetModuleHandle failed! \n");
        CloseHandle(hProcess);
        return FALSE;
    }
    
    // 6. 获取LoadLibraryA函数地址
    dwLoadAddr = (DWORD)GetProcAddress(hModule, "LoadLibraryA");
    if (!dwLoadAddr){
        OutputDebugString("GetProcAddress failed! \n");
        CloseHandle(hModule);
        CloseHandle(hProcess);
        return FALSE;
    }
    
    // 7. 创建远程线程，加载DLL
    hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)dwLoadAddr, lpAllocAddr, 0, &dwThreadID);
    if (!hThread){
        OutputDebugString("CreateRemoteThread failed! \n");
        CloseHandle(hModule);
        CloseHandle(hProcess);
        return FALSE;
    }
    
    // 8. 关闭进程句柄
    CloseHandle(hThread);
    CloseHandle(hProcess);
    
    return TRUE;
}
 
int main(int argc, char* argv[]) {
    
    LoadDll(384, "C:\\Documents and Settings\\Administrator\\桌面\\test\\B.dll");
    getchar();
    return 0;
}
````



结论：打开目标程序，拿到句柄。在目前程序申请内存空间，把dll路径写进去(LoadLibraryA的参数)。然后在目标程序里面创建远程，调用LoadLibraryA加载dll。然后dll里面又可以存放恶意代码。

我们要灵活运行远程线程，配合上win32api进行各种骚操作。



## 模块隐藏

前面的注入程序很容易就可以通过API来获取当前加载的DLL模块，所以我们需要使用模块隐藏技术来隐藏自己需要注入的DLL模块。



### 模块隐藏之断链

API是通过什么将模块查询出来的？其实API都是从这几个结构体（**结构体属于3环应用层**）中查询出来的：

1. TEB(Thread Environment Block，线程环境块)，它存放线程的相关信息，每一个线程都有自己的TEB信息，FS:[0]即是当前线程的TEB。
2. PEB(Process Environment Block，进程环境块)，它存放进程的相关信息，每个进程都有自己的PEB信息，FS:[0x30]即当前进程的PEB。





FS寄存器中存储的就是当前正在使用的线程的TEB结构体的地址。

![image-20220108220340853](https://gitee.com/safe6/img/raw/master/image-20220108220340853.png)



然后找到30偏移出，跟随。

![image-20220108220843365](https://gitee.com/safe6/img/raw/master/image-20220108220843365.png)

![image-20220108221102610](https://gitee.com/safe6/img/raw/master/image-20220108221102610.png)





我们了解到了API函数遍历模块就是查看PEB那个链表，所以我们要想办法让它**在查询的时候断链**。



代码实现

```c
void HideModule(char* szModuleName) {
    // 获取模块的句柄
    HMODULE hMod = GetModuleHandle(szModuleName);
    PLIST_ENTRY Head, Cur;
    PPEB_LDR_DATA ldr;
    PLDR_MODULE ldmod;
    
    __asm {
        mov eax, fs:[0x30] // 取PEB结构体，fs30偏移处
            mov ecx, [eax + 0x0c] // 取PEB结构体的00c偏移的结构体，就是PEB_LDR_DATA
            mov ldr, ecx // 将ecx给到ldr
    }
    // 获取正在加载的模块列表
    Head = &(ldr->InLoadOrderModuleList);
    // 
    Cur = Head->Flink;
    do {
        // 宏CONTAINING_RECORD根据结构体中某成员的地址来推算出该结构体整体的地址
        ldmod = CONTAINING_RECORD(Cur, LDR_MODULE, InLoadOrderModuleList);
        // 循环遍历，如果地址一致则表示找到对应模块来，就进行断链
        if(hMod == ldmod->BaseAddress) {
            // 断链原理很简单就是将属性交错替换
            ldmod->InLoadOrderModuleList.Blink->Flink = ldmod->InLoadOrderModuleList.Flink;
            ldmod->InLoadOrderModuleList.Flink->Blink = ldmod->InLoadOrderModuleList.Blink;
            
            ldmod->InInitializationOrderModuleList.Blink->Flink = ldmod->InInitializationOrderModuleList.Flink;
            ldmod->InInitializationOrderModuleList.Flink->Blink = ldmod->InInitializationOrderModuleList.Blink;
            
            ldmod->InMemoryOrderModuleList.Blink->Flink = ldmod->InMemoryOrderModuleList.Flink;
            ldmod->InMemoryOrderModuleList.Flink->Blink = ldmod->InMemoryOrderModuleList.Blink;
        }
        Cur = Cur->Flink;
    } while (Head != Cur);
}

int main(int argc, char* argv[]) {
    getchar();
    HideModule("kernel32.dll");
    getchar();
    return 0;
}
```

### 模块隐藏之PE指纹



![](https://gitee.com/safe6/img/raw/master/image-20220108222205721.png)



### 模块隐藏之VAD树





## 注入代码



复制代码的编写原则

1. 不能有全局变量
2. 不能使用常量字符串
3. 不能使用系统调用
4. 不能嵌套调用其他函数



```c
#include <tlhelp32.h>
#include <stdio.h>
#include <windows.h>
 
typedef struct {
    DWORD dwCreateAPIAddr;                // Createfile函数的地址
    LPCTSTR lpFileName;                    // 下面都是CreateFile所需要用到的参数
    DWORD dwDesiredAccess;
    DWORD dwShareMode;
    LPSECURITY_ATTRIBUTES lpSecurityAttributes;
    DWORD dwCreationDisposition;
    DWORD dwFlagsAndAttributes;
    HANDLE hTemplateFile;
} CREATEFILE_PARAM;
 
// 定义一个函数指针
typedef HANDLE(WINAPI* PFN_CreateFile) (
    LPCTSTR lpFileName,
    DWORD dwDesiredAccess,
    DWORD dwShareMode,
    LPSECURITY_ATTRIBUTES lpSecurityAttributes,
    DWORD dwCreationDisposition,
    DWORD dwFlagsAndAttributes,
    HANDLE hTemplateFile
);
 
// 编写要复制到目标进程的函数
DWORD _stdcall CreateFileThreadProc(LPVOID lparam)
{
    CREATEFILE_PARAM* Gcreate = (CREATEFILE_PARAM*)lparam;
    PFN_CreateFile pfnCreateFile;
    pfnCreateFile = (PFN_CreateFile)Gcreate->dwCreateAPIAddr;
 
    // creatFile结构体全部参数
    pfnCreateFile(
        Gcreate->lpFileName,
        Gcreate->dwDesiredAccess,
        Gcreate->dwShareMode,
        Gcreate->lpSecurityAttributes,
        Gcreate->dwCreationDisposition,
        Gcreate->dwFlagsAndAttributes,
        Gcreate->hTemplateFile
    );
    
    return 0;
}
 
// 远程创建文件
BOOL RemotCreateFile(DWORD dwProcessID, char* szFilePathName)
{
    BOOL bRet;
    DWORD dwThread;
    HANDLE hProcess;
    HANDLE hThread;
    DWORD dwThreadFunSize;
    CREATEFILE_PARAM GCreateFile;
    LPVOID lpFilePathName;
    LPVOID lpRemotThreadAddr;
    LPVOID lpFileParamAddr;
    DWORD dwFunAddr;
    HMODULE hModule;
    
 
    bRet = 0;
    hProcess = 0;
    dwThreadFunSize = 0x400;
    // 1. 获取进程的句柄
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessID);
    if (hProcess == NULL)
    {
        OutputDebugString("OpenProcessError! \n");
        return FALSE;
    }
    // 2. 分配3段内存：存储参数，线程函数，文件名
 
    // 2.1 用来存储文件名 +1是要计算到结尾处
    lpFilePathName = VirtualAllocEx(hProcess, NULL, strlen(szFilePathName)+1, MEM_COMMIT, PAGE_READWRITE); // 在指定的进程中分配内存
    
    // 2.2 用来存储线程函数
    lpRemotThreadAddr = VirtualAllocEx(hProcess, NULL, dwThreadFunSize, MEM_COMMIT, PAGE_READWRITE); // 在指定的进程中分配内存
 
    // 2.3 用来存储文件参数
    lpFileParamAddr = VirtualAllocEx(hProcess, NULL, sizeof(CREATEFILE_PARAM), MEM_COMMIT, PAGE_READWRITE); // 在指定的进程中分配内存
 
 
    // 3. 初始化CreateFile参数
    GCreateFile.dwDesiredAccess = GENERIC_READ | GENERIC_WRITE;
    GCreateFile.dwShareMode = 0;
    GCreateFile.lpSecurityAttributes = NULL;
    GCreateFile.dwCreationDisposition = OPEN_ALWAYS;
    GCreateFile.dwFlagsAndAttributes = FILE_ATTRIBUTE_NORMAL;
    GCreateFile.hTemplateFile = NULL;
    
    // 4. 获取CreateFile的地址
    // 因为每个进程中的LoadLibrary函数都在Kernel32.dll中，而且此dll的物理页是共享的，所以我们进程中获得的LoadLibrary地址和别的进程都是一样的
    hModule = GetModuleHandle("kernel32.dll");
    GCreateFile.dwCreateAPIAddr = (DWORD)GetProcAddress(hModule, "CreateFileA");
    FreeLibrary(hModule);
 
    // 5. 初始化CreatFile文件名
    GCreateFile.lpFileName = (LPCTSTR)lpFilePathName;
 
    // 6. 修改线程函数起始地址
    dwFunAddr = (DWORD)CreateFileThreadProc;
    // 间接跳
    if (*((BYTE*)dwFunAddr) == 0xE9)
    {
        dwFunAddr = dwFunAddr + 5 + *(DWORD*)(dwFunAddr + 1);
    }
 
    // 7. 开始复制
    // 7.1 拷贝文件名
    WriteProcessMemory(hProcess, lpFilePathName, szFilePathName, strlen(szFilePathName) + 1, 0);
 
    // 7.2 拷贝线程函数
    WriteProcessMemory(hProcess, lpRemotThreadAddr, (LPVOID)dwFunAddr, dwThreadFunSize, 0);
 
    // 7.3 拷贝参数
    WriteProcessMemory(hProcess, lpFileParamAddr, &GCreateFile, sizeof(CREATEFILE_PARAM), 0);
 
    // 8. 创建远程线程
    hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)lpRemotThreadAddr, lpFileParamAddr, 0, &dwThread);// lpAllocAddr传给线程函数的参数.因为dll名字分配在内存中
    if (hThread == NULL)
    {
        OutputDebugString("CreateRemoteThread Error! \n");
        CloseHandle(hProcess);
        CloseHandle(hModule);
        return FALSE;
    }
 
    // 9. 关闭资源
    CloseHandle(hProcess);
    CloseHandle(hThread);
    CloseHandle(hModule);
    return TRUE;
 
}
 
// 根据进程名称获取进程ID
DWORD GetPID(char *szName)
{
    HANDLE hProcessSnapShot = NULL;
    PROCESSENTRY32 pe32 = {0};
    
    hProcessSnapShot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hProcessSnapShot == (HANDLE)-1)
    {
        return 0;
    }
    
    pe32.dwSize = sizeof(PROCESSENTRY32);
    if (Process32First(hProcessSnapShot, &pe32))
    {
        do {
            if (!strcmp(szName, pe32.szExeFile)) {
                return (int)pe32.th32ProcessID;
            }
        } while (Process32Next(hProcessSnapShot, &pe32));
    }
    else
    {
        CloseHandle(hProcessSnapShot);
    }
    return 0;
}
 
int main()
{
    RemotCreateFile(GetPID("进程名"), "文件名");
    return 0;
}
```

