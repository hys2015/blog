---
title: Debug in visual c++
date: 2017-02-18 10:12:00
tags: 
- VisualC++ 
- Debug 
- VisualStudio
category: 学习记录
---

最近在研究师兄的代码，编译一遍挺久，所以趁此机会了解一下相关debug的技巧和方法，有助于提高自己的开发效率。

<!--more-->

## VS调试器调试技巧
http://blog.jobbole.com/33865/
http://hovertree.com/h/bjaf/pbqy1f1k.htm
http://blog.jobbole.com/45249/

- 调用堆栈
- 鼠标悬停查看变量信息
- 调试过程中直接给变量赋值
- 拖动黄色箭头改变程序运行流程
- 通过`ptr+offset,count`添加变量监视(watch)

## 断点和追踪点
https://msdn.microsoft.com/zh-cn/library/232dxah7(v=vs.90).aspx
##  MACRO - ASSERT、TRACE、VERIFY
http://blog.csdn.net/chocolateconanlan/article/details/4061545
博客最后部分
以及MSDN
https://msdn.microsoft.com/en-us/library/9sb57dw4.aspx
>**Syntax语法**
```
assert(   
   expression   
);  
void _assert(  
   char const* message,  
   char const* filename,  
   unsigned line  
);  
void _wassert(  
   wchar_t const* message,  
   wchar_t const* filename,  
   unsigned line  
);  
```
>**Parameters参数**
expression
: A scalar expression (including pointer expressions) that evaluates to nonzero (true) or 0 (false).

message
: The message to display.

filename
: The name of the source file the assertion failed in.

line
: The line number in the source file of the failed assertion.

## MACRO - OutputDebugString
>You can use OutputDebugString. OutputDebugString is a macro that depending on your build options either maps to `OutputDebugStringA(char const*)`or `OutputDebugStringW(wchar_t const*)`. In the later case you will have to supply a wide character string to the function. To create a wide character literal you can use the L prefix:
`OutputDebugStringW(L"My output string.");`
Normally you will use the macro version together with the _T macro like this:
`OutputDebugString(_T("My output string."));`
If you project is configured to build for UNICODE it will expand into:
`OutputDebugStringW(L"My output string.");`
If you are not building for UNICODE it will expand into:
`OutputDebugStringA("My output string.");`

## Logging Class
Logging Framework from Dr.p
http://www.drdobbs.com/cpp/logging-in-c/201804215

简单的stderr debug类，来自
http://stackoverflow.com/questions/6168107/how-to-implement-a-good-debug-logging-feature-in-a-project
```
#ifndef _LOGGER_HPP_
#define _LOGGER_HPP_

#include <iostream>
#include <sstream>

/* consider adding boost thread id since we'll want to know whose writting and
 * won't want to repeat it for every single call */

/* consider adding policy class to allow users to redirect logging to specific
 * files via the command line
 */

enum loglevel_e
    {logERROR, logWARNING, logINFO, logDEBUG, logDEBUG1, logDEBUG2, logDEBUG3, logDEBUG4};

class logIt
{
public:
    logIt(loglevel_e _loglevel = logERROR) {
        _buffer << _loglevel << " :" 
            << std::string(
                _loglevel > logDEBUG 
                ? (_loglevel - logDEBUG) * 4 
                : 1
                , ' ');
    }

    template <typename T>
    logIt & operator<<(T const & value)
    {
        _buffer << value;
        return *this;
    }

    ~logIt()
    {
        _buffer << std::endl;
        // This is atomic according to the POSIX standard
        // http://www.gnu.org/s/libc/manual/html_node/Streams-and-Threads.html
        std::cerr << _buffer.str();
    }

private:
    std::ostringstream _buffer;
};

extern loglevel_e loglevel;

#define log(level) \
if (level > loglevel) ; \
else logIt(level)

#endif
```
用法
```
// define and turn off for the rest of the test suite
loglevel_e loglevel = logERROR;

void logTest(void) {
    loglevel_e loglevel_save = loglevel;

    loglevel = logDEBUG4;

    log(logINFO) << "foo " << "bar " << "baz";

    int count = 3;
    log(logDEBUG) << "A loop with "    << count << " iterations";
    for (int i = 0; i != count; ++i)
    {
        log(logDEBUG1) << "the counter i = " << i;
        log(logDEBUG2) << "the counter i = " << i;
    }

    loglevel = loglevel_save;
}
```