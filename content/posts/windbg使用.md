---
title: "Windbg使用"
date: 2020-11-30T14:46:08+08:00
draft: true
tags: ["windbg"]
categories: ["逆向"] 
---



### 实现进程隐藏

`dt_eprocess` 可查看到当前的EPROCESS结构体的偏移信息

```
kd> dt_eprocess

nt!_EPROCESS
   +0x000 Pcb              : _KPROCESS
   +0x160 ProcessLock      : _EX_PUSH_LOCK
   +0x168 CreateTime       : _LARGE_INTEGER                   // 创建时间
   +0x170 ExitTime         : _LARGE_INTEGER                     // 退出时间
   +0x180 UniqueProcessId  : Ptr64 Void                         // 进程的PID
   +0x188 ActiveProcessLinks : _LIST_ENTRY                    // 活动进程链表
   +0x200 ObjectTable      : Ptr64 _HANDLE_TABLE          // 指向句柄表的指针
   +0x2d8 Session          : Ptr64 Void                                // 会话列表
   +0x2e0 ImageFileName    : [15] UChar                         // 进程的名称
   +0x308 ThreadListHead   : _LIST_ENTRY                       // 进程中的线程链表结构
   +0x320 Wow64Process     : Ptr64 Void                        // 32位进程链表
   +0x328 ActiveThreads    : Uint4B                                 // 活动的线程
   +0x32c ImagePathHash    : Uint4B                               // 镜像路径的Hash值
   +0x338 Peb              : Ptr64 _PEB                                 // 指向PEB结构的指针
   +0x440 Flags            : Uint4B                                       // 进程标志
```

要实现进程的隐藏我们需要关注结构中的 `ActiveProcessLinks` 该指针把每个进程的EPROCESS结构体连接成了双向链表，我们可以使用 `ZwQuerySystemInformation` 这个函数来遍历出所有的进程信息，要实现进程的隐藏，只需要将某个进程的EPROCESS从结构体中摘除，那么通过`ZwQuerySystemInformation`函数就无法遍历出被摘链的进程了，从而实现了进程的隐藏。

通过 `PsLookupProcessByProcessId`函数获取到指定进程的ID，然后通过`PsGetProcessImageFileName` 取出结构名称，最后通过 `_strcmp` 判断是不是我们要的隐藏的程序。

代码实现如下

```C++
PEPROCESS GetProcessObjectByName(char *name)
{
	SIZE_T temp;
	for (temp = 100; temp<10000; temp += 4)
	{
		NTSTATUS status;
		PEPROCESS ep;
		status = PsLookupProcessByProcessId((HANDLE)temp, &ep);
		if (NT_SUCCESS(status))
		{
			char *pn = PsGetProcessImageFileName(ep);
			if (_stricmp(pn, name) == 0)
				return ep;
		}
	}
	return NULL;
}
```

获得句柄之后直接摘除进程的结构即可实现隐藏。

```C++
VOID RemoveListEntry(PLIST_ENTRY ListEntry)
{
	KIRQL OldIrql;
	OldIrql = KeRaiseIrqlToDpcLevel();
	if (ListEntry->Flink != ListEntry &&ListEntry->Blink != ListEntry &&ListEntry->Blink->Flink == ListEntry &&ListEntry->Flink->Blink == ListEntry)
	{
		ListEntry->Flink->Blink = ListEntry->Blink;
		ListEntry->Blink->Flink = ListEntry->Flink;
		ListEntry->Flink = ListEntry;
		ListEntry->Blink = ListEntry;
	}
	KeLowerIrql(OldIrql);
}

```

其中有一位(偏移为0x43c)：

```
+0x43c ProtectedProcess : Pos 11, 1 Bit
```

如果此位为1，进程拒绝被打开。

[进程隐藏参考](https://www.cnblogs.com/LyShark/p/11652019.html)