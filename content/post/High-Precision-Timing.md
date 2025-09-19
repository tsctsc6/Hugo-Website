+++
date = '2025-09-19T18:03:27+08:00'
draft = false
title = '高精度计时'
categories = ['Main Sections']
tags = ['C', '.NET']
+++

目前，要实现高精度计时，要依靠CPU内部时钟来计时。而这需要依赖操作系统的接口，因此对于各个操作系统，方法不同。

## Windows
官方文档: [获取高分辨率时间戳](https://learn.microsoft.com/zh-cn/windows/win32/sysinfo/acquiring-high-resolution-time-stamps)

### C
```
#include <stdio.h>
#include <windows.h>

int main()
{
    LARGE_INTEGER tick;
    LONGLONG start, end, freq;
    QueryPerformanceFrequency(&tick);
    freq = tick.QuadPart;

    QueryPerformanceCounter(&tick);
    start = tick.QuadPart;
    printf("Done something.\n");
    QueryPerformanceCounter(&tick);
    end = tick.QuadPart;

    printf("time: %.9f s\n", (end - start) / (double)freq);
    return 0;
}
```

### .NET
官方文档: [Stopwatch 类](https://learn.microsoft.com/zh-cn/dotnet/api/system.diagnostics.stopwatch)

```
long start = Stopwatch.GetTimestamp();
Console.WriteLine("Done something.");
TimeSpan delta = Stopwatch.GetElapsedTime(start);
```
