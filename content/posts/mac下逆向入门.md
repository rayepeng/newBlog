---
title: "Mac下逆向入门"
date: 2020-12-01T11:06:55+08:00
draft: true
tags: []
categories: [] 
---



## IDA基本知识



```
 52 #define _BYTE  uint8
 53 #define _WORD  uint16
 54 #define _DWORD uint32
 55 #define _QWORD uint64
 51 // Partially defined types:
 52 #define _BYTE  uint8
 53 #define _WORD  uint16
 54 #define _DWORD uint32
 55 #define _QWORD uint64
```



mac IDA安装 FindCrypt插件

```bash
sudo easy_install pip #自带的python没有pip命令
python -m pip install yara-python==3.11.0
```





## KCTF春季赛复盘

### 签到题





定位到代码：

```
.text:00401A16                 call    ?GetBuffer@CString@@QAEPADH@Z ; CString::GetBuffer(int)
.text:00401A1B                 mov     [ebp+Str], eax
.text:00401A1E                 mov     edx, [ebp+Str]
.text:00401A21                 push    edx             ; Str
.text:00401A22                 call    ds:atoi
.text:00401A28                 add     esp, 4
.text:00401A2B                 mov     [ebp+var_20], eax
.text:00401A2E                 mov     eax, [ebp+var_20]
.text:00401A31                 cdq
.text:00401A32                 sub     eax, edx
.text:00401A34                 sar     eax, 1
.text:00401A36                 cmp     eax, 50h
.text:00401A39                 jnz     short loc_401A42
.text:00401A3B                 call    sub_401750
.text:00401A40                 jmp     short loc_401A47
```

最关键的就是

```
.text:00401A34                 sar     eax, 1
.text:00401A36                 cmp     eax, 50h
```

sar右移，除2，再和 50h 比较，如果不为0就跳出去了，那么输入160就行了

![](https://picgo-1258058044.cos.ap-chengdu.myqcloud.com/img/20201127155156.png)





### 第二题

分析IDA代码遇到很奇怪的问题



```C++
int __cdecl sub_401000(int a1, signed int a2, int a3)
{
  int v3; // ebp@3
  int v4; // esi@4
  int result; // eax@4
  int v6; // edi@5
  char v7; // cl@6
  signed int v8; // edx@6
  unsigned __int8 v9; // cl@7
  char v10; // cl@9

  if ( a1 && !(a2 % 2) && (v3 = a3) != 0 )
  {
    v4 = 0;
    result = a2 / 2; //32
    if ( a2 / 2 > 0 )
    {
      v6 = a1;
      do
      {
        v7 = *(_BYTE *)v6;                      // 取出第一个字节
                                                // 
        BYTE1(a1) = *(_BYTE *)(v6 + 1);       
        LOBYTE(a1) = v7;  
        v8 = 0;
        do
        {
          v9 = *((_BYTE *)&a1 + v8);
          if ( v9 < '0' || v9 > '9' )
          {
            if ( v9 < 'a' || v9 > 'f' )
            {
              if ( v9 < 'A' || v9 > 'F' )
                goto LABEL_19;
              v10 = v9 - '7';
            }
            else
            {
              v10 = v9 - 'W';
            }
          }
          else
          {
            v10 = v9 - '0';
          }
          *((_BYTE *)&a1 + v8++) = v10;
        }
        while ( v8 < 2 ); 
        v6 += 2;
        *(_BYTE *)(v4++ + v3) = BYTE1(a1) & 0xF | 16 * a1;
      }
      while ( v4 < result );
    }
  }
  else
  {
LABEL_19:
    result = 0;
  }
  return result;
}
```

这里最大的困惑就是

```C++
  v7 = *(_BYTE *)v6;                      // 取出第一个字节
                                          // 
  BYTE1(a1) = *(_BYTE *)(v6 + 1);       
  LOBYTE(a1) = v7; 
```

要知道a1是指向了输入的序列号的指针，这里直接对a1进行了修改导致我以为会修改输入的序列号。

经过不断地调试发现，这里并没有修改，只是借用了a1的末尾两个字节做存储而已

代码的作用如下：

将输入的 `eaa209cf68dfd48b77e1cddd00a4a48abef8decfc1df630e242548c364503eff`

转为 `0xeaa209cf68dfd48b77e1cddd00a4a48abef8decfc1df630e242548c364503eff` 的形式

![](https://gitee.com/xinyongp/image/raw/master/20201201213528.png)



如何识别加密算法？

看到 FindCrypt识别出来的常量

![](https://gitee.com/xinyongp/image/raw/master/20201202094004.png)

AES加密走的是这里：

```C++
signed int __cdecl sub_4053B0(_DWORD *a1, int a2, unsigned int *a3)
{
  signed int result; // eax
  int v4; // esi
  int v5; // edx
  unsigned int *v6; // ecx
  unsigned int *v7; // eax
  unsigned int v8; // edi
  unsigned int v9; // edi
  unsigned int v10; // edi
  unsigned int v11; // edi
  signed int v12; // esi
  int *v13; // eax
  unsigned int v14; // ecx
  int v15; // edx
  int v16; // ebx
  unsigned int v17; // ecx
  int v18; // edx
  int v19; // ebx
  unsigned int v20; // ecx
  int v21; // edx
  int v22; // ebx
  unsigned int v23; // ecx

  result = sub_404FF0(a1, a2, a3);
  if ( result < 0 )
    return result;
  v4 = 0;
  v5 = 4 * a3[60];
  if ( v5 > 0 )
  {
    v6 = &a3[4 * a3[60] + 2];
    v7 = a3 + 2;
    do
    {
      v8 = *(v7 - 2);
      *(v7 - 2) = *(v6 - 2);
      *(v6 - 2) = v8;
      v9 = *(v7 - 1);
      *(v7 - 1) = *(v6 - 1);
      *(v6 - 1) = v9;
      v10 = *v7;
      *v7 = *v6;
      *v6 = v10;
      v11 = v7[1];
      v7[1] = v6[1];
      v6[1] = v11;
      v4 += 4;
      v5 -= 4;
      v7 += 4;
      v6 -= 4;
    }
    while ( v4 < v5 );
  }
  v12 = 1;
  if ( (signed int)a3[60] > 1 )
  {
    v13 = (int *)(a3 + 2);
    do
    {
      v14 = v13[2];
      v13 += 4;
      v15 = dword_41A398[dword_419798[v14 >> 24] & 0xFF] ^ dword_41AB98[dword_419798[BYTE1(v14)] & 0xFF] ^ dword_41A798[dword_419798[BYTE2(v14)] & 0xFF];
      v16 = dword_41AF98[dword_419798[(unsigned __int8)v14] & 0xFF];
      v17 = *(v13 - 1);
      *(v13 - 2) = v16 ^ v15;
      v18 = dword_41A398[dword_419798[v17 >> 24] & 0xFF] ^ dword_41AB98[dword_419798[BYTE1(v17)] & 0xFF] ^ dword_41A798[dword_419798[BYTE2(v17)] & 0xFF];
      v19 = dword_41AF98[dword_419798[(unsigned __int8)v17] & 0xFF];
      v20 = *v13;
      *(v13 - 1) = v19 ^ v18;
      v21 = dword_41A398[dword_419798[v20 >> 24] & 0xFF] ^ dword_41AB98[dword_419798[BYTE1(v20)] & 0xFF] ^ dword_41A798[dword_419798[BYTE2(v20)] & 0xFF];
      v22 = dword_41AF98[dword_419798[(unsigned __int8)v20] & 0xFF];
      v23 = v13[1];
      *v13 = v22 ^ v21;
      ++v12;
      v13[1] = dword_41AF98[dword_419798[(unsigned __int8)v23] & 0xFF] ^ dword_41A398[dword_419798[v23 >> 24] & 0xFF] ^ dword_41AB98[dword_419798[BYTE1(v23)] & 0xFF] ^ dword_41A798[dword_419798[BYTE2(v23)] & 0xFF];
    }
    while ( v12 < (signed int)a3[60] );
  }
  result = 0;
  return result;
}
```



OpenSSL 中，do_encrypt为1时加密，为0时解密。



解密脚本

```python
import gmpy2
from Crypto.Cipher import AES
 
n = 0x69823028577465AB3991DF045146F91D556DEE8870845D8EE1CD3CF77E4A0C39
q = 201522792635114097998567775554303915819
p = 236811285547763449711675622888914229291
e = 0x10001
c = 0x2888888888888888888888888880014AF58AD4D76D59D8D2171FFB4CA2231
phi_n= (p - 1) * (q - 1)
d = gmpy2.invert(e, phi_n)
 
print(hex(pow(c,d,n)))
 
 
s='2d5f4c9d567c43399312b8898d6c7f2ec799c64bde4fe39eb01771be1e7f4795'.decode('hex')
key = '480B62C3ACD6C8A36B18D9E906CD90D2'.decode('hex')
 
cipher = AES.new(key, AES.MODE_ECB)
encrypted = cipher.encrypt(s).encode('hex')
print("name = KCTF")
print("sn = "+encrypted)
```

### 第三题

采用了quickjs进行编译，难度有点大先放着

